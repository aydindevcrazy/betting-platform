# API Reference

## Base URL
```
Development: http://localhost:8080
Production:  https://api.yourdomain.com
```

## Authentication
All protected endpoints require a Bearer token:
```
Authorization: Bearer <jwt_token>
```

---

## Auth Endpoints

### Register
```http
POST /api/auth/register
Content-Type: application/json

{
  "phone": "0712345678",
  "password": "secure_password",
  "country_code": "KE"
}

Response 201:
{
  "user_id": "uuid",
  "access_token": "jwt",
  "refresh_token": "jwt"
}
```

### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "phone": "0712345678",
  "password": "secure_password"
}

Response 200:
{
  "access_token": "jwt",
  "refresh_token": "jwt",
  "expires_in": 900
}
```

### Refresh Token
```http
POST /api/auth/refresh
Content-Type: application/json

{
  "refresh_token": "jwt"
}

Response 200:
{
  "access_token": "jwt",
  "expires_in": 900
}
```

---

## Wallet Endpoints

### Get Balance
```http
GET /api/wallet/balance
Authorization: Bearer <token>

Response 200:
{
  "balance": 5000.00,
  "currency": "KES",
  "pending_withdrawals": 0
}
```

### Deposit (M-Pesa STK Push)
```http
POST /api/wallet/deposit
Authorization: Bearer <token>
Content-Type: application/json

{
  "amount": 500,
  "phone": "0712345678"
}

Response 200:
{
  "transaction_id": "uuid",
  "status": "pending",
  "message": "Check your phone for M-Pesa prompt"
}
```

### Withdraw (M-Pesa B2C)
```http
POST /api/wallet/withdraw
Authorization: Bearer <token>
Content-Type: application/json

{
  "amount": 1000,
  "phone": "0712345678"
}

Response 200:
{
  "transaction_id": "uuid",
  "status": "processing",
  "message": "Withdrawal initiated. Funds will arrive in ~30 seconds"
}
```

### Transaction History
```http
GET /api/wallet/transactions?page=1&limit=20
Authorization: Bearer <token>

Response 200:
{
  "transactions": [
    {
      "id": "uuid",
      "type": "credit",
      "amount": 500.00,
      "balance_before": 4500.00,
      "balance_after": 5000.00,
      "reference": "QKJ1234ABC",
      "created_at": "2026-01-15T10:30:00Z"
    }
  ],
  "total": 45,
  "page": 1,
  "limit": 20
}
```

---

## Betting Endpoints

### List Matches
```http
GET /api/matches?sport=football&status=upcoming&page=1
Authorization: Bearer <token>

Response 200:
{
  "matches": [
    {
      "id": "uuid",
      "home_team": "Gor Mahia",
      "away_team": "AFC Leopards",
      "league": "Kenyan Premier League",
      "start_time": "2026-01-15T15:00:00Z",
      "odds": {
        "home": 1.85,
        "draw": 3.20,
        "away": 4.50
      }
    }
  ]
}
```

### Get Match Details
```http
GET /api/matches/{match_id}
Authorization: Bearer <token>

Response 200:
{
  "id": "uuid",
  "home_team": "Gor Mahia",
  "away_team": "AFC Leopards",
  "league": "Kenyan Premier League",
  "start_time": "2026-01-15T15:00:00Z",
  "markets": [
    {
      "name": "1X2",
      "selections": [
        {"name": "Home", "odds": 1.85},
        {"name": "Draw", "odds": 3.20},
        {"name": "Away", "odds": 4.50}
      ]
    },
    {
      "name": "Over/Under 2.5",
      "selections": [
        {"name": "Over", "odds": 2.10},
        {"name": "Under", "odds": 1.75}
      ]
    }
  ]
}
```

### Place Bet (Single)
```http
POST /api/bets
Authorization: Bearer <token>
Content-Type: application/json

{
  "bet_type": "single",
  "stake": 100,
  "selections": [
    {
      "match_id": "uuid",
      "market": "1X2",
      "selection": "Home",
      "odds": 1.85
    }
  ]
}

Response 201:
{
  "bet_id": "uuid",
  "status": "active",
  "stake": 100.00,
  "potential_winnings": 185.00,
  "tax_deducted": 15.00,
  "net_stake": 85.00
}
```

### Place Bet (Multi/Accumulator)
```http
POST /api/bets
Authorization: Bearer <token>
Content-Type: application/json

{
  "bet_type": "multi",
  "stake": 200,
  "selections": [
    {"match_id": "uuid1", "market": "1X2", "selection": "Home", "odds": 1.85},
    {"match_id": "uuid2", "market": "1X2", "selection": "Away", "odds": 2.50},
    {"match_id": "uuid3", "market": "O/U", "selection": "Over", "odds": 1.90}
  ]
}

Response 201:
{
  "bet_id": "uuid",
  "status": "active",
  "stake": 200.00,
  "combined_odds": 8.78,
  "potential_winnings": 1755.00,
  "tax_deducted": 30.00,
  "net_stake": 170.00
}
```

### Bet History
```http
GET /api/bets?status=active&page=1
Authorization: Bearer <token>

