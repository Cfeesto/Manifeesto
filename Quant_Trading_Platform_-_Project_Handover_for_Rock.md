# Quant Trading Platform - Project Handover for Rocket AI

## Project Overview
This is an **institutional-grade algorithmic trading platform** built with **Next.js + TypeScript + tRPC + React 19 + Tailwind 4**. The platform features live charting, strategy building, backtesting, AI analysis, and live forward testing with Binance Crypto and OANDA Forex integration.

**Project Location:** `/home/ubuntu/quant-trading-web`
**Dev Server:** Running on port 3000
**Database:** MySQL with Drizzle ORM
**Status:** Phase 2 in progress (Design system implemented, placeholder pages created)

---

## Completed Work

### Phase 1: Database Schema ✅
- All 10 tables created and deployed:
  - `strategies` - Trading strategy configurations
  - `ohlcvData` - OHLCV market data storage
  - `signals` - Buy/sell signals with non-repainting flag
  - `backtestResults` - Backtest metrics and equity curves
  - `walkForwardResults` - Walk-forward validation data
  - `marketRegimes` - Market regime classification (5 states)
  - `liveTrades` - Open/closed trades tracking
  - `apiCredentials` - Encrypted API key storage
  - `notifications` - Owner notifications
  - `users` - User authentication (pre-existing)

### Phase 2: Design System ✅
- **Neon-Noir Color Palette** implemented in `/client/src/index.css`:
  - Deep midnight navy background: `oklch(0.08 0.01 230)`
  - Hot pink/magenta accent: `oklch(0.65 0.25 300)`
  - Electric cyan/blue: `oklch(0.60 0.25 180)` and `oklch(0.40 0.20 270)`
  - Text glow effects with box-shadows
  - Neon utility classes: `.neon-glow`, `.neon-glow-cyan`, `.btn-neon`, `.card-neon`, `.input-neon`

- **Placeholder Pages Created:**
  - `/client/src/pages/Chart.tsx` - Candlestick/footprint chart page
  - `/client/src/pages/StrategyBuilder.tsx` - Strategy configuration UI
  - `/client/src/pages/Backtest.tsx` - Backtest results display
  - `/client/src/pages/AIAnalysis.tsx` - PDF parser and AI indicators
  - `/client/src/pages/Dashboard.tsx` - Main dashboard router

---

## Remaining Work (Phases 3-10)

### Phase 3: Interactive Charts 🔴
**File:** `client/src/pages/Chart.tsx` (placeholder exists)

**Tasks:**
1. Implement candlestick chart using **Recharts** or **Canvas API**
   - Display OHLCV data with proper scaling
   - Add zoom, pan, and tooltip interactions
   - Implement volume bars below price chart

2. Implement footprint volume chart
   - Show bid/ask delta per price level
   - Color-code volume imbalances (red/green)
   - Display cumulative delta

3. Add non-repainting signal overlays
   - Plot buy signals (green triangles) at confirmed entry points
   - Plot sell signals (red triangles) at confirmed exit points
   - Ensure `isRepainting = 0` in database before displaying

4. Add symbol and timeframe selectors
   - Fetch data from Binance/OANDA APIs
   - Update chart on selection change

**API Endpoints Needed:**
- `trpc.market.getOhlcvData.useQuery({ symbol, timeframe, limit })`
- `trpc.market.getSignals.useQuery({ strategyId })`

---

### Phase 4: API Integration 🔴
**Files:** 
- `server/db.ts` (add market data helpers)
- `server/routers.ts` (add market procedures)
- `server/_core/` (create API clients)

**Tasks:**
1. Create Binance API client
   - Fetch OHLCV data for crypto pairs (BTCUSDT, ETHUSDT, etc.)
   - Support multiple timeframes (1m, 5m, 15m, 1h, 4h, 1d)
   - Implement rate limiting (1200 requests/minute)

2. Create OANDA API client
   - Fetch OHLCV data for forex pairs (EURUSD, GBPUSD, etc.)
   - Support same timeframes as Binance
   - Handle authentication with API key

