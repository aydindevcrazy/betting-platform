# 🎨 Frontend Development Plan — Betting Platform

> **Design System:** Linear (dark mode) + Stripe (landing)  
> **Stack:** Next.js 14 + TypeScript + Tailwind + shadcn/ui + Three.js + GSAP  
> **Target:** Premium, dark-mode-first, mobile-first, 60fps animations

---

## 📐 Design Strategy

### Why Linear + Stripe?

| Page | Design System | Reason |
|------|--------------|--------|
| **Landing / Marketing** | Stripe | Trust, premium fintech feel, conversion-optimized |
| **Dashboard / App** | Linear | Dark mode native, data-dense, developer-grade precision |
| **Crash Game** | Custom (Three.js) | 3D immersive experience, unique signature feature |

### Design Tokens (Unified)

```css
/* ═══════════════════════════════════════════
   DESIGN TOKENS — Linear Dark + Stripe Accent
   ═══════════════════════════════════════════ */

:root {
  /* ── Backgrounds (Linear Dark) ── */
  --bg-base: #08090a;           /* Marketing black */
  --bg-panel: #0f1011;          /* Sidebar/panel */
  --bg-surface: #191a1b;        /* Cards, elevated */
  --bg-hover: #28282c;          /* Hover state */
  --bg-overlay: rgba(0,0,0,0.85);

  /* ── Text (Linear) ── */
  --text-primary: #f7f8f8;      /* Near-white */
  --text-secondary: #d0d6e0;    /* Silver-gray */
  --text-tertiary: #8a8f98;     /* Muted */
  --text-quaternary: #62666d;   /* Subtle */

  /* ── Accent (Stripe Purple + Betting Green) ── */
  --accent-brand: #5e6ad2;      /* Linear indigo (primary) */
  --accent-brand-hover: #828fff;
  --accent-stripe: #533afd;     /* Stripe purple (CTAs) */
  --accent-green: #00c853;      /* Wins, success */
  --accent-red: #ff1744;        /* Losses, errors */
  --accent-gold: #ffd600;       /* Bonus, VIP */

  /* ── Borders (Linear translucent) ── */
  --border-subtle: rgba(255,255,255,0.05);
  --border-default: rgba(255,255,255,0.08);
  --border-strong: rgba(255,255,255,0.12);

  /* ── Typography ── */
  --font-primary: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;
  --font-feature: 'cv01', 'ss03'; /* Linear's OpenType */

  /* ── Spacing (8px grid) ── */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;
  --space-8: 64px;
  --space-9: 80px;

  /* ── Radius ── */
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 12px;
  --radius-pill: 9999px;

  /* ── Shadows (dark-mode optimized) ── */
  --shadow-sm: rgba(0,0,0,0.03) 0px 1.2px 0px 0px;
  --shadow-md: rgba(0,0,0,0.2) 0px 0px 0px 1px;
  --shadow-lg: rgba(0,0,0,0.4) 0px 2px 4px;
  --shadow-dialog: rgba(0,0,0,0) 0px 8px 2px,
                   rgba(0,0,0,0.01) 0px 5px 2px,
                   rgba(0,0,0,0.04) 0px 3px 2px,
                   rgba(0,0,0,0.07) 0px 1px 1px,
                   rgba(0,0,0,0.08) 0px 0px 1px;

  /* ── Transitions ── */
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --duration-fast: 150ms;
  --duration-normal: 300ms;
  --duration-slow: 500ms;
}
```

---

## 🗂️ Project Structure

```
betting-frontend/
├── app/
│   ├── (marketing)/            # Stripe-style landing
│   │   ├── page.tsx            # Home / Landing
│   │   ├── pricing/page.tsx    # Pricing (if applicable)
│   │   └── layout.tsx          # Marketing layout
│   │
│   ├── (app)/                  # Linear-style dashboard
│   │   ├── layout.tsx          # App shell (sidebar + header)
│   │   ├── dashboard/page.tsx  # Dashboard home
│   │   ├── sports/
│   │   │   ├── page.tsx        # Sports list
│   │   │   └── [id]/page.tsx   # Match detail
│   │   ├── live/page.tsx       # Live betting
│   │   ├── crash/
│   │   │   ├── page.tsx        # Crash game lobby
│   │   │   └── [id]/page.tsx   # Active crash game
│   │   ├── wallet/
│   │   │   ├── page.tsx        # Wallet overview
│   │   │   ├── deposit/page.tsx
│   │   │   └── withdraw/page.tsx
│   │   ├── account/
│   │   │   ├── page.tsx        # Profile
│   │   │   ├── kyc/page.tsx    # KYC verification
│   │   │   └── settings/page.tsx
│   │   └── history/page.tsx    # Bet history
│   │
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   └── layout.tsx          # Auth layout (minimal)
│   │
│   ├── api/                    # API routes (proxy to backend)
│   │   ├── auth/[...nextauth]/route.ts
│   │   └── proxy/[...path]/route.ts
│   │
│   ├── layout.tsx              # Root layout
│   ├── globals.css             # Global styles + Tailwind
│   └── providers.tsx           # React Query + Auth providers
│
├── components/
│   ├── ui/                     # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── input.tsx
│   │   ├── select.tsx
│   │   ├── tabs.tsx
│   │   ├── toast.tsx
│   │   └── skeleton.tsx
│   │
│   ├── layout/
│   │   ├── sidebar.tsx         # App sidebar (Linear-style)
│   │   ├── header.tsx          # Top header
│   │   ├── mobile-nav.tsx      # Bottom nav (mobile)
│   │   └── marketing-header.tsx # Landing header (Stripe-style)
│   │
│   ├── betting/
│   │   ├── match-card.tsx      # Match display card
│   │   ├── odds-button.tsx     # Odds selection button
│   │   ├── bet-slip.tsx        # Floating bet slip
│   │   ├── bet-slip-item.tsx   # Single bet in slip
│   │   ├── live-indicator.tsx  # Live match indicator
│   │   └── scoreboard.tsx      # Mini scoreboard
│   │
│   ├── crash/
│   │   ├── crash-canvas.tsx    # Three.js canvas
│   │   ├── crash-plane.tsx     # 3D airplane
│   │   ├── crash-sky.tsx       # Sky/environment
│   │   ├── crash-particles.tsx # Particle effects
│   │   ├── multiplier-display.tsx # Large multiplier
│   │   ├── bet-controls.tsx    # Bet input + auto-cashout
│   │   ├── cashout-button.tsx  # Big green button
│   │   ├── game-history.tsx    # Last 20 rounds
│   │   └── fairness-verify.tsx # Provably fair
│   │
│   ├── wallet/
│   │   ├── balance-card.tsx    # Balance display
│   │   ├── deposit-form.tsx    # M-Pesa deposit
│   │   ├── withdraw-form.tsx   # M-Pesa withdraw
│   │   ├── transaction-list.tsx # Transaction history
│   │   └── limits-display.tsx  # Deposit limits
│   │
│   └── shared/
│       ├── loading-spinner.tsx
│       ├── empty-state.tsx
│       ├── error-boundary.tsx
│       └── responsive.tsx
│
├── hooks/
│   ├── useAuth.ts              # Authentication
│   ├── useWebSocket.ts         # WebSocket connection
│   ├── useCrashGame.ts         # Crash game state
│   ├── useBetSlip.ts           # Bet slip management
│   ├── useWallet.ts            # Wallet operations
│   ├── useMatches.ts           # Match data
│   └── useMediaQuery.ts        # Responsive hooks
│
├── lib/
│   ├── api.ts                  # API client (fetch wrapper)
│   ├── websocket.ts            # WebSocket manager
│   ├── auth.ts                 # Auth utilities
│   ├── utils.ts                # General utilities
│   ├── constants.ts            # App constants
│   └── validations.ts          # Zod schemas
│
├── stores/
│   ├── auth-store.ts           # Auth state (Zustand)
│   ├── bet-slip-store.ts       # Bet slip state
│   ├── crash-store.ts          # Crash game state
│   ├── wallet-store.ts         # Wallet state
│   └── ui-store.ts             # UI state (modals, sidebar)
│
├── types/
│   ├── api.ts                  # API types
│   ├── models.ts               # Domain models
│   └── index.ts                # Re-exports
│
├── public/
│   ├── fonts/                  # Custom fonts
│   ├── images/
│   │   ├── logo.svg
│   │   ├── hero/               # Landing hero images
│   │   └── icons/              # App icons
│   └── sounds/                 # Crash game sounds
│       ├── tick.mp3
│       ├── cashout.mp3
│       └── crash.mp3
│
├── tailwind.config.ts
├── next.config.ts
├── tsconfig.json
├── package.json
└── .env.local
```

---

## 📄 Page-by-Page Breakdown

### 1. Landing Page (`/`) — Stripe Style

```
┌─────────────────────────────────────────────────────┐
│  HEADER: Logo | Features | Pricing | Login | CTA   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  HERO SECTION                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  "Bet Smarter. Win Bigger."                 │   │
│  │  Subtitle: Africa's most trusted platform   │   │
│  │  [Start Betting] [Watch Demo]               │   │
│  │                                             │   │
│  │  ┌─────────────────────────────────────┐   │   │
│  │  │  Dashboard Preview (animated)       │   │   │
│  │  │  Blue-tinted shadow, rounded 8px    │   │   │
│  │  └─────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
├─────────────────────────────────────────────────────┤
│  FEATURES SECTION (dark bg #1c1e54)                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 🏆 Sports│ │ ✈️ Crash │ │ 💰 M-Pesa│           │
│  │  Betting │ │  Game    │ │  Payments│           │
│  └──────────┘ └──────────┘ └──────────┘           │
│                                                     │
├─────────────────────────────────────────────────────┤
│  CRASH GAME PREVIEW (animated demo)                 │
│  ┌─────────────────────────────────────────────┐   │
│  │  3D airplane flying, multiplier counting    │   │
│  │  "Provably Fair • Instant Payouts"          │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
├─────────────────────────────────────────────────────┤
│  STATS SECTION                                      │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐      │
│  │ 50K+   │ │ KES 1B+│ │ <100ms │ │ 99.9%  │      │
│  │ Users  │ │ Paid   │ │ Speed  │ │ Uptime │      │
│  └────────┘ └────────┘ └────────┘ └────────┘      │
│                                                     │
├─────────────────────────────────────────────────────┤
│  HOW IT WORKS                                       │
│  ① Sign Up → ② Deposit → ③ Bet → ④ Withdraw       │
│                                                     │
├─────────────────────────────────────────────────────┤
│  CTA SECTION (dark)                                 │
│  "Ready to start? Join 50,000+ winners"            │
│  [Create Free Account]                              │
│                                                     │
├─────────────────────────────────────────────────────┤
│  FOOTER                                             │
│  Logo | Sports | Casino | Payments | Legal | Social│
└─────────────────────────────────────────────────────┘
```

**Key Components:**
- Animated hero with floating dashboard preview
- Feature cards with hover effects (shadow intensify)
- Live crash game demo (miniature 3D)
- Stats counter animation (GSAP)
- Trust badges (M-Pesa, BCLB, etc.)

---

### 2. Dashboard (`/dashboard`) — Linear Style

```
┌─────────────────────────────────────────────────────┐
│  HEADER: Logo | Search (Cmd+K) | Balance | Avatar  │
├──────────┬──────────────────────────────────────────┤
│          │                                          │
│ SIDEBAR  │  MAIN CONTENT                            │
│          │                                          │
│ 🏠 Home  │  ┌──────────────────────────────────┐   │
│ 🏆 Sports│  │  QUICK STATS                     │   │
│ 📡 Live  │  │  Balance | Open Bets | Winnings  │   │
│ ✈️ Crash │  └──────────────────────────────────┘   │
│ 💰 Wallet│                                          │
│ 📋 History│ ┌──────────────────────────────────┐   │
│ 👤 Account│ │  FEATURED MATCHES                │   │
│ ⚙️ Settings│ │  [Match 1] [Match 2] [Match 3]  │   │
│          │  └──────────────────────────────────┘   │
│          │                                          │
│          │  ┌──────────────────────────────────┐   │
│          │  │  LIVE NOW                        │   │
│          │  │  🔴 Gor Mahia 2-1 AFC Leopards   │   │
│          │  │  🔴 Liverpool 0-0 Man City       │   │
│          │  └──────────────────────────────────┘   │
│          │                                          │
│          │  ┌──────────────────────────────────┐   │
│          │  │  CRASH GAME MINI                 │   │
│          │  │  Current: 3.42x | Next in 5s     │   │
│          │  │  [Play Now →]                    │   │
│          │  └──────────────────────────────────┘   │
│          │                                          │
├──────────┴──────────────────────────────────────────┤
│  MOBILE: Bottom nav (Home | Sports | Crash | Wallet)│
└─────────────────────────────────────────────────────┘
```

---

### 3. Sports Betting (`/sports`) — Linear Style

```
┌─────────────────────────────────────────────────────┐
│  HEADER                                             │
├──────────┬──────────────────────────────────────────┤
│          │  FILTER BAR                              │
│ SIDEBAR  │  [Football ▼] [Today ▼] [League ▼]     │
│          │                                          │
│ Sports   │  ┌──────────────────────────────────┐   │
│ ⚽ Foot  │  │  MATCH CARD                      │   │
│ 🏀 Bask  │  │  ┌────────────────────────────┐  │   │
│ 🎾 Tenn  │  │  │ Gor Mahia vs AFC Leopards  │  │   │
│ 🏏 Crick │  │  │ Today, 15:00 | KPL         │  │   │
│          │  │  │                            │  │   │
│ Leagues  │  │  │  [1.85]  [3.20]  [4.50]   │  │   │
│ 🇰🇪 KPL  │  │  │   Home    Draw    Away     │  │   │
│ 🏴󠁧󠁢󠁥󠁮󠁧󠁿 EPL  │  │  └────────────────────────────┘  │   │
│ 🇪🇸 LaLi │  └──────────────────────────────────┘   │
│          │                                          │
│          │  ┌──────────────────────────────────┐   │
│          │  │  MATCH CARD (Live)               │   │
│          │  │  ┌────────────────────────────┐  │   │
│          │  │  │ 🔴 LIVE | 67'              │  │   │
│          │  │  │ Liverpool 2 - 1 Man City    │  │   │
│          │  │  │ [2.10]  [3.40]  [2.80]     │  │   │
│          │  │  │  ↑ Odds changed              │  │   │
│          │  │  └────────────────────────────┘  │   │
│          │  └──────────────────────────────────┘   │
│          │                                          │
│          │  ┌──────────────────────────────────┐   │
│          │  │  📱 BET SLIP (floating)          │   │
│          │  │  ┌────────────────────────────┐  │   │
│          │  │  │ Gor Mahia vs AFC Leopards  │  │   │
│          │  │  │ Home @ 1.85                │  │   │
│          │  │  │ Stake: [100    ] KES       │  │   │
│          │  │  │ Potential: 185 KES         │  │   │
│          │  │  │ Tax: -15 KES                │  │   │
│          │  │  │                            │  │   │
│          │  │  │ [PLACE BET]                │  │   │
│          │  │  └────────────────────────────┘  │   │
│          │  └──────────────────────────────────┘   │
└──────────┴──────────────────────────────────────────┘
```

**Match Card States:**
- Default: Dark surface, odds buttons visible
- Selected: Brand indigo border, checkmark
- Live: Red pulse indicator, live score
- Started: Locked, grayed out

---

### 4. Crash Game (`/crash/[id]`) — Custom 3D

```
┌─────────────────────────────────────────────────────┐
│  ← Back                    Balance: 5,000 KES      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │                                             │   │
│  │           3D CANVAS (Three.js)              │   │
│  │                                             │   │
│  │              ☁️  ☁️                          │   │
│  │                                             │   │
│  │           ✈️  ← airplane                    │   │
│  │          ╱                                  │   │
│  │         ╱  trail particles                  │   │
│  │        ╱                                    │   │
│  │                                             │   │
│  │     ═══════════════════════════             │   │
│  │           3.42x  ← multiplier               │   │
│  │                                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌──────────────────┐  ┌────────────────────────┐  │
│  │ BET CONTROLS     │  │ GAME HISTORY           │  │
│  │                  │  │                        │  │
│  │ Amount: [100  ]  │  │ 2.34x  1.89x  4.56x   │  │
│  │ Auto:   [2.00x]  │  │ 1.12x  3.21x  1.05x   │  │
│  │                  │  │ 5.67x  1.98x  2.34x   │  │
│  │ [PLACE BET]      │  │                        │  │
│  │                  │  │ Provably Fair ✅       │  │
│  │ ─────────────── │  │ [Verify]              │  │
│  │                  │  │                        │  │
│  │ ┌──────────────┐ │  └────────────────────────┘  │
│  │ │  💰 CASHOUT  │ │                              │
│  │ │   342 KES    │ │                              │
│  │ │   @ 3.42x    │ │                              │
│  │ └──────────────┘ │                              │
│  └──────────────────┘                              │
│                                                     │
│  ROUND INFO                                         │
│  Round #12345 | Server Seed: abc... | Verify →     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Crash Game States:**

| State | Visual | Interaction |
|-------|--------|-------------|
| **Waiting** | Countdown timer, "Next round in 5s" | Bet input enabled |
| **Flying** | Airplane ascending, multiplier counting | Cashout button active |
| **Crashed** | Explosion particles, red flash | Show crash point, results |
| **Cashed Out** | Green flash, win amount | Show profit |

**Animation Details:**
- Airplane: Smooth GSAP animation along bezier curve
- Multiplier: Counting animation (1.00x → crash_point)
- Particles: Trail behind airplane, explosion on crash
- Sound: Tick sound (each 0.5x), cashout ding, crash boom

---

### 5. Wallet (`/wallet`) — Linear Style

```
┌─────────────────────────────────────────────────────┐
│  WALLET                                    👤 Menu │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  BALANCE CARD                                │   │
│  │                                              │   │
│  │  Available Balance                           │   │
│  │  KES 5,000.00                                │   │
│  │                                              │   │
│  │  Pending Withdrawals: KES 0.00              │   │
│  │                                              │   │
│  │  [+ Deposit]          [− Withdraw]          │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  DEPOSIT LIMITS                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  Daily:   15,000 / 50,000 KES  ████░░░░░   │   │
│  │  Weekly:  45,000 / 200,000 KES  ██░░░░░░░   │   │
│  │  Monthly: 120,000 / 500,000 KES ██░░░░░░░   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  TRANSACTION HISTORY                                │
│  ┌─────────────────────────────────────────────┐   │
│  │  Date        Type     Amount    Balance     │   │
│  │  ──────────  ───────  ────────  ──────────  │   │
│  │  Jan 15      Deposit  +500.00   5,000.00   │   │
│  │  Jan 14      Bet      -100.00   4,500.00   │   │
│  │  Jan 14      Win      +185.00   4,600.00   │   │
│  │  Jan 13      Withdraw -1,000    4,415.00   │   │
│  │  ...                                        │   │
│  │                                             │   │
│  │  [Load More]                                │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### 6. Account (`/account`) — Linear Style

