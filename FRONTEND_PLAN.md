# 🎨 Frontend Plan — Betting Platform

> **This document is for Phase 2: Frontend Development**
> Backend is ready. This is the plan for building the frontend.

---

## Design Direction

**Reference sites:**
- hattrick.bet — Clean, modern, sports-focused
- bet365.com — Industry standard for sports betting
- stake.com — Modern crypto betting, great UX

**Design principles:**
- Dark mode first (industry standard for betting)
- Trust & professionalism (Stripe-style clean design)
- Fast & responsive (mobile-first)
- 3D/animated elements (Three.js for crash game)

---

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Framework** | Next.js 14 (App Router) | SSR, API routes, great DX |
| **Language** | TypeScript | Type safety |
| **Styling** | Tailwind CSS + shadcn/ui | Rapid development |
| **State** | Zustand | Lightweight, simple |
| **API** | React Query (TanStack) | Caching, refetching |
| **Auth** | Custom JWT (from backend) | Already have JWT auth |
| **WebSocket** | Native WS | Crash game real-time |
| **3D** | Three.js + React Three Fiber | Crash game animation |
| **Animation** | GSAP + Framer Motion | Smooth UX |
| **Charts** | Recharts | Odds visualization |
| **Forms** | React Hook Form + Zod | Validation |
| **i18n** | next-intl | Multi-language support |

---

## Page Structure

### 1. Home (`/`)
- Hero section with CTA
- Featured matches (live odds)
- Crash game widget (real-time multiplier)
- Popular sports sections
- Promotions/banners

### 2. Sports (`/sports`)
- Sport selector (Football, Basketball, Tennis, etc.)
- League/competition filter
- Match cards with odds
- Bet slip (floating panel)

### 3. Match (`/match/:id`)
- Match header (teams, score, time)
- Market groups (1X2, Over/Under, BTTS, etc.)
- Odds buttons (click to add to bet slip)
- Live match tracker (if in-play)

### 4. Live (`/live`)
- All in-play matches
- Live score updates (WebSocket)
- Quick bet feature
- Live match visualization

### 5. Crash Game (`/crash`)
- 3D airplane animation (Three.js)
- Multiplier display (real-time)
- Bet input + auto-cashout setting
- Cashout button (big, prominent)
- Game history (last 20 rounds)
- Provably fair verification
- Active players count

### 6. Wallet (`/wallet`)
- Balance display (real-time)
- Deposit form (M-Pesa phone + amount)
- Withdrawal form
- Transaction history table
- Bonus balance

### 7. Account (`/account`)
- Profile info
- KYC verification (upload ID)
- Self-exclusion settings
- Deposit limits
- Change password

### 8. History (`/history`)
- Bet history (filterable by status)
- Game history
- Export to CSV

---