Response 200:
{
  "bets": [
    {
      "id": "uuid",
      "type": "single",
      "status": "active",
      "stake": 100.00,
      "potential_winnings": 185.00,
      "selections": [...],
      "created_at": "2026-01-15T10:30:00Z"
    }
  ]
}
```

---

## Crash Game Endpoints

### WebSocket Connection
```javascript
const ws = new WebSocket('ws://localhost:8080/ws/games/crash/{game_id}');

// Connection message
ws.onopen = () => {
  ws.send(JSON.stringify({
    action: 'join',
    token: 'jwt_token'
  }));
};

// Game events
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  switch(data.type) {
    case 'round_start':
      // New round starting
      console.log('Round:', data.round_id);
      console.log('Commitment:', data.commitment);
      break;
      
    case 'tick':
      // Multiplier update (every 100ms)
      console.log('Multiplier:', data.multiplier + 'x');
      break;
      
    case 'crash':
      // Round ended
      console.log('Crashed at:', data.crash_point + 'x');
      break;
      
    case 'bet_confirmed':
      // Bet placed successfully
      console.log('Bet confirmed:', data.bet_id);
      break;
      
    case 'cashout_result':
      // Cashout completed
      console.log('Cashed out at:', data.cashout_multiplier + 'x');
      console.log('Winnings:', data.winnings);
      break;
  }
};
```

### Place Bet (via WebSocket)
```json
{
  "action": "place_bet",
  "amount": 100,
  "auto_cashout": 2.0
}
```

### Cashout (via WebSocket)
```json
{
  "action": "cashout"
}
```

### Provably Fair Commitment
```http
POST /api/fairness/commitment
Authorization: Bearer <token>
Content-Type: application/json

{
  "server_seed_hash": "sha256_of_server_seed"
}

Response 200:
{
  "round_id": "uuid",
  "commitment": "sha256_hash",
  "client_seed": "user_provided_or_random"
}
```

### Verify Round Fairness
```http
GET /api/fairness/verify/{round_id}
Authorization: Bearer <token>

Response 200:
{
  "round_id": "uuid",
  "server_seed": "revealed_after_round",
  "client_seed": "...",
  "round_number": 12345,
  "crash_point": 3.42,
  "verification_hash": "sha256",
  "is_valid": true
}
```

---

## KYC & Compliance Endpoints

### Verify Identity
```http
POST /api/kyc/verify-user
Authorization: Bearer <token>
Content-Type: application/json

{
  "id_type": "NATIONAL_ID",
  "id_number": "12345678",
  "first_name": "John",
  "last_name": "Doe",
  "date_of_birth": "1990-01-15"
}

Response 200:
{
  "verification_id": "uuid",
  "status": "verified",
  "confidence_score": 0.98
}
```

### Self-Exclusion
```http
POST /api/compliance/self-exclude
Authorization: Bearer <token>
Content-Type: application/json

{
  "duration_months": 6,
  "reason": "responsible_gaming"
}

Response 200:
{
  "exclusion_id": "uuid",
  "expires_at": "2026-07-15T00:00:00Z",
  "message": "You have been self-excluded for 6 months"
}
```

### Deposit Limits
```http
GET /api/compliance/deposit-limits
Authorization: Bearer <token>

Response 200:
{
  "daily_limit": 50000,
  "daily_used": 15000,
  "weekly_limit": 200000,
  "weekly_used": 45000,
  "monthly_limit": 500000,
  "monthly_used": 120000
}
```

---

## Callbacks (Internal)

### M-Pesa Deposit Callback
```http
POST /api/mpesa/callback
Content-Type: application/json

{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "12345",
      "CheckoutRequestID": "ws_CO_12345",
      "ResultCode": 0,
      "ResultDesc": "Success",
      "CallbackMetadata": {
        "Item": [
          {"Name": "Amount", "Value": 500},
          {"Name": "MpesaReceiptNumber", "Value": "QKJ1234ABC"},
          {"Name": "PhoneNumber", "Value": 254712345678}
        ]
      }
    }
  }
}
```

### M-Pesa Withdrawal Callback
```http
POST /api/mpesa/b2c/callback
Content-Type: application/json

{
  "Result": {
    "ResultCode": 0,
    "ResultDesc": "Success",
    "TransactionID": "QKJ1234ABC",
    "OrgAccountBalance": 100000
  }
}
```

---

## Error Responses

All errors follow this format:
```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Your balance is insufficient for this bet",
    "details": {
      "current_balance": 50.00,
      "required": 100.00
    }
  }
}
```

### Error Codes
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or expired token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid request data |
| `INSUFFICIENT_BALANCE` | 400 | Not enough funds |
| `BET_LIMIT_EXCEEDED` | 400 | Stake exceeds limit |
| `ODDS_CHANGED` | 409 | Odds have changed |
| `MATCH_STARTED` | 409 | Match already started |
| `SELF_EXCLUDED` | 403 | User is self-excluded |
| `KYC_REQUIRED` | 403 | KYC verification required |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |
