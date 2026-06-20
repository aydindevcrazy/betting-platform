# 🚀 lets-bet — Quick Setup Guide

> **Production-ready betting platform** built with Go microservices.
> Sports betting + Crash games + M-Pesa/Flutterwave payments.

---

## 📋 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start (5 minutes)](#quick-start)
4. [Environment Variables](#environment-variables)
5. [API Endpoints](#api-endpoints)
6. [Database Schema](#database-schema)
7. [Deployment](#deployment)
8. [Next Steps: Frontend](#next-steps-frontend)

---

## 🏗 Architecture Overview

```
┌─────────────────────────────────────────────────┐
│              API GATEWAY (Port 8080)             │
│     HTTP/REST + WebSocket + Auth + Rate Limit    │
└──────────────────────┬──────────────────────────┘
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
┌────▼────┐      ┌────▼────┐      ┌────▼────┐
│ WALLET  │      │ ENGINE  │      │  GAMES  │
│ Service │      │ Service │      │ Service │
└────┬────┘      └────┬────┘      └────┬────┘
     │                 │                 │
     └─────────────────┼─────────────────┘
                       │
          ┌────────────▼────────────┐
          │     PostgreSQL 15       │
          │  (Multi-tenant + ACID)  │
          └─────────────────────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
     ┌────▼────┐ ┌────▼────┐ ┌────▼─────┐
     │  Redis  │ │  NATS   │ │Cloudflare│
     │ (Cache) │ │ (Queue) │ │  (CDN)   │
     └─────────┘ └─────────┘ └──────────┘
```

### Microservices

| Service | Port | Description |
|---------|------|-------------|
| **Gateway** | 8080 | Public API, WebSocket, auth, rate limiting |
| **Wallet** | 8081 | Balance, deposits, withdrawals |
| **Engine** | 8082 | Betting logic, odds processing |
| **Settlement** | 8083 | Winner payouts, tax calculation |
| **Games** | 8084 | Crash game engine (Aviator-style) |
| **Migrate** | — | Database migration tool |

### Tech Stack

- **Backend:** Go 1.26+ (high concurrency, low latency)
- **Database:** PostgreSQL 15 (ACID + optimistic locking)
- **Cache:** Redis 7 (live odds, sessions, leaderboards)
- **Queue:** NATS (event-driven with JetStream)
- **WebSocket:** Gorilla WebSocket (real-time crash games)
- **Payments:** M-Pesa (Kenya), Flutterwave (NG/GH), Paystack (NG)
- **KYC:** Smile ID (Kenya identity verification)
- **Observability:** Prometheus metrics, structured logging

---

## 📦 Prerequisites

- **Go 1.26+** — [Download](https://go.dev/dl/)
- **Docker & Docker Compose** — [Download](https://docs.docker.com/get-docker/)
- **Make** (usually pre-installed on Linux/Mac)

### Optional (for production)
- AWS account (af-south-1 Cape Town region recommended)
- Safaricom Daraja API credentials (M-Pesa)
- Sportradar or Betgenius API key (odds feed)
- Smile ID account (KYC)
- Cloudflare account (CDN + WAF)

---

## ⚡ Quick Start

### 1. Clone & Start Infrastructure

```bash
git clone https://github.com/nutcas3/lets-bet.git
cd lets-bet

# Start PostgreSQL, Redis, NATS via Docker
docker compose up -d
```

### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env with your settings (see below)
```

### 3. Build All Services

```bash
make build
```

### 4. Run Database Migrations

```bash
make migrate-up
```

### 5. Start All Services

```bash
# Start all services at once
make run-all

# Or start individually:
./bin/gateway   # Port 8080
./bin/wallet    # Port 8081
./bin/engine    # Port 8082
./bin/settlement # Port 8083
./bin/games     # Port 8084
```

### 6. Verify It's Running

```bash
# Health check
curl http://localhost:8080/healthz

# Metrics
curl http://localhost:8080/metrics

# Register a user
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "phone_number": "254712345678",
    "email": "test@example.com",
    "password": "SecurePass123!",
    "full_name": "Test User",
    "date_of_birth": "1990-01-15",
    "national_id": "12345678"
  }'
```

---

## 🔧 Environment Variables

### Core Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `ENVIRONMENT` | `development` | `development` or `production` |
| `PORT` | `8080` | Gateway port |
| `COUNTRY_CODE` | `KE` | `KE` (Kenya), `NG` (Nigeria), `GH` (Ghana) |
| `CURRENCY` | `KES` | `KES`, `NGN`, `GHS` |
| `TIMEZONE` | `Africa/Nairobi` | Server timezone |

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_HOST` | `localhost` | PostgreSQL host |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_NAME` | `betting_db` | Database name |
| `DB_USER` | `betting_user` | Database user |
| `DB_PASSWORD` | `betting_pass_2026` | Database password |
| `DB_SSL_MODE` | `disable` | Set to `require` in production |

### Redis & NATS

| Variable | Default | Description |
|----------|---------|-------------|
| `REDIS_HOST` | `localhost` | Redis host |
| `REDIS_PORT` | `6379` | Redis port |
| `NATS_URL` | `nats://localhost:4222` | NATS connection URL |

### Security

| Variable | Default | Description |
|----------|---------|-------------|
| `JWT_SECRET` | — | **CHANGE THIS** — JWT signing key |
| `JWT_EXPIRY` | `24h` | Access token expiry |
| `JWT_REFRESH_EXPIRY` | `168h` | Refresh token expiry (7 days) |
| `BCRYPT_COST` | `12` | Password hashing cost |
| `RATE_LIMIT_REQUESTS` | `100` | Requests per window |
| `RATE_LIMIT_WINDOW` | `1m` | Rate limit window |

### Betting Limits

| Variable | Default | Description |
|----------|---------|-------------|
| `MIN_BET_STAKE` | `10` | Minimum bet amount |
| `MAX_BET_STAKE` | `100000` | Maximum bet amount |
| `MAX_SELECTIONS_MULTI` | `20` | Max selections in multi bet |
| `CRASH_MIN_BET` | `10` | Min crash game bet |
| `CRASH_MAX_BET` | `10000` | Max crash game bet |
| `CRASH_MAX_MULTIPLIER` | `100` | Max crash multiplier |
| `CRASH_HOUSE_EDGE` | `0.01` | 1% house edge |

### Tax (Kenya)

| Variable | Default | Description |
|----------|---------|-------------|
| `TAX_GGR_RATE` | `0.15` | 15% on Gross Gaming Revenue |
| `TAX_WHT_RATE` | `0.20` | 20% Withholding Tax on winnings |
| `TAX_THRESHOLD` | `500` | Min winning amount subject to tax |

### Payment Providers

**M-Pesa (Kenya):**
```
MPESA_ENVIRONMENT=sandbox
MPESA_CONSUMER_KEY=your_key
MPESA_CONSUMER_SECRET=your_secret
MPESA_SHORTCODE=174379
MPESA_PASSKEY=your_passkey
MPESA_CALLBACK_URL=https://yourdomain.com/api/mpesa/callback
```

**Paystack (Nigeria):**
```
PAYSTACK_SECRET_KEY=your_key
PAYSTACK_PUBLIC_KEY=your_key
```

**Flutterwave (Multi-country):**
```
FLUTTERWAVE_SECRET_KEY=your_key
FLUTTERWAVE_PUBLIC_KEY=your_key
```

### Odds Providers

```
SPORTRADAR_API_KEY=your_key
BETGENIUS_API_KEY=your_key
```

### KYC Providers

```
SMILE_ID_API_KEY=your_key
SMILE_ID_PARTNER_ID=your_id
METAMAP_CLIENT_ID=your_id
METAMAP_CLIENT_SECRET=your_secret
```

---

## 🔌 API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/auth/register` | Register new user |
| `POST` | `/api/v1/auth/login` | Login, get JWT |
| `POST` | `/api/v1/auth/refresh` | Refresh token |

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/users/me` | Get profile |
| `PUT` | `/api/v1/users/me` | Update profile |
| `POST` | `/api/v1/users/me/self-exclude` | Self-exclusion |

### Wallet & Payments

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/users/me/wallet` | Get balance |
| `POST` | `/api/v1/payments/deposit` | Deposit (M-Pesa STK Push) |
| `POST` | `/api/v1/payments/withdraw` | Withdraw (M-Pesa B2C) |
| `GET` | `/api/v1/payments/transactions` | Transaction history |

### Sports Betting

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/odds/live` | Live matches with odds |
| `GET` | `/api/v1/odds/upcoming` | Upcoming matches |
| `POST` | `/api/v1/bets` | Place bet (single/multi) |
| `GET` | `/api/v1/bets/{bet_id}` | Bet details |
| `GET` | `/api/v1/bets/history` | Bet history |

### Crash Game (Aviator)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/games/crash/current` | Current game state |
| `GET` | `/api/v1/games/crash/history` | Game history |
| `POST` | `/api/v1/games/crash/bet` | Place crash bet |
| `POST` | `/api/v1/games/crash/verify` | Verify provably fair |

### WebSocket

| Endpoint | Description |
|----------|-------------|
| `ws://host/ws/games/{game_id}` | Real-time crash game |

---

## 🗄 Database Schema

### Core Tables

- **users** — User accounts with KYC data
- **wallets** — User balances (multi-currency)
- **transactions** — All financial transactions
- **bets** — Bet records (single, multi, system)
- **bet_selections** — Individual selections within bets
- **events** — Sports events/matches
- **markets** — Betting markets (1X2, O/U, etc.)
- **odds** — Odds data per market
- **games** — Crash game rounds
- **game_bets** — Crash game bets
- **mpesa_deposits** — M-Pesa deposit records
- **flutterwave_deposits** — Flutterwave deposit records

### Migrations

```bash
# Run all pending migrations
make migrate-up

# Rollback last migration
make migrate-down

# Check current version
make migrate-version
```

---

## 🚢 Deployment

### Docker (Local/Testing)

```bash
# Build all service images
make docker-build

# Run everything
docker compose up -d
```

### AWS Production (Recommended)

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for full production deployment guide including:
- AWS ECS Fargate setup
- RDS PostgreSQL (Multi-AZ)
- ElastiCache Redis
- Cloudflare CDN/WAF
- M-Pesa production integration
- BCLB compliance setup
- Load testing with k6

### Production Checklist

- [ ] Change all default passwords
- [ ] Set `ENVIRONMENT=production`
- [ ] Set `DB_SSL_MODE=require`
- [ ] Generate strong `JWT_SECRET`
- [ ] Configure M-Pesa production credentials
- [ ] Set up SSL/TLS (Cloudflare or AWS ACM)
- [ ] Enable rate limiting
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Configure backups (RDS automated + manual)
- [ ] Load test at 2x expected peak

---

## 🎨 Next Steps: Frontend

This is a **backend-only** project. To build a complete platform like hattrick.bet, you need a frontend.

### Recommended Frontend Stack

| Layer | Technology |
|-------|------------|
| **Framework** | Next.js 14 (React) |
| **Styling** | Tailwind CSS + shadcn/ui |
| **State** | Zustand or Jotai |
| **API Client** | tRPC or React Query |
| **Auth** | NextAuth.js or custom JWT |
| **WebSocket** | Native WebSocket API |
| **3D/Animation** | Three.js + GSAP |
| **Design System** | Stripe/Linear/Vercel style |

### Frontend Pages Needed

1. **Home** — Hero + featured matches + crash game
2. **Sports** — Match list + odds + bet slip
3. **Live** — In-play matches with live odds
4. **Crash Game** — Aviator-style game with WebSocket
5. **Wallet** — Balance + deposit/withdraw
6. **Account** — Profile + KYC + settings
7. **History** — Bet history + transactions

### Design Reference

For premium design (like hattrick.bet), use:
- **Stripe-style** design system (clean, modern, trustworthy)
- **Dark mode** (standard for betting sites)
- **3D elements** (Three.js for crash game animation)
- **Micro-interactions** (GSAP for smooth UX)

---

## 📁 Project Structure

```
lets-bet/
├── cmd/                    # Service entry points
│   ├── gateway/            # API Gateway
│   ├── wallet/             # Wallet Service
│   ├── engine/             # Betting Engine
│   ├── settlement/         # Settlement Service
│   ├── games/              # Crash Game Service
│   └── migrate/            # Migration Tool
├── internal/
│   ├── core/               # Shared business logic
│   │   ├── domain/         # Entities (User, Bet, Transaction)
│   │   └── usecase/        # Business logic
│   ├── infrastructure/     # Shared infrastructure
│   │   ├── config/         # Configuration
│   │   ├── database/       # PostgreSQL
│   │   ├── http/           # HTTP handlers
│   │   ├── events/         # NATS event bus
│   │   ├── kyc/            # KYC integration
│   │   ├── logging/        # Structured logging
│   │   └── metrics/        # Prometheus
│   └── tenant/             # Country-specific code
│       └── ke/             # Kenya adapters
├── migrations/             # Database migrations
├── deployments/            # Docker/K8s configs
├── docs/                   # Documentation
│   ├── API.md              # Full API reference
│   ├── DEPLOYMENT.md       # Production deployment
│   ├── PROJECT_OVERVIEW.md # Architecture details
│   └── TESTING.md          # Testing guide
├── docker-compose.yml      # Local infrastructure
├── Makefile                # Build commands
├── go.mod                  # Go dependencies
└── .env.example            # Environment template
```

---

## 🧪 Testing

```bash
# Run all tests
make test

# Run with verbose output
go test -v ./...

# Run specific package
go test -v ./internal/core/usecase/...
```

---

## 📊 Monitoring

- **Health:** `GET /healthz`
- **Metrics:** `GET /metrics` (Prometheus format)
- **Structured Logs:** JSON format with request tracing

---

## 📝 License

See [LICENSE](LICENSE) for details.

---

## 🔗 Links

- **GitHub:** https://github.com/nutcas3/lets-bet
- **M-Pesa Daraja API:** https://developer.safaricom.co.ke
- **Sportradar:** https://developer.sportradar.com
- **Smile ID:** https://smileid.com
- **Flutterwave:** https://developer.flutterwave.com