```
┌─────────────────────────────────────────────────────┐
│  ACCOUNT                                   👤 Menu │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  PROFILE                                     │   │
│  │  ┌──────┐                                   │   │
│  │  │ 👤   │  John Doe                         │   │
│  │  │      │  0712345678                       │   │
│  │  └──────┘  Member since Jan 2026            │   │
│  │                                              │   │
│  │  [Edit Profile]                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  KYC STATUS                                         │
│  ┌─────────────────────────────────────────────┐   │
│  │  ✅ Identity Verified (National ID)         │   │
│  │  ✅ Phone Verified                          │   │
│  │  ⏳ Address Verification (Pending)          │   │
│  │                                              │   │
│  │  [Complete KYC →]                           │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  RESPONSIBLE GAMING                                 │
│  ┌─────────────────────────────────────────────┐   │
│  │  Self-Exclusion: Not Active                 │   │
│  │  Deposit Limits: Active                     │   │
│  │  Time Reminders: Off                        │   │
│  │                                              │   │
│  │  [Manage Limits]  [Self-Exclude]            │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  BET HISTORY                                        │
│  ┌─────────────────────────────────────────────┐   │
│  │  [Active] [Settled] [All]                   │   │
│  │                                              │   │
│  │  Jan 15 | Gor Mahia vs AFC | Home @ 1.85   │   │
│  │  Stake: 100 KES | Potential: 185 KES        │   │
│  │  Status: 🟢 Active                          │   │
│  │                                              │   │
│  │  Jan 14 | Liverpool vs MC | Over @ 2.10    │   │
│  │  Stake: 200 KES | Won: 420 KES              │   │
│  │  Status: ✅ Won                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🔄 State Management (Zustand)

### Auth Store
```typescript
interface AuthStore {
  user: User | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  login: (phone: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => void;
  refresh: () => Promise<void>;
}
```

### Bet Slip Store
```typescript
interface BetSlipStore {
  selections: BetSelection[];
  stake: number;
  betType: 'single' | 'multi';
  potentialWinnings: number;
  tax: number;
  addSelection: (selection: BetSelection) => void;
  removeSelection: (matchId: string) => void;
  updateOdds: (matchId: string, newOdds: number) => void;
  setStake: (amount: number) => void;
  setBetType: (type: 'single' | 'multi') => void;
  clear: () => void;
  placeBet: () => Promise<BetResult>;
}
```

### Crash Store
```typescript
interface CrashStore {
  gameId: string | null;
  status: 'idle' | 'waiting' | 'flying' | 'crashed';
  multiplier: number;
  crashPoint: number | null;
  betAmount: number | null;
  autoCashout: number | null;
  cashedOut: boolean;
  cashoutMultiplier: number | null;
  winnings: number | null;
  history: CrashRound[];
  serverSeed: string | null;
  commitment: string | null;
  connect: (gameId: string) => void;
  placeBet: (amount: number, autoCashout?: number) => void;
  cashout: () => void;
  verify: (roundId: string) => Promise<boolean>;
}
```

### Wallet Store
```typescript
interface WalletStore {
  balance: number;
  currency: string;
  pendingWithdrawals: number;
  transactions: Transaction[];
  limits: DepositLimits;
  deposit: (amount: number, phone: string) => Promise<void>;
  withdraw: (amount: number, phone: string) => Promise<void>;
  refreshBalance: () => Promise<void>;
  loadTransactions: (page: number) => Promise<void>;
}
```

---

## 🔌 WebSocket Architecture

### Connection Manager
```typescript
class WebSocketManager {
  private ws: WebSocket | null;
  private reconnectAttempts: number;
  private maxReconnectAttempts: number;
  private listeners: Map<string, Set<Function>>;
  