3. Implement encrypted credential storage
   - Use Node.js `crypto` module with AES-256-GCM
   - Encrypt API keys before storing in `apiCredentials` table
   - Decrypt on-demand when making API calls

4. Create tRPC procedures
   - `market.getOhlcvData` - Fetch historical OHLCV
   - `market.getLivePrice` - Get current price
   - `credentials.saveApiKey` - Store encrypted credentials
   - `credentials.testConnection` - Verify API key validity

**Database Helpers (in `server/db.ts`):**
```typescript
export async function saveOhlcvData(data: InsertOhlcvData[]) { ... }
export async function getOhlcvData(symbol: string, timeframe: string, limit: number) { ... }
export async function saveSignal(signal: InsertSignal) { ... }
export async function getSignalsByStrategy(strategyId: number) { ... }
```

---

### Phase 5: Strategy Builder 🔴
**File:** `client/src/pages/StrategyBuilder.tsx` (placeholder exists)

**Tasks:**
1. Design drag-and-drop indicator rule system
   - Create `IndicatorRule` component for each condition
   - Support operators: >, <, =, AND, OR, NOT
   - Allow adding/removing rules dynamically

2. Implement entry/exit condition configuration
   - Tab 1: Entry Rules (e.g., RSI > 70 AND MACD Crossover)
   - Tab 2: Exit Rules (e.g., RSI < 30)
   - Store as JSON in `strategies.entryRules` and `strategies.exitRules`

3. Implement position sizing and risk controls
   - Position Size (% of account)
   - Stop Loss (pips or %)
   - Take Profit (pips or %)
   - Max Drawdown threshold

4. Add strategy save/load functionality
   - `trpc.strategy.create.useMutation()`
   - `trpc.strategy.update.useMutation()`
   - `trpc.strategy.list.useQuery()`

**Database Helpers:**
```typescript
export async function createStrategy(strategy: InsertStrategy) { ... }
export async function updateStrategy(id: number, updates: Partial<Strategy>) { ... }
export async function getStrategiesByUser(userId: number) { ... }
```

---

### Phase 6: Backtesting Engine 🔴
**Files:**
- `server/backtesting/engine.ts` (create new)
- `client/src/pages/Backtest.tsx` (implement)

**Tasks:**
1. Implement backtesting logic
   - Load historical OHLCV data
   - Apply strategy rules to each candle
   - Generate trade signals
   - Execute trades and track P&L

2. Calculate performance metrics
   - Total Trades, Winning Trades, Losing Trades
   - Win Rate = (Winning Trades / Total Trades) × 100
   - Profit Factor = (Gross Profit / Gross Loss)
   - Max Drawdown = (Peak to Trough) / Peak
   - Sharpe Ratio = (Return - Risk-Free Rate) / Std Dev
   - Total Return = (Final Balance - Initial Balance) / Initial Balance

3. Generate equity curve
   - Array of cumulative balance after each trade
   - Store as JSON in `backtestResults.equityCurve`

4. Implement walk-forward validation
   - Split data into in-sample and out-of-sample periods
   - Train on in-sample, test on out-of-sample
   - Calculate overfitting score = 1 - (OOS Return / IS Return)
   - Flag as overfit if score > 0.3

**Backtesting Engine Pseudocode:**
```typescript
async function runBacktest(strategyId: number, startDate: Date, endDate: Date) {
  const strategy = await getStrategy(strategyId);
  const ohlcv = await getOhlcvData(strategy.symbol, strategy.timeframe, startDate, endDate);
  
  let balance = 10000;
  const trades = [];
  
  for (let i = 0; i < ohlcv.length; i++) {
    const candle = ohlcv[i];
    
    // Check entry rules
    if (evaluateRules(strategy.entryRules, ohlcv.slice(0, i+1))) {
      const trade = { entryPrice: candle.close, entryTime: candle.openTime };
      
      // Check exit rules
      for (let j = i + 1; j < ohlcv.length; j++) {
        if (evaluateRules(strategy.exitRules, ohlcv.slice(0, j+1))) {
          trade.exitPrice = ohlcv[j].close;
          trade.exitTime = ohlcv[j].openTime;
          const pnl = (trade.exitPrice - trade.entryPrice) * positionSize;
          balance += pnl;
          trades.push(trade);
          break;
        }
      }
    }
  }
  
  return calculateMetrics(trades, balance);
}
```

