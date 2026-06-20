# Architecture Guide

## System Design

### Microservices Architecture

The betting platform uses a **multi-tenant microservices** architecture designed for high concurrency, low latency, and regulatory compliance across multiple African countries.

```
                    ┌──────────────────────┐
                    │     Load Balancer     │
                    │    (Cloudflare/AWS)   │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │    API Gateway        │
                    │    (Port 8080)        │
                    │  HTTP + WebSocket     │
                    │  Auth + Rate Limit    │
                    └──────────┬───────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
   ┌──────▼──────┐    ┌───────▼──────┐    ┌───────▼──────┐
   │   Wallet    │    │   Betting    │    │    Games     │
   │   Service   │    │   Engine     │    │   Service    │
   │  (Port 8081)│    │  (Port 8082) │    │  (Port 8083) │
   └──────┬──────┘    └───────┬──────┘    └───────┬──────┘
          │                    │                    │
   ┌──────▼────────────────────▼────────────────────▼──────┐
   │                    NATS Event Bus                      │
   │              (JetStream for persistence)               │
   └──────┬────────────────────┬────────────────────┬──────┘
          │                    │                    │
   ┌──────▼──────┐    ┌───────▼──────┐    ┌───────▼──────┐
   │ PostgreSQL  │    │    Redis     │    │  Settlement  │
   │   (RDS)     │    │ (ElastiCache)│    │   Service    │
   └─────────────┘    └──────────────┘    └──────────────┘
```

## Service Breakdown

### 1. API Gateway (`cmd/gateway`)
- **Port:** 8080
- **Responsibilities:**
  - HTTP/REST API routing
  - WebSocket connections (crash games)
  - JWT authentication
  - Rate limiting (Redis-backed)
  - Request validation
  - CORS handling
  - Prometheus metrics

### 2. Wallet Service (`cmd/wallet`)
- **Port:** 8081
- **Responsibilities:**
  - Balance management (atomic operations)
  - Deposit processing (M-Pesa STK Push)
  - Withdrawal processing (M-Pesa B2C)
  - Transaction history
  - Audit trail (balance snapshots)
- **Key Pattern:** Optimistic locking with `SELECT ... FOR UPDATE`

### 3. Betting Engine (`cmd/engine`)
- **Port:** 8082
- **Responsibilities:**
  - Bet placement (single, multi, system)
  - Odds management
  - Bet validation
  - Tax calculation
  - Stake reservation
- **Key Pattern:** State machine for bet lifecycle

### 4. Games Service (`cmd/games`)
- **Port:** 8083
- **Responsibilities:**
  - Crash game engine
  - Provably fair algorithm
  - Real-time multiplier (WebSocket)
  - Game history
  - House edge management
- **Key Pattern:** Goroutine per game round + WebSocket broadcast

### 5. Settlement Service (`cmd/settlement`)
- **Port:** 8084
- **Responsibilities:**
  - Winner calculation
  - Payout processing
  - Tax deduction on winnings
  - Jackpot distribution
- **Key Pattern:** Event-driven via NATS

## Database Design

### Multi-Tenant Isolation
Every table includes `country_code` for data isolation:
```sql
CREATE TABLE bets (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    country_code VARCHAR(2) NOT NULL,  -- 'KE', 'NG', 'GH'
    stake DECIMAL(12,2) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Queries always filter by country
CREATE INDEX idx_bets_country ON bets(country_code, created_at);
```

### Optimistic Locking (Wallet)
```sql
-- Prevent double-spending
BEGIN;
SELECT balance, version FROM wallets 
WHERE user_id = $1 FOR UPDATE;

UPDATE wallets 
SET balance = balance - $2, version = version + 1
WHERE id = $1 AND version = $3;

INSERT INTO transactions (user_id, amount, balance_before, balance_after, type)
VALUES ($1, $2, $4, $4 - $2, 'debit');
COMMIT;
```

### Transaction Audit Trail
```sql
CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    country_code VARCHAR(2) NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    balance_before DECIMAL(12,2) NOT NULL,
    balance_after DECIMAL(12,2) NOT NULL,
    type VARCHAR(20) NOT NULL,  -- 'credit', 'debit'
    reference VARCHAR(100),      -- M-Pesa receipt, bet ID
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Event-Driven Communication

### NATS Subjects
```
bets.placed        → Bet placed event
bets.settled       → Bet settled event
wallet.credited    → Wallet credited
wallet.debited     → Wallet debited
games.crash.round  → Crash game round result
games.crash.tick   → Crash game multiplier tick
kyc.verified       → KYC verification complete
```

### Event Flow: Bet Placement
```
1. Client POST /api/bets
2. Gateway validates JWT + request
3. Engine validates odds + stake
4. Engine publishes "bet.validated" to NATS
5. Wallet service debits stake (atomic)
6. Wallet publishes "wallet.debited" to NATS
7. Engine persists bet
8. Engine publishes "bet.placed" to NATS
9. Settlement service subscribes for future settlement
```

### Event Flow: Crash Game Round
```
1. Games service starts new round
2. Generates provably fair seed
3. Publishes "games.crash.round" with commitment
4. Every 100ms: publishes "games.crash.tick" with multiplier
5. On crash: publishes "games.crash.crashed" with final multiplier
6. Settlement service processes all bets
7. Wallet service credits winners
```

## Provably Fair Algorithm

### Crash Game Fairness
```go
// 1. Generate server seed (random 32 bytes)
serverSeed := crypto.RandomBytes(32)

// 2. Create commitment (hash of server seed)
commitment := sha256(serverSeed)

// 3. Combine with client seed + round number
combined := sha256(serverSeed + clientSeed + roundNumber)

// 4. First 4 bytes → uint32 → float [0, 1)
value := binary.BigEndian.Uint32(combined[:4]) / float64(2^32)

// 5. Calculate crash point
crashPoint = 0.99 / (1 - value)

// 6. Apply house edge
crashPoint = crashPoint * (1 - houseEdge)
```

### Verification
Players can verify any round:
```
GET /api/fairness/verify/{roundId}
→ Returns: server_seed, client_seed, round_number, crash_point
→ Player can independently calculate and verify
```

## Security

### Authentication
- JWT tokens (access + refresh)
- Bcrypt password hashing (cost=12)
- Token expiration: 15min access, 7d refresh

### Rate Limiting
- Redis-backed sliding window
- Per-endpoint limits
- Per-user limits
- IP-based fallback

### Input Validation
- Request size limits
- SQL injection prevention (parameterized queries)
- XSS protection (secure JSON encoding)
- Integer overflow bounds checking

### Infrastructure
- TLS 1.3 everywhere
- Database SSL required in production
- Secrets management (AWS Secrets Manager)
- Network isolation (VPC)

## Performance

### Caching Strategy (Redis)
| Data | TTL | Invalidation |
|------|-----|--------------|
| User sessions | 24h | On logout |
| Live odds | 5s | On update |
| Match list | 30s | On change |
| Leaderboard | 60s | Periodic |
| Rate limit counters | 1min | Automatic |

### Database Optimization
- Connection pooling (pgx)
- Prepared statements
- Composite indexes on (country_code, created_at)
- Partitioning by country_code for large tables
- Read replicas for analytics

### WebSocket Optimization
- Goroutine per connection
- Buffered channels for broadcast
- Message batching for high-frequency updates
- Connection pooling for horizontal scaling
