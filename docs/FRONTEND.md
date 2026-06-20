# Frontend Guide (Phase 2)

## Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| State | Zustand |
| API Client | React Query (TanStack) |
| 3D/Animation | Three.js + React Three Fiber + GSAP |
| Charts | Recharts |
| Forms | React Hook Form + Zod |

## Design System

### Colors (Dark Mode)
```css
--bg-primary: #0a0a0a;
--bg-secondary: #141414;
--bg-tertiary: #1f1f1f;
--text-primary: #ffffff;
--text-secondary: #a0a0a0;
--accent-green: #00c853;    /* wins, positive */
--accent-red: #ff1744;      /* losses, negative */
--accent-blue: #2979ff;     /* links, actions */
--accent-gold: #ffd600;     /* bonus, VIP */
--border: #2a2a2a;
```

### Design Reference
Use **Stripe** or **Linear** design system as reference. See `popular-web-designs` skill.

## Project Structure

```
betting-frontend/
├── app/                    # Next.js App Router
│   ├── (auth)/             # Auth group (login, register)
│   ├── (main)/             # Main app group
│   │   ├── page.tsx        # Home page
│   │   ├── sports/         # Sports betting
│   │   ├── live/           # Live betting
│   │   ├── crash/          # Crash game
│   │   ├── wallet/         # Wallet management
│   │   └── account/        # Account settings
│   └── api/                # API routes (proxy)
├── components/
│   ├── ui/                 # shadcn/ui components
│   ├── layout/             # Layout components
│   ├── betting/            # Betting-specific
│   ├── crash/              # Crash game
│   └── wallet/             # Wallet components
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities, API client
├── stores/                 # Zustand stores
├── types/                  # TypeScript types
└── public/                 # Static assets
```

## Key Pages

### 1. Home Page
- Hero section with featured matches
- Crash game widget (miniature)
- Quick bet slip
- Live matches ticker

### 2. Sports Betting
- Match list with filters (sport, league, date)
- Odds display (decimal format)
- Bet slip (floating panel)
- Multi-bet builder

### 3. Live Betting
- Real-time match updates (WebSocket)
- Live odds (auto-refresh)
- Match tracker (mini scoreboard)
- Quick bet on live events

### 4. Crash Game (Signature Feature)
- 3D airplane flying through sky (Three.js)
- Real-time multiplier display (large, animated)
- Bet input + auto-cashout setting
- Big green cashout button
- Game history (last 20 rounds)
- Provably fair verification
- Particle effects on crash

### 5. Wallet
- Balance display
- Deposit form (M-Pesa)
- Withdrawal form
- Transaction history (paginated)
- Deposit limits display

### 6. Account
- Profile settings
- KYC verification flow
- Self-exclusion settings
- Responsible gaming controls
- Bet history

## Crash Game Implementation

### 3D Scene Setup
```tsx
import { Canvas } from '@react-three/fiber'
import { Plane } from './components/Plane'
import { Sky } from './components/Sky'
import { Particles } from './components/Particles'

function CrashGame() {
  return (
    <Canvas camera={{ position: [0, 5, 10], fov: 60 }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[10, 10, 5]} />
      <Sky />
      <Plane />
      <Particles />
    </Canvas>
  )
}
```

### WebSocket Connection
```tsx
function useCrashGame(gameId: string) {
  const [multiplier, setMultiplier] = useState(1.0)
  const [status, setStatus] = useState<'waiting' | 'running' | 'crashed'>('waiting')
  
  useEffect(() => {
    const ws = new WebSocket(`wss://api.yourdomain.com/ws/games/crash/${gameId}`)
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data)
      
      switch (data.type) {
        case 'tick':
          setMultiplier(data.multiplier)
          break
        case 'crash':
          setStatus('crashed')
          setMultiplier(data.crash_point)
          break
        case 'round_start':
          setStatus('running')
          setMultiplier(1.0)
          break
      }
    }
    
    return () => ws.close()
  }, [gameId])
  
  return { multiplier, status }
}
```

### Cashout Button
```tsx
function CashoutButton({ multiplier, onCashout }: Props) {
  const winnings = betAmount * multiplier
  
  return (
    <button
      onClick={onCashout}
      className="bg-green-500 hover:bg-green-600 text-white font-bold py-6 px-12 rounded-xl text-2xl shadow-lg shadow-green-500/30 transition-all transform hover:scale-105"
    >
      <div>Cashout</div>
      <div className="text-3xl">{winnings.toFixed(2)} KES</div>
      <div className="text-sm opacity-75">{multiplier.toFixed(2)}x</div>
    </button>
  )
}
```

## State Management

### Bet Slip Store
```typescript
interface BetSlipStore {
  selections: BetSelection[]
  stake: number
  betType: 'single' | 'multi'
  addSelection: (selection: BetSelection) => void
  removeSelection: (matchId: string) => void
  setStake: (amount: number) => void
  clear: () => void
  placeBet: () => Promise<void>
}
```

### Wallet Store
```typescript
interface WalletStore {
  balance: number
  currency: string
  transactions: Transaction[]
  deposit: (amount: number, phone: string) => Promise<void>
  withdraw: (amount: number, phone: string) => Promise<void>
  refreshBalance: () => Promise<void>
}
```

## API Client

```typescript
// lib/api.ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30000,
      retry: 3,
    },
  },
})

// hooks/useMatches.ts
export function useMatches(sport: string, status: string) {
  return useQuery({
    queryKey: ['matches', sport, status],
    queryFn: () => api.getMatches(sport, status),
    refetchInterval: status === 'live' ? 5000 : 30000,
  })
}

// hooks/useCrashGame.ts
export function useCrashGame(gameId: string) {
  return useQuery({
    queryKey: ['crash-game', gameId],
    queryFn: () => api.getCrashGameState(gameId),
  })
}
```

## Responsive Design

### Breakpoints
- Mobile: < 640px (primary)
- Tablet: 640px - 1024px
- Desktop: > 1024px

### Mobile-First Approach
- Bottom navigation bar
- Floating bet slip
- Swipeable cards
- Touch-optimized buttons (min 44px)
- Pull-to-refresh

## Performance

### Targets
- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lighthouse Score: > 90
- WebSocket Latency: < 100ms

### Optimizations
- Image optimization (next/image)
- Code splitting (dynamic imports)
- Skeleton loading states
- Optimistic updates for bets
- Service worker for offline support