## Component Architecture

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/             # Auth group (login, register)
│   ├── (main)/             # Main app group
│   │   ├── sports/
│   │   ├── live/
│   │   ├── crash/
│   │   ├── wallet/
│   │   ├── account/
│   │   └── history/
│   └── api/                # API routes (proxy to backend)
├── components/
│   ├── ui/                 # shadcn/ui components
│   ├── layout/             # Header, Footer, Sidebar
│   ├── sports/             # Match cards, odds buttons
│   ├── crash/              # Crash game components
│   ├── wallet/             # Deposit/withdraw forms
│   └── shared/             # Loading, Error, etc.
├── hooks/                  # Custom React hooks
│   ├── useAuth.ts
│   ├── useWebSocket.ts
│   ├── useBetSlip.ts
│   └── useOdds.ts
├── lib/                    # Utilities
│   ├── api.ts              # API client
│   ├── auth.ts             # Auth helpers
│   └── utils.ts
├── stores/                 # Zustand stores
│   ├── authStore.ts
│   ├── betSlipStore.ts
│   └── walletStore.ts
├── types/                  # TypeScript types
└── styles/                 # Global styles
```

---

## API Integration

### Base URL
```
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
NEXT_PUBLIC_WS_URL=ws://localhost:8080
```

### Auth Flow
1. User registers → POST `/auth/register`
2. User logs in → POST `/auth/login` → get JWT
3. Store JWT in httpOnly cookie
4. Refresh token automatically before expiry

### Bet Flow
1. User clicks odds → add to bet slip (Zustand store)
2. User enters stake → calculate potential win
3. User confirms → POST `/bets`
4. Show confirmation + update wallet

### Crash Game Flow
1. Page loads → GET `/games/crash/current`
2. Connect to WebSocket → receive multiplier updates
3. User places bet → POST `/games/crash/bet`
4. User clicks cashout → WS message `cashout`
5. Server responds with win amount

---

## Design System

### Colors (Dark Mode)
```css
--bg-primary: #0a0a0a;
--bg-secondary: #141414;
--bg-tertiary: #1f1f1f;
--text-primary: #ffffff;
--text-secondary: #a0a0a0;
--accent-green: #00c853;    /* Wins, positive */
--accent-red: #ff1744;      /* Losses, negative */
--accent-blue: #2979ff;     /* Links, actions */
--accent-gold: #ffd600;     /* Bonus, VIP */
--border: #2a2a2a;
```

### Typography
```css
--font-sans: 'Inter', system-ui, sans-serif;
--font-mono: 'JetBrains Mono', monospace;  /* For odds numbers */
```

### Spacing
- Base unit: 4px
- Card padding: 16px
- Section gap: 24px
- Page max-width: 1440px

---

## Crash Game 3D Design

The crash game is the signature feature. It needs to be visually stunning:

1. **3D Airplane** — Flying through clouds/stars
2. **Multiplier Curve** — Real-time graph (Recharts)
3. **Particle Effects** — Explosion on crash
4. **Sound Effects** — Engine roar, cashout ding, crash boom
5. **Screen Shake** — Subtle shake as multiplier increases

### Three.js Scene
```
Scene
├── Camera (perspective, following plane)
├── Plane (3D model, animated)
├── Skybox (starfield/clouds)
├── Particles (trail behind plane)
├── Lighting (dramatic, dynamic)
└── UI Overlay (multiplier, buttons)
```

---

## Responsive Breakpoints

| Breakpoint | Width | Layout |
|------------|-------|--------|
| Mobile | < 640px | Single column, bottom nav |
| Tablet | 640-1024px | Two columns, side nav collapsed |
| Desktop | > 1024px | Three columns, full sidebar |

---

## Performance Targets

| Metric | Target |
|--------|--------|
| First Contentful Paint | < 1.5s |
| Largest Contentful Paint | < 2.5s |
| Time to Interactive | < 3s |
| Cumulative Layout Shift | < 0.1 |
| WebSocket latency | < 100ms |
| API response (P95) | < 200ms |

---

## Development Phases

### Phase 1: Foundation (Week 1)
- [ ] Next.js project setup
- [ ] Tailwind + shadcn/ui configuration
- [ ] Auth flow (register, login, JWT)
- [ ] API client + React Query setup
- [ ] Layout (header, footer, sidebar)
- [ ] Dark mode theme

### Phase 2: Sports Betting (Week 2)
- [ ] Sports page with match list
- [ ] Odds display + bet slip
- [ ] Place bet flow
- [ ] Bet history
- [ ] Live matches page

### Phase 3: Crash Game (Week 3)
- [ ] Three.js scene setup
- [ ] Airplane animation
- [ ] WebSocket integration
- [ ] Bet/cashout flow
- [ ] Game history
- [ ] Provably fair verification

### Phase 4: Wallet & Account (Week 4)
- [ ] Wallet page
- [ ] Deposit/withdraw forms
- [ ] Transaction history
- [ ] Account settings
- [ ] KYC flow

### Phase 5: Polish (Week 5)
- [ ] Animations (GSAP + Framer Motion)
- [ ] Sound effects
- [ ] Responsive design
- [ ] Performance optimization
- [ ] Error handling
- [ ] Loading states

---

## Deployment

### Vercel (Recommended)
```bash
vercel --prod
```

### Environment Variables
```
NEXT_PUBLIC_API_URL=https://api.yourdomain.com/api/v1
NEXT_PUBLIC_WS_URL=wss://api.yourdomain.com
```

---

## Cost Estimate

| Service | Cost |
|---------|------|
| Vercel Pro | $20/month |
| Domain | $10/year |
| Total | ~$30/month |

---

**Ready to start building! 🚀**