---

### Phase 7: AI Strategy Parser 🔴
**Files:**
- `server/ai/pdfParser.ts` (create new)
- `client/src/pages/AIAnalysis.tsx` (implement)

**Tasks:**
1. Implement PDF upload and parsing
   - Accept PDF file upload
   - Extract text using PDF library (pdfjs or similar)
   - Send to LLM for rule extraction

2. Integrate LLM for rule extraction
   - Use `invokeLLM()` from `server/_core/llm.ts`
   - Prompt: "Extract trading rules from this PDF. Return JSON with: entryRules, exitRules, indicators, positionSize, stopLoss, takeProfitRules"
   - Parse LLM response into structured format

3. Generate strategy configurations
   - Convert extracted rules to `Strategy` object
   - Validate rule syntax
   - Save to database

4. Add UI for PDF parser results
   - Display extracted rules in readable format
   - Allow user to edit/refine rules
   - Generate strategy from rules

**LLM Prompt Template:**
```
You are a trading strategy expert. Extract trading rules from the following PDF content.
Return a JSON object with this structure:
{
  "entryRules": "description of entry conditions",
  "exitRules": "description of exit conditions",
  "indicators": ["list", "of", "indicators"],
  "positionSize": 100,
  "stopLoss": 50,
  "takeProfit": 100
}
```

---

### Phase 8: Market Regime Classifier & Live Trading 🔴
**Files:**
- `server/regimes/classifier.ts` (create new)
- `client/src/pages/LiveTrading.tsx` (create new)

**Tasks:**
1. Implement market regime classifier
   - Analyze volatility, trend, and price action
   - Classify into 5 states:
     - Strong Bull: Uptrend + Low Volatility
     - Strong Bear: Downtrend + Low Volatility
     - Sideways: Low Trend + Low Volatility
     - High Volatility: Any trend + High Volatility
     - Institutional Absorption: Specific pattern (large volume, small range)

2. Build regime-specific performance tracking
   - Calculate strategy performance per regime
   - Store regime classification in `marketRegimes` table
   - Display regime-specific win rates and metrics

3. Create live forward testing dashboard
   - Real-time signal display
   - Open positions and P&L tracking
   - Alert configuration UI

4. Implement real-time signal display
   - WebSocket or polling for live updates
   - Show buy/sell signals as they occur
   - Display signal confidence and regime

**Regime Classification Logic:**
```typescript
function classifyRegime(ohlcv: OhlcvData[], volatility: number, trend: number) {
  if (volatility > 0.03) return 'high_volatility';
  if (trend > 0.02 && volatility < 0.01) return 'strong_bull';
  if (trend < -0.02 && volatility < 0.01) return 'strong_bear';
  if (Math.abs(trend) < 0.01 && volatility < 0.01) return 'sideways';
  return 'institutional_absorption';
}
```

---

### Phase 9: Notifications 🔴
**Files:**
- `server/_core/notification.ts` (already exists, extend it)
- `client/src/pages/Settings.tsx` (create new)

**Tasks:**
1. Implement owner notification system
   - Use existing `notifyOwner()` helper
   - Trigger on new buy/sell signals
   - Trigger on drawdown breach

2. Add drawdown breach notifications
   - Monitor live P&L against `strategies.maxDrawdown`
   - Send alert when exceeded

3. Implement notification preferences
   - Allow user to configure notification types
   - Email, webhook, or in-app notifications

4. Create Settings page
   - API credential management
   - Notification preferences
   - Strategy management

---