  connect(url: string, token: string): void;
  disconnect(): void;
  subscribe(event: string, callback: Function): void;
  unsubscribe(event: string, callback: Function): void;
  send(message: object): void;
  private reconnect(): void;
}
```

### Events
```typescript
// Incoming (Server → Client)
interface WSIncoming {
  'crash:tick': { multiplier: number; elapsed: number };
  'crash:crash': { crashPoint: number; roundId: string };
  'crash:round_start': { roundId: string; commitment: string };
  'crash:bet_confirmed': { betId: string; amount: number };
  'crash:cashout_result': { multiplier: number; winnings: number };
  'match:odds_update': { matchId: string; odds: Odds };
  'match:score_update': { matchId: string; score: Score };
  'match:status_change': { matchId: string; status: MatchStatus };
  'wallet:balance_update': { balance: number };
  'wallet:transaction': { transaction: Transaction };
}

// Outgoing (Client → Server)
interface WSOutgoing {
  'crash:join': { gameId: string; token: string };
  'crash:place_bet': { amount: number; auto_cashout?: number };
  'crash:cashout': {};
  'match:subscribe': { matchIds: string[] };
  'match:unsubscribe': { matchIds: string[] };
}
```

---

## 🎬 Animation Plan (GSAP)

### Page Transitions
```typescript
// Fade + slide up on enter
gsap.from(pageElement, {
  opacity: 0,
  y: 20,
  duration: 0.4,
  ease: 'power3.out'
});

// Fade out on exit
gsap.to(pageElement, {
  opacity: 0,
  y: -10,
  duration: 0.2,
  ease: 'power3.in'
});
```

### Crash Game Animations
```typescript
// Airplane flight path (bezier curve)
gsap.to(plane.position, {
  x: targetX,
  y: targetY,
  duration: flightDuration,
  ease: 'none',
  onUpdate: () => {
    // Update trail particles
    trailParticles.emit(plane.position);
  }
});

// Multiplier counting
gsap.to(multiplierObj, {
  value: crashPoint,
  duration: flightDuration,
  ease: 'none',
  onUpdate: () => {
    display.textContent = multiplierObj.value.toFixed(2) + 'x';
  }
});

// Crash explosion
gsap.to(explosion, {
  scale: 3,
  opacity: 0,
  duration: 0.5,
  ease: 'power2.out'
});

// Cashout celebration
gsap.from(cashoutText, {
  scale: 0.5,
  opacity: 0,
  duration: 0.3,
  ease: 'back.out(1.7)'
});
```

### UI Micro-interactions
```typescript
// Odds button hover
gsap.to(button, {
  scale: 1.05,
  duration: 0.15,
  ease: 'power2.out'
});

// Bet slip slide in
gsap.from(betSlip, {
  x: 300,
  opacity: 0,
  duration: 0.3,
  ease: 'power3.out'
});

// Balance update flash
gsap.from(balanceElement, {
  color: '#00c853',
  duration: 0.3,
  yoyo: true,
  repeat: 1
});

// Live indicator pulse
gsap.to(liveDot, {
  scale: 1.3,
  opacity: 0.5,
  duration: 0.8,
  repeat: -1,
  yoyo: true,
  ease: 'sine.inOut'
});
```

---

## 📱 Responsive Breakpoints

| Breakpoint | Width | Layout |
|------------|-------|--------|
| `xs` | < 480px | Single column, bottom nav, full-width cards |
| `sm` | 480-640px | Single column, bottom nav, compact cards |
| `md` | 640-768px | 2-column grids, bottom nav |
| `lg` | 768-1024px | Sidebar collapses to icons, 2-column |
| `xl` | 1024-1280px | Full sidebar, 3-column grids |
| `2xl` | > 1280px | Full layout, max-width container |

### Mobile-First Strategy
- Bottom navigation (5 tabs: Home, Sports, Crash, Wallet, Account)
- Floating bet slip (bottom sheet)
- Swipeable match cards
- Touch-optimized buttons (min 44px hit target)
- Pull-to-refresh on lists
- Skeleton loading states

---

## 🧪 Testing Strategy

### Unit Tests (Vitest)
- Store logic (Zustand)
- Utility functions
- Validation schemas (Zod)

### Component Tests (React Testing Library)
- Button states (default, hover, loading, disabled)
- Form validation
- Modal open/close

### Integration Tests (Playwright)
- Auth flow (register → login → dashboard)
- Bet placement flow
- Deposit/withdrawal flow
- Crash game flow (mock WebSocket)

### E2E Tests
- Full user journey: Register → Deposit → Bet → Win → Withdraw
- Multi-bet placement
- Live betting
- KYC verification

---

## 📦 Dependencies

```json
{
  "dependencies": {
    "next": "^14.2.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "typescript": "^5.4.0",
    
    "tailwindcss": "^3.4.0",
    "@radix-ui/react-dialog": "^1.0.0",
    "@radix-ui/react-select": "^2.0.0",
    "@radix-ui/react-tabs": "^1.0.0",
    "@radix-ui/react-toast": "^1.0.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.3.0",
    "lucide-react": "^0.378.0",
    
    "zustand": "^4.5.0",
    "@tanstack/react-query": "^5.32.0",
    "axios": "^1.6.0",
    "swr": "^2.2.0",
    
    "three": "^0.163.0",
    "@react-three/fiber": "^8.16.0",
    "@react-three/drei": "^9.105.0",
    "gsap": "^3.12.0",
    
    "recharts": "^2.12.0",
    "react-hook-form": "^7.51.0",
    "zod": "^3.23.0",
    "@hookform/resolvers": "^3.3.0",
    
    "next-auth": "^4.24.0",
    "framer-motion": "^11.1.0",
    "date-fns": "^3.6.0",
    "sonner": "^1.4.0"
  },
  "devDependencies": {
    "vitest": "^1.6.0",
    "@testing-library/react": "^15.0.0",
    "@playwright/test": "^1.43.0",
    "eslint": "^8.57.0",
    "prettier": "^3.2.0"
  }
}
```

---

## 🛣️ Implementation Order

### Sprint 1: Foundation (Week 1)
1. ✅ Next.js 14 project setup
2. ✅ Tailwind + shadcn/ui configuration
3. ✅ Design tokens (CSS variables)
4. ✅ Layout components (sidebar, header, mobile nav)
5. ✅ Auth pages (login, register)
6. ✅ API client + WebSocket manager
7. ✅ Zustand stores (auth, bet-slip, wallet, crash)

### Sprint 2: Core Pages (Week 2)
8. ✅ Dashboard page
9. ✅ Sports betting page (match list, filters)
10. ✅ Match detail page
11. ✅ Bet slip component
12. ✅ Wallet page (balance, transactions)
13. ✅ Deposit/Withdraw forms

### Sprint 3: Crash Game (Week 3)
14. ✅ Three.js scene setup
15. ✅ 3D airplane model + animation
16. ✅ Multiplier display
17. ✅ Bet controls + cashout button
18. ✅ WebSocket integration
19. ✅ Game history
20. ✅ Provably fair verification
21. ✅ Sound effects

### Sprint 4: Polish (Week 4)
22. ✅ Landing page (Stripe style)
23. ✅ Live betting page
24. ✅ Account page (KYC, settings)
25. ✅ Bet history
26. ✅ Animations (GSAP)
27. ✅ Responsive design
28. ✅ Loading states + error handling
29. ✅ Performance optimization

### Sprint 5: Testing & Deploy (Week 5)
30. ✅ Unit tests
31. ✅ Component tests
32. ✅ E2E tests
33. ✅ Vercel deployment
34. ✅ Custom domain
35. ✅ SSL + CDN

---

## 🎯 Success Criteria

| Metric | Target |
|--------|--------|
| Lighthouse Performance | > 90 |
| Lighthouse Accessibility | > 95 |
| First Contentful Paint | < 1.5s |
| Time to Interactive | < 3s |
| Crash Game FPS | 60fps |
| WebSocket Latency | < 100ms |
| Mobile Responsive | All pages |
| Test Coverage | > 80% |

---

*This is a living document. Update as implementation progresses.*