### Phase 10: Testing & Delivery 🔴
**Tasks:**
1. Write vitest unit tests
   - Test backtesting engine calculations
   - Test regime classifier
   - Test strategy rule evaluation

2. Test API integrations
   - Verify Binance and OANDA data fetching
   - Test encrypted credential storage

3. Perform end-to-end testing
   - Test full flow: upload PDF → generate strategy → backtest → live trade

4. Optimize performance
   - Cache OHLCV data
   - Optimize database queries
   - Minimize re-renders in React

---

## Architecture Overview

### Tech Stack
- **Frontend:** React 19 + TypeScript + Tailwind 4 + shadcn/ui
- **Backend:** Express + tRPC + Node.js
- **Database:** MySQL + Drizzle ORM
- **APIs:** Binance REST API, OANDA REST API
- **LLM:** Manus built-in LLM integration

### Key Files Structure
```
/home/ubuntu/quant-trading-web/
├── client/src/
│   ├── pages/
│   │   ├── Chart.tsx (placeholder)
│   │   ├── StrategyBuilder.tsx (placeholder)
│   │   ├── Backtest.tsx (placeholder)
│   │   ├── AIAnalysis.tsx (placeholder)
│   │   ├── LiveTrading.tsx (TO CREATE)
│   │   ├── Settings.tsx (TO CREATE)
│   │   └── Dashboard.tsx (router)
│   ├── components/
│   │   └── DashboardLayout.tsx (pre-built)
│   └── index.css (neon-noir design ✅)
├── server/
│   ├── routers.ts (add market, strategy, backtest procedures)
│   ├── db.ts (add market data helpers)
│   ├── backtesting/ (TO CREATE)
│   ├── regimes/ (TO CREATE)
│   └── ai/ (TO CREATE)
├── drizzle/
│   └── schema.ts (all tables ✅)
└── todo.md (tracking file)
```

---

## Critical Implementation Notes

### Non-Repainting Signals
- Only display signals where `signals.isRepainting = 0`
- Set `isRepainting = 0` only when candle closes
- Use `isRepainting = 1` for tentative signals during candle formation

### Encrypted Credential Storage
```typescript
import crypto from 'crypto';

function encryptCredential(plaintext: string, key: string): string {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', Buffer.from(key), iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const authTag = cipher.getAuthTag();
  return iv.toString('hex') + ':' + encrypted.toString('hex') + ':' + authTag.toString('hex');
}

function decryptCredential(encrypted: string, key: string): string {
  const parts = encrypted.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const decipher = crypto.createDecipheriv('aes-256-gcm', Buffer.from(key), iv);
  decipher.setAuthTag(Buffer.from(parts[2], 'hex'));
  return decipher.update(Buffer.from(parts[1], 'hex')) + decipher.final('utf8');
}
```

### Neon-Noir Design Utilities
All components should use these classes:
- `.neon-glow` - Hot pink text with glow
- `.neon-glow-cyan` - Cyan text with glow
- `.btn-neon` - Neon button styling
- `.card-neon` - Card with neon border and subtle glow
- `.input-neon` - Input with neon focus state
- `.accent-line-vertical` - Vertical accent line

---

## Running the Project

```bash
cd /home/ubuntu/quant-trading-web

# Install dependencies
pnpm install

# Start dev server
pnpm dev

# Run tests
pnpm test

# Build for production
pnpm build
```

---

## Database Connection
- **URL:** Provided in environment variables
- **Tables:** All 10 tables created and ready
- **ORM:** Drizzle with MySQL2 driver

---

## Next Steps for Rocket AI

1. **Start with Phase 3:** Implement the interactive chart component (most critical)
2. **Then Phase 4:** Complete API integration for Binance and OANDA
3. **Then Phase 5-10:** Follow the remaining phases in order
4. **Testing:** Write vitest tests for each phase before moving to the next
5. **Checkpoint:** Save a checkpoint after completing each phase

---

## Contact & Support
All specifications, database schema, and design tokens are documented in this file. The project is fully set up and ready for implementation. Good luck! 🚀
