# F&O Trading for Retail Investors: From Zero to Algorithmic Trader

A self-paced, ~40-week course for an Indian retail trader who wants to **first learn F&O properly, then design, test and deploy algorithms**.

> **Read this first.** SEBI's own studies show roughly 9 in 10 individual F&O traders lose money. This course is built around that fact. Its goal is not "make ₹1 lakh/month from ₹50,000" (that is a 200% monthly return, which no one achieves reliably). Its goal is to make you **someone who understands what they are doing, controls risk, and can tell a real edge from a lucky streak**. Profit, if it comes, is a by-product of those skills.
>
> Numbers in this document that change over time (lot sizes, margins, STT rates, expiry days, SEBI algo rules) are marked **[VERIFY]**. Always check the latest NSE/SEBI circulars and your broker before relying on them. Nothing here is investment, legal or tax advice.

---

## 0. How the course works

### 0.1 Structure

| Stage | Weeks | Goal | Real money? |
|---|---|---|---|
| **A. Foundations** | 1-5 | Understand market, contracts, costs, tax, regulation | No |
| **B. Options & Futures mastery** | 6-16 | Pricing, Greeks, volatility, strategies, expiry mechanics | No (paper / simulator) |
| **C. Risk & Psychology** | 17-19 | Sizing, drawdowns, journaling, behaviour | No |
| **D. Quant & Python toolkit** | 20-25 | Data, statistics, backtesting fundamentals | No |
| **E. Algorithm design** | 26-34 | Research, backtest, validate, build engine | No (paper) |
| **F. Go-live & operations** | 35-40+ | Staged deployment, monitoring, review | Tiny, then scaled by rules |

Time: ~8-12 hours/week. Do not rush; the market will still be there.

### 0.2 Gates (do not skip)

You may only move to the next stage when you pass the **gate** at the end of the current one. Gates are written as concrete, checkable deliverables, not "I feel ready".

### 0.3 Rules of engagement

1. **No real money until Stage F.** Stages A-E use a simulator, paper trading, or historical data.
2. **Every claim needs a test.** If you cannot back a belief with data, it is a hypothesis.
3. **Keep a learning journal** (`journal/`) from day one: what you learned, what confused you, what you got wrong.
4. **Costs are part of the strategy.** A strategy is evaluated only after brokerage, STT, exchange charges, GST, stamp duty and slippage.
5. **Capital you can lose.** Use only money whose loss won't affect your life. Keep a separate emergency fund and clear high-interest debt first.

### 0.4 Tools you'll need

- A laptop, Python 3.11+, Jupyter, Git
- A discount broker account with API access (Zerodha, Upstox, Angel One, Dhan, Fyers, etc.)
- A free/cheap charting platform (TradingView, Sensibull for payoffs, Opstra, NSE option chain)
- A spreadsheet
- A paper-trading environment (broker sandbox, Sensibull virtual, or your own simulator, which you'll build in Stage E)

---

# STAGE A: FOUNDATIONS (Weeks 1-5)

## Module 1: How the Indian market works (Week 1)

**Learning objectives**
- Explain what exchanges, clearing corporations, depositories, brokers and regulators do.
- Trace one order from your phone to settlement.

**Topics**
1. Primary vs secondary markets; NSE vs BSE; NSE Clearing / ICCL
2. Roles: SEBI (regulator), exchange (matching), clearing corporation (counterparty guarantee, margins), broker (access), depository (CDSL/NSDL)
3. Market timings, pre-open, circuit limits, price bands, trading holidays
4. Order types: market, limit, SL, SL-M, IOC, GTT/GTC, bracket/cover (and why some are being phased out), freeze quantity, iceberg
5. Settlement: T+1 for cash equity, daily MTM and cash settlement for futures, physical settlement of stock derivatives [VERIFY current rules]
6. Market microstructure basics: order book, bid-ask spread, depth, slippage, impact

**Exercises**
- Open an exchange option chain and a stock order book; screenshot and annotate bid, ask, depth, LTP, OI, volume.
- Place (on paper) a market order and a limit order for the same instrument. Describe what could differ in fills.

**Self-check questions**
1. Why does a clearing corporation exist, and what protects you from the counterparty defaulting?
2. What is freeze quantity and why might your algo hit it?
3. What happens to a long stock-futures position that is held to expiry?

## Module 2: Derivatives 101 (Week 2)

**Learning objectives**
- Define a derivative and its uses (hedging, speculation, arbitrage).
- Read contract specs for any F&O instrument.

**Topics**
1. Forward vs future; why standardisation matters
2. Underlyings: index (Nifty 50, Bank Nifty, Fin Nifty, Midcap Nifty, Sensex), stocks, ETFs; which are cash-settled vs physically settled
3. Contract specifications: lot size, tick size, expiry cycle (weekly/monthly), strike intervals, premium quoting, contract value [VERIFY: SEBI's 2024 framework changed lot sizes, limited weekly expiries to one index per exchange, raised minimum contract value, and tightened expiry-day rules]
4. Leverage: notional vs margin; how leverage magnifies both gains and losses
5. Mark-to-market (MTM) and daily settlement of futures
6. Open interest, volume, rollover: what each means and what each doesn't
7. Contango / backwardation; cost of carry; fair value of a future

**Key formula**
```
Future fair value ≈ Spot × e^((r - q) × T)      (r = interest rate, q = dividend yield, T in years)
Basis = Future price − Spot
```

**Exercises**
- Create a spreadsheet: for Nifty, list lot size, current-month future price, spot, basis, annualised basis %. Compare with the repo rate / T-bill yield.
- For Nifty, Bank Nifty and one stock, compute notional value of 1 lot and margin required. Express margin as % of notional.

**Self-check questions**
1. If Nifty is at 24,000 and the lot size is 65 (check current), what's the notional of one lot? If margin is ₹1.5 lakh, what is the effective leverage?
2. A future trades at a premium to spot. Is that a "bullish signal"? Why or why not?

## Module 3: Costs, margins, taxation and regulation (Week 3)

**Learning objectives**
- Compute the full round-trip cost of an F&O trade.
- Understand how margins (SPAN + exposure + ELM) work and why they spike near expiry.
- File F&O income correctly.

**Topics**
1. **Cost stack:** brokerage (flat ₹20/order typical), exchange transaction charges, SEBI turnover fee, STT (on sell side for options premium and futures value; rates have changed repeatedly, so [VERIFY]), GST on brokerage + charges, stamp duty (buy side), DP charges (n/a for F&O unless delivery), **slippage** (the largest hidden cost)
2. **Options-specific STT trap:** STT on exercised in-the-money options is charged on intrinsic value at a far higher effective rate. Never hold ITM long options through expiry unless you intend exercise.
3. **Margins:** SPAN (risk-based), exposure margin, extreme loss margin (ELM), upfront collection of option premium for buyers, peak-margin reporting and penalties, no cross-margin benefit on certain expiry-day positions [VERIFY]
4. **Tax in India (verify with a CA each year):**
   - F&O gains/losses = **non-speculative business income** (not capital gains), reported in **ITR-3**
   - Turnover calculation per ICAI guidance (absolute sum of profits + losses for futures; options turnover rules differ)
   - Tax audit (Section 44AB) triggers: turnover thresholds and the loss/low-profit presumptive-scheme interaction
   - Business losses can be carried forward 8 years (only if ITR filed on time); set off against other business income, not against salary
   - Advance tax obligations; expenses allowed (internet, software, data, depreciation)
   - Slab-rate taxation: model your *post-tax* returns
5. **Regulation:** SEBI's F&O measures (suitability, upfront premium, intraday position monitoring, lot-size and expiry changes), SEBI's retail algo framework (broker accountability, algo registration/tagging, static IP, rate limits [VERIFY current status and deadlines]), risk of unregistered "tips/advisory" services

**Exercises**
- Download your broker's brokerage calculator. Compute the exact all-in cost of: (a) 1 lot Nifty future round trip, (b) selling 1 lot of ATM Nifty option and buying it back, (c) a 4-leg iron condor entered and exited.
- Compute breakeven move (in Nifty points) required for each to net zero.
- Draft a one-page tax-planning memo for yourself; take it to a CA and compare.

**Self-check questions**
1. Why can a strategy with a +₹3/lot gross edge be a guaranteed loser net of costs?
2. How does the ITM-expiry STT rule change how you should close a long option position?

## Module 4: Broker, platform and workflow setup (Week 4)

**Topics**
1. Choosing a broker: API cost, rate limits, websocket quality, historical data availability, order types, margin/pledge facilities, reliability on expiry days
2. Using the option chain properly: IV, OI, change in OI, volume, bid/ask, Greeks columns
3. Payoff and analytics tools; the difference between payoff-at-expiry and current P&L (T+0) curves
4. Setting up: Python environment, virtualenv/poetry, Git repo, secrets management (never commit API keys!)
5. Authenticating to a broker API; placing a *paper* order via sandbox or a 1-share equity test

**Exercises**
- Set up the repo layout (see Section 12), a `.env` for secrets, and a `.gitignore` that excludes it.
- Write a script that logs in and prints your margins and a live quote, with no order placement.

## Module 5: Stage A consolidation (Week 5)

**Capstone A:** a 3-5 page "Market Structure & Cost Report" containing: contract specs table (5 instruments), full cost stack for 3 trade types, tax summary reviewed by a CA, and a list of 10 things you could do wrong operationally.

### ✅ GATE A
- [ ] You can compute any trade's all-in cost and breakeven by hand and match your broker calculator
- [ ] You can explain SPAN/exposure/ELM margin and why margin changes
- [ ] You know your tax category, ITR form and audit thresholds (CA-confirmed)
- [ ] Capstone A completed and reviewed

---

# STAGE B: OPTIONS & FUTURES MASTERY (Weeks 6-16)

## Module 6: Option basics and payoffs (Week 6)

**Topics**
1. Calls and puts; buyer vs seller; rights vs obligations
2. Moneyness: ITM / ATM / OTM; intrinsic vs time value
3. Payoff diagrams at expiry for the 4 basic positions (long/short call/put)
4. European-style index options vs stock options; exercise and assignment in India
5. Breakevens, max profit, max loss, risk-reward
6. Put-call parity: `C − P = S − K·e^(−rT)` (with dividends adjustments)

**Exercises**
- By hand and in Python (`matplotlib`), plot payoffs of the 4 basic positions for Nifty 24,000 strikes.
- Verify put-call parity numerically using live option chain data; explain deviations (bid-ask, dividends, borrowing).

**Self-check**
1. Why is selling an option "picking up pennies in front of a steamroller" and how does margin hide that?
2. Why does a long call lose money even if the underlying rises slightly?

## Module 7: Option pricing (Week 7)

**Learning objectives:** Understand where option prices come from without needing to memorise math.

**Topics**
1. Intuition: expected payoff under risk-neutral measure, no-arbitrage replication
2. **Black-Scholes-Merton (BSM)** for European options (use Black-76 for options on futures)
   ```
   d1 = [ln(S/K) + (r − q + σ²/2)T] / (σ√T)
   d2 = d1 − σ√T
   C  = S·e^(−qT)·N(d1) − K·e^(−rT)·N(d2)
   P  = K·e^(−rT)·N(−d2) − S·e^(−qT)·N(−d1)
   ```
3. Assumptions and where they break: constant vol, lognormal returns, continuous trading, no jumps (reality: fat tails, vol smile, gaps, overnight events)
4. Inputs: S, K, T (use trading time vs calendar time), r, q, σ
5. Implied volatility: backing σ out of market price (Newton-Raphson/bisection)
6. Binomial tree intuition (optional)

**Exercises**
- Implement BSM price and IV solver in Python from scratch. Validate against a library (`py_vollib` or `scipy`) and against live market prices.
- Plot option price vs spot, vs time, vs volatility.

**Self-check**
1. If IV doubles, what happens to ATM option price? To deep OTM?
2. What does "market price implies vol of 14%" actually mean?

## Module 8: The Greeks (Weeks 8-9)

**Topics (one at a time, with intuition → formula → behaviour → trading implication)**

| Greek | Measures | Key behaviours | Trading implication |
|---|---|---|---|
| **Delta** | ∂V/∂S | ATM ≈ 0.5, deep ITM → 1; also approx probability of finishing ITM | Directional exposure; delta-neutral construction |
| **Gamma** | ∂²V/∂S² | Highest ATM, explodes near expiry | Short gamma = hurt by big moves; "expiry-day risk" |
| **Theta** | ∂V/∂t | Accelerates near expiry for ATM | Sellers collect it, buyers pay it |
| **Vega** | ∂V/∂σ | Highest ATM and for longer maturities | Exposure to IV changes (results, events, VIX) |
| **Rho** | ∂V/∂r | Minor for short-dated | Mostly ignorable in India short-dated |
| **Second order** | Vanna, Volga, Charm | Dynamics of delta w.r.t. vol and time | Advanced; understand at concept level |

Key relationships:
- **Gamma-theta trade-off:** long gamma costs theta; short theta earns by being short gamma. Over a day, `P&L ≈ ½·Γ·(ΔS)² + Θ·Δt` for a delta-hedged position. This is the heart of option trading.
- **Greeks of a portfolio** add across positions; learn to compute net delta/gamma/theta/vega in rupees per point and per 1% IV.

**Exercises**
- Build a Greeks dashboard: input a list of positions, output net Greeks and a P&L grid across spot × IV × days-to-expiry.
- Simulate a delta-hedged long straddle over 100 random paths. Show the distribution of P&L vs realised-vs-implied vol.
- Take an iron condor; plot its theta and gamma profile through the last 5 days to expiry.

**Self-check**
1. Why does a short-straddle seller's risk change character in the last 2 days?
2. You're long 1 lot ATM call (delta 0.5, gamma 0.004, theta −8/day, vega 12). Nifty rises 100 points and IV falls 2 points. Estimate P&L.

## Module 9: Volatility (Weeks 10-11)

This is the most important module for option traders.

**Topics**
1. **Historical (realised) volatility:** close-to-close, Parkinson, Garman-Klass, Yang-Zhang; annualisation (√252)
2. **Implied volatility:** what the market is pricing; India VIX construction and interpretation
3. **Variance risk premium (VRP):** implied tends to exceed realised on average, but with brutal left-tail losses. This is the structural reason option selling "works" until it doesn't.
4. **IV Rank / IV Percentile:** definitions, uses and limits
5. **Volatility smile/skew:** why OTM puts are pricier in equity indices; skew as fear gauge
6. **Term structure:** weekly vs monthly IV; event premium (budget, RBI, elections, results)
7. **Volatility clustering and mean reversion:** GARCH intuition
8. **IV crush:** the post-event IV drop that wrecks long-option buyers
9. **Regimes:** low-vol grinding markets vs gap-down crashes

**Exercises**
- Download 3-5 years of Nifty and India VIX daily data. Compute rolling 20-day realised vol, compare with VIX. Plot VRP over time; find the worst episodes.
- Build an IV-rank calculator and a table of average next-5-day outcomes by IV-rank bucket.
- Plot the live skew for Nifty weekly options and describe it.

**Self-check**
1. If VIX is 11, is it "cheap to buy options"? What else must you check?
2. Why do most retail option buyers lose even when they get direction right?

## Module 10: Options strategies: the catalogue (Weeks 12-13)

For each strategy learn: **setup, payoff, max profit/loss, breakevens, Greeks profile, ideal market view, ideal IV view, margin, common failure modes, adjustments.**

**Directional**
- Long call / long put; bull call spread, bear put spread (debit); bull put spread, bear call spread (credit)
- Risk-reversal / synthetic futures; ratio spreads and back-spreads

**Neutral / volatility**
- Short straddle, short strangle (undefined risk, **avoid at small capital**)
- Long straddle, long strangle
- Iron condor, iron butterfly (defined risk)
- Calendar and diagonal spreads (watch expiry-day rules)
- Jade lizard, broken-wing butterfly

**Hedging / income**
- Covered call, protective put, collar (needs stock holding or futures)
- Hedged futures positions

**Arbitrage-style (educational)**
- Cash-futures arbitrage, calendar arbitrage, put-call parity arb, box spreads, conversions. Understand why retail rarely captures these after costs.

**How to choose a strategy: a decision framework**
```
View on direction?  ──► bullish / bearish / neutral
View on volatility? ──► expect IV rise / fall / unsure
View on time?       ──► event date, days to expiry
Risk budget?        ──► max loss per trade ≤ 1-2% of capital
Instrument?         ──► liquid (tight spread, high OI) only
→ pick the structure whose Greeks match the view and whose max loss fits the budget
```

**Exercises**
- Build a strategy library in Python: each strategy is a class returning legs, payoff, net Greeks, margin estimate, max loss/profit, breakevens.
- For your capital (₹50,000), list which strategies are even feasible given margin and lot size. (Likely: only defined-risk spreads at 1 lot, and possibly none on the largest indices. This is a finding, not a failure.)

## Module 11: Adjustments, expiry mechanics and trade management (Week 14)

**Topics**
1. Rules for taking profits and cutting losses *before* entry (e.g., close at 50% of max profit; exit if loss = 1-2× credit received, or if short strike is touched)
2. Rolling, widening, converting to condors, adding hedges; when adjusting just adds risk ("throwing good money after bad")
3. **Expiry day** behaviour: gamma explosion, pin risk, illiquidity, STT on exercise, margin changes, auto-square-off by brokers, circuit/halt scenarios, settlement price mechanics [VERIFY current expiry rules]
4. Event risk: RBI policy, Budget, US Fed, results, elections, geopolitical gaps
5. Gap risk and overnight holding: why stop-loss orders don't protect against gaps
6. Liquidity risk: wide spreads in far-OTM, stock options
7. Operational failures: API down, order rejection, partial fills, freeze-quantity slicing

**Exercises**
- Write a "trade plan template" (entry, size, max loss, profit target, adjustment rules, exit rules, invalidation) and fill it out for 5 hypothetical trades.
- Replay a historical crash day (e.g., 2020-03, a major gap day) with a short strangle and compute the loss in rupees at 1 lot.

## Module 12: Futures, OI analytics and market indicators (Week 15)

**Topics**
1. Futures trading: leverage, MTM, rollover, basis trades, pair trades (index vs sector)
2. OI interpretation grid (and its limits): price ↑ + OI ↑ = long build-up; price ↓ + OI ↑ = short build-up; price ↑ + OI ↓ = short covering; price ↓ + OI ↓ = long unwinding. **Treat as hypotheses, not signals.**
3. Put-Call Ratio (OI and volume), max pain: popular but weakly predictive, and test them
4. Advance-decline, FII/DII derivatives positioning (index futures long/short ratio), participant-wise OI
5. Technical context: support/resistance, VWAP, ATR, moving averages, Bollinger, RSI. Evaluate each as a *testable feature*, not gospel.
6. Macro/behavioural context: rates, crude, USD/INR, global cues, GIFT Nifty

**Exercises**
- Download participant-wise OI (NSE archives); plot FII index-futures net position vs next-week Nifty return. Is there a relationship, out of sample?
- Backtest "PCR extreme → contrarian" and "max pain pin" on historical data; report hit rate and expectancy after costs.

## Module 13: Stage B consolidation (Week 16)

**Capstone B:** "Strategy Playbook": 5 strategies each with: thesis, payoff, Greeks, when to use, when not to, risk rules, historical stress test (using real chain data if you can get it, else a model), and cost-adjusted expectation.

### ✅ GATE B
- [ ] You can price an option and solve IV from scratch in code and match market data within tolerance
- [ ] You can predict the sign and rough size of P&L change for a given spot/IV/time move using Greeks (pass 20-question quiz at ≥ 85%)
- [ ] You can explain VRP and why short-vol strategies have negative skew
- [ ] Capstone B reviewed
- [ ] 8 weeks of **paper trades** logged with full trade plans (min 30 trades), reviewed for process errors

---

# STAGE C: RISK & PSYCHOLOGY (Weeks 17-19)

## Module 14: Risk management and position sizing (Week 17)

**Topics**
1. Risk of ruin; why survival beats optimisation
2. Fixed-fractional risk per trade (1-2%), max daily/weekly/monthly loss limits
3. Position sizing for defined-risk vs undefined-risk strategies
4. Kelly criterion and why to use *fractional* Kelly (¼ or less); sensitivity to estimation error
5. Expectancy: `E = (Win% × AvgWin) − (Loss% × AvgLoss)`; profit factor; payoff asymmetry
6. Drawdown mathematics: a 50% loss requires a 100% gain; recovery time
7. Correlation and concentration: three Nifty positions are one position
8. Portfolio-level Greeks limits (net delta, vega caps), stress tests (−5%, −10% spot; +50% IV)
9. Black-swan planning: always have defined-risk or hedges for tail events

**Exercises**
- Monte-Carlo a strategy with 60% win rate, avg win ₹1,000, avg loss ₹2,000, risking X% of capital per trade. Plot distribution of drawdowns and probability of ruin for X = 1%, 2%, 5%, 10%.
- Write a risk policy (1 page): per-trade, daily, weekly limits, what happens when limits are hit, and a cool-down rule.

## Module 15: Trading psychology and process (Week 18)

**Topics**
- Common biases: loss aversion, revenge trading, overconfidence, recency, FOMO, sunk cost, gambler's fallacy, "one more adjustment"
- Why expiry-day gambling is addictive (variable-ratio rewards) and the data on its outcomes
- Process over outcome: judge a trade by whether you followed the plan, not by P&L
- Journal design: pre-trade (thesis, plan), post-trade (outcome, execution quality, emotions), weekly review
- Setting limits externally (broker-side loss limits, algo kill switches, time-boxed trading)
- Recognising problem trading and when to stop; helplines/resources if it harms health, finances or relationships

**Exercises**
- Create a trade journal template (CSV/Notion/Sheets). Journal every paper trade with emotion tags.
- After 30 trades, do a "mistake taxonomy": categorise every loss as *good-process-bad-luck*, *bad-process*, or *bad-idea*.

## Module 16: Stage C consolidation (Week 19)

**Capstone C:** a personal **Risk & Conduct Policy** (signed by you!) covering capital allocation, risk limits, forbidden behaviours (e.g., no averaging down, no trading to recover losses, no undefined-risk positions before X months of live profit), and review cadence.

### ✅ GATE C
- [ ] Risk & Conduct Policy written and shared with an accountability partner
- [ ] Monte-Carlo ruin analysis completed
- [ ] Trade journal with ≥ 30 entries and mistake taxonomy

---

# STAGE D: QUANT & PYTHON TOOLKIT (Weeks 20-25)

## Module 17: Python for trading (Weeks 20-21)

**Topics**
- Python essentials: data types, functions, classes, packaging, virtual environments, typing, testing (`pytest`), logging
- Data: `numpy`, `pandas` (time-series indexing, resampling, rolling windows, merging, timezone handling `Asia/Kolkata`), `polars` (optional)
- Visualisation: `matplotlib`, `plotly`
- Performance: vectorisation vs loops; `numba` (optional)
- Storage: CSV, Parquet, SQLite/DuckDB, PostgreSQL/TimescaleDB for tick data
- Code quality: git workflow, code review, type hints, linting (`ruff`), pre-commit hooks, CI

**Exercises**
- Write a data-loader module that returns clean OHLCV DataFrames from files or API, handling holidays, splits/bonus, contract rollover.
- Write unit tests for your BSM pricer and cost calculator.

## Module 18: Data for Indian F&O (Week 22)

**Topics**
1. Sources: broker historical API, NSE bhavcopy (F&O end-of-day), NSE/BSE archives (participant OI), paid vendors (Global Datafeeds, TrueData, etc.), and the realities of free data
2. Data types: spot OHLC (1-min/EOD), futures (continuous contract construction: roll, back-adjust vs ratio-adjust), option chains (EOD vs tick; IV and Greeks must be *computed*), India VIX
3. **Data quality hazards:** missing minutes, bad prints, corporate actions, lot-size changes over time, symbol changes, expiry-calendar changes, timezone and holiday errors, **survivorship bias**, **look-ahead bias** (e.g., using day's close to decide an entry "at open")
4. Building an options dataset: store raw chain snapshots, derive mid, spread, IV, Greeks; keep point-in-time (what was *knowable* then)
5. Storage design and data versioning; reproducibility

**Exercises**
- Build a pipeline that downloads and stores 3+ years of daily F&O bhavcopy and VIX; add data-quality checks (assert no gaps, validate monotonic dates, outlier flags).
- Construct a continuous Nifty futures series both ways; show the difference in returns.

## Module 19: Statistics and probability for traders (Week 23)

**Topics**
- Returns: simple vs log; distribution shape (skew, kurtosis, fat tails); stationarity; autocorrelation
- Hypothesis testing and **p-hacking**: with enough variants, something always "works"; multiple testing correction (Bonferroni, FDR), deflated Sharpe ratio
- Confidence intervals via bootstrap; sample size needed to tell a 55% from a 50% win rate (it's large!)
- Regression basics, feature relevance, overfitting; bias-variance
- Time-series: ARIMA/GARCH intuition; regime detection (HMM, rolling stats)
- Performance metrics: CAGR, volatility, Sharpe, Sortino, Calmar, max drawdown, profit factor, win rate, avg win/loss, expectancy, tail ratio, time under water
- Why backtest Sharpe overstates live Sharpe

**Exercises**
- Simulate 1,000 random strategies on random-walk data; show the best has a "great" Sharpe purely by luck.
- Compute the number of trades needed for a statistically significant edge given your win-rate/payoff.

## Module 20: Backtesting theory and pitfalls (Weeks 24-25)

**Topics**
1. Event-driven vs vectorised backtesting; when each is appropriate (options strategies need event-driven or careful vectorised design)
2. **The seven deadly sins:**
   1. Look-ahead bias
   2. Survivorship bias
   3. Overfitting / curve-fitting (too many parameters)
   4. Ignoring costs and slippage
   5. Unrealistic fills (assuming mid-price or LTP fill; ignoring bid-ask, depth, partial fills)
   6. Ignoring margin/capital constraints (position sizing vs real margin)
   7. Data snooping / testing on the same period used to design
3. Realistic option fill models: fill at bid/ask ± slippage, wider spreads at open/close/expiry, volume-participation limits
4. **Validation protocols:** train/validation/test split by time, walk-forward analysis, k-fold purged cross-validation, parameter-stability heatmaps (plateaus good, spikes bad), Monte-Carlo trade-shuffling, bootstrapped equity curves, stress-period testing (2008-like, 2020 crash, election days), perturbation tests
5. Benchmarks: compare to buy-and-hold index, to a random-entry strategy with the same exit rules, and to a simple baseline
6. **Out-of-sample discipline:** one final holdout set you touch *once*
7. Reporting: tear-sheet with equity curve, drawdowns, monthly returns heatmap, trade distribution, per-regime performance, cost sensitivity

**Exercises**
- Build a minimal event-driven backtester (events: market data → signal → order → fill → portfolio update) with a pluggable cost/slippage model.
- Take a deliberately silly strategy ("buy when Nifty closes on a Tuesday") and show how easy it is to get a pretty backtest by tweaking parameters; then show it failing out-of-sample.

### ✅ GATE D
- [ ] Python toolkit: data loader, BSM/Greeks library, cost model, metrics, all unit-tested
- [ ] A reproducible dataset (≥ 3 years) with documented quality checks
- [ ] A working backtester that you've validated against a hand-computed example
- [ ] You can explain each of the seven biases and show where your code defends against it

---

# STAGE E: ALGORITHM DESIGN (Weeks 26-34)

## Module 21: Strategy research process (Week 26)

**The research loop**
```
Idea → Hypothesis (why should this make money? who is on the other side?)
     → Minimal backtest → Reject quickly or refine (limited parameters)
     → Robustness battery → Out-of-sample → Paper trade → Tiny live → Scale/kill
```

**Where edges plausibly come from**
1. **Risk premia:** being paid for bearing risk (variance risk premium, carry). Persistent but with tail risk.
2. **Behavioural patterns:** overreaction, expiry flows, retail positioning. Fades as it is arbitraged.
3. **Structural/microstructure:** index rebalances, expiry mechanics, bid-ask capture. Needs speed/infra; mostly not retail.
4. **Information/speed:** not available to retail.
5. **Execution alpha:** reducing slippage and costs, which is real and under-appreciated.

**Questions every strategy must answer**
- Who is the loser on the other side, and why do they keep losing?
- Why hasn't this been arbitraged away?
- What regime kills it? How will I detect that regime?
- What is the capacity (ok for ₹50k; irrelevant at scale)?
- What is the worst realistic day and can I survive it?

**Exercise:** write 10 one-paragraph strategy hypotheses with the answers above. Kill 7 before coding.

## Module 22: Strategy families: designs to study (Weeks 27-29)

For each family you'll produce a design doc: hypothesis, signals, entry/exit/sizing, risk controls, parameters (kept few!), expected failure regimes, backtest plan.

### 22.1 Defined-risk premium selling (volatility risk premium harvest)
- Weekly/monthly iron condors or credit spreads at fixed delta (e.g., 10-16Δ short strikes), wings at fixed width
- Filters: IV-rank/percentile, VIX level, event calendar avoidance, trend filter
- Management: profit target (e.g., 50%), loss stop (e.g., 1-2× credit), time stop (exit N days before expiry), delta-breach adjustments
- Study: effect of wing width, entry day-of-week, DTE (days to expiry), stop levels. Show that results depend heavily on the left-tail days.

### 22.2 Directional debit-spread / trend following
- Trend filters (moving-average slope, breakout of N-day high, ADX) → trade bull call / bear put spreads
- Expected: low win rate, positive skew, long losing streaks. Psychological test.

### 22.3 Volatility mean-reversion
- Trade IV-rank extremes, e.g., buy long-vol structures (calendars, straddles) when IV is cheap vs realised, sell when rich. Compare IV to forecast realised vol (GARCH/HAR).

### 22.4 Event / IV-crush strategies
- Sell premium into events with defined risk; or buy ahead of known-ramp IV and exit before event. Focus on the IV term-structure around events.

### 22.5 Futures-based systematic strategies
- Time-series momentum, mean reversion (RSI/Bollinger on liquid futures), opening range breakout, VWAP reversion, gap fade/continuation. Careful with intraday costs and slippage; test with 1-min data.

### 22.6 Statistical arbitrage / pairs (stretch)
- Cointegrated pairs (e.g., HDFCBANK vs ICICIBANK futures, Nifty vs Bank Nifty ratio); hedge ratios, z-score entries, stops on cointegration break.

### 22.7 Delta-hedged volatility trading (stretch)
- Systematic gamma scalping or short-gamma with dynamic delta hedging; needs tick-ish data and careful cost modelling. Usually costs destroy retail P&L here, so learn *why*.

### 22.8 What to avoid
- Martingale/averaging-down, "100% win-rate" claims, selling naked options with tiny capital, high-frequency/latency plays, strategies whose edge is only visible in a single year, black-box ML with no economic rationale.

**Exercises**
- Pick **two** families (recommendation: 22.1 and 22.5 or 22.2) and write complete design docs.
- For each, define the exact entry/exit rules in pseudo-code that a stranger could implement identically.

## Module 23: Building and testing the strategies (Weeks 30-31)

**Topics**
1. Implement the strategy as a class with a clean interface: `on_bar()/on_tick()`, `generate_signals()`, `size_position()`, `risk_checks()`
2. Backtest with the realistic fill/cost model (Module 20)
3. Parameter sensitivity: heatmaps and stability regions; keep to ≤ 3-4 free parameters
4. Walk-forward optimisation and out-of-sample test; Monte-Carlo resampling of trades; regime breakdown
5. Stress tests: worst day, worst week, gap-open scenarios, IV spike, liquidity drought, doubled slippage, margin hikes
6. Sizing at ₹50k: confirm lot and margin feasibility; if infeasible, **say so honestly**, and test on paper or with a larger notional to learn
7. Decision rule: strategy is promoted to paper trading only if it meets **pre-registered acceptance criteria**. Example: net profit factor > 1.3 after 2× costs; max drawdown < X% of capital; positive in ≥ 70% of walk-forward windows; no single trade > 20% of total profit; stable across neighbouring parameters

**Deliverable:** for each strategy, a tear-sheet + a one-page "Why this might fail" memo.

## Module 24: Trading system architecture (Weeks 32-33)

**Components**
```
┌────────────┐   ┌──────────────┐   ┌──────────────┐   ┌───────────────┐
│ Market data│──►│ Data handler │──►│  Strategy    │──►│ Risk manager  │
│ (WS/REST)  │   │ (clean, bars)│   │ (signals)    │   │ (limits, kill)│
└────────────┘   └──────────────┘   └──────────────┘   └──────┬────────┘
                                                              ▼
┌────────────┐   ┌──────────────┐   ┌──────────────┐   ┌───────────────┐
│ Monitoring │◄──│ Logger/Store │◄──│ Portfolio /  │◄──│ Execution     │
│ & alerts   │   │ (DB, files)  │   │ position book│   │ (broker API)  │
└────────────┘   └──────────────┘   └──────────────┘   └───────────────┘
```

**Design principles**
1. **Same code path for backtest, paper and live.** Only the data source and execution adapter change. This avoids the "worked in backtest, differs live" trap.
2. **Idempotent, stateful, restartable:** on crash/restart, reconcile local state with the broker's positions and orders before doing anything.
3. **Order management:** unique client order IDs, retries with backoff only where safe, handling of rejected/partial/pending orders, freeze-quantity slicing, price protection (limit orders with bounded slippage rather than naked market orders on illiquid strikes).
4. **Risk layer is separate and has veto power:** max order size, max open lots, max daily loss, max margin utilisation, max order rate, price sanity checks (reject if order price deviates > X% from last price), "fat finger" guard, duplicate order guard.
5. **Kill switches:** automatic (daily loss limit, error-rate threshold, data feed stale, unexpected position) and manual (one-click flatten and halt). Test them.
6. **Time handling:** exchange calendar, market hours, IST, expiry-day special rules, auto-square-off windows.
7. **Observability:** structured logs, metrics, alerts to phone (Telegram/WhatsApp/PagerDuty), daily P&L and position reports, heartbeat.
8. **Security:** secrets in env/secret manager, least-privilege API keys, IP whitelisting/static IP as per broker/SEBI, 2FA token refresh handling, no keys in git, audit trail.
9. **Deployment:** cloud VM (static IP) in Mumbai region where possible, process supervisor (systemd/supervisor/Docker), time sync (NTP), auto-restart, backups. Prefer boring tech.
10. **Compliance:** use only your own account; understand broker obligations under SEBI's retail algo framework (algo tagging/registration, order-rate thresholds, static IP, no third-party sharing) [VERIFY]. Do not sell signals/advice without SEBI registration.

**Broker/API topics**
- Authentication flows and daily token expiry; websockets for ticks; instrument master files and token mapping; rate limits; order, GTT and basket endpoints; margin calculator endpoints; historical data limits; sandbox availability
- Libraries to evaluate: `kiteconnect`, `upstox-python-sdk`, `smartapi-python`, `dhanhq`, `fyers-apiv3`, `openalgo` (open-source abstraction layer), etc.

**Suggested module layout (Python)**
```
src/
  data/        loaders, feeds, instrument master, calendar
  pricing/     bsm, iv_solver, greeks
  costs/       fee_model, slippage_model, margin_model
  strategies/  base.py, iron_condor.py, trend_spread.py
  backtest/    engine.py, fills.py, metrics.py, reports.py
  risk/        limits.py, killswitch.py, sizing.py
  execution/   broker_base.py, paper_broker.py, zerodha.py, order_manager.py
  live/        runner.py, reconcile.py, scheduler.py
  monitoring/  alerts.py, dashboards.py
tests/
notebooks/
configs/
journal/
```

**Exercises**
- Implement `PaperBroker` that fills against recorded order-book/quote snapshots with realistic latency and slippage; run the *same* strategy class against it and the backtester.
- Write a reconciliation routine and test it by "killing" the process mid-trade.
- Write chaos tests: feed stale data, duplicate ticks, API 500s, partial fills, and verify the system fails *safe*.

## Module 25: Stage E consolidation (Week 34)

**Capstone E:** A complete, documented trading system running in **paper mode** against live market data for ≥ 4 weeks, with automated daily reports, alerts and passing chaos tests.

### ✅ GATE E
- [ ] ≥ 1 strategy passed pre-registered acceptance criteria on walk-forward + out-of-sample data after realistic costs
- [ ] System architecture implemented with separate risk layer and tested kill switches
- [ ] ≥ 4 weeks of paper trading; live vs backtest discrepancy explained (slippage, fills, timing) and within tolerance
- [ ] Reviewer (a friend with trading/engineering experience or a mentor) has read your code and design doc

---

# STAGE F: GO-LIVE & OPERATIONS (Weeks 35-40+)

## Module 26: The staged live-deployment ladder (Weeks 35-38)

Never go from paper to full size. Use a ladder with explicit promotion/demotion rules.

| Rung | Size | Duration (min) | Promotion criteria | Demotion trigger |
|---|---|---|---|---|
| **L0** | Paper | 4+ weeks | Gate E | n/a |
| **L1** | 1 lot, defined-risk only, ≤ 25% of capital at risk | 4-8 weeks | Execution matches paper within tolerance; zero rule violations; no operational incident | Any risk-limit breach or unexplained P&L gap |
| **L2** | 1-2 lots, same strategy | 8-12 weeks | Net P&L positive or drawdown within backtested envelope; slippage ≤ modelled | Drawdown > 1.5× backtest max |
| **L3** | Scale with capital growth only (no top-ups to "catch up") | Ongoing | Sharpe/expectancy consistent with research; policy adhered to | Edge decay signals (below) |

**Hard rules**
- No increasing size after a win streak; scale only on schedule and criteria.
- No top-ups after a loss streak.
- If a kill-switch triggers, you do a post-mortem *before* restarting.

## Module 27: Monitoring, post-mortems and edge decay (Weeks 39-40)

**Daily:** P&L, positions, margin utilisation, fills vs expected, errors, data gaps.
**Weekly:** strategy vs backtest distribution (is live performance inside the simulated envelope?), cost drift, journal review.
**Monthly:** parameter stability, regime analysis, tax records, ledger reconciliation with broker P&L.

**Edge-decay detection**
- Rolling expectancy and Sharpe vs the bootstrapped backtest band
- CUSUM / Bayesian change-point tests on trade P&L
- Increased slippage or fill rejection
- Regime shift in realised vs implied vol, spreads, or exchange rules (e.g., SEBI changing expiries/lot sizes invalidates old calibrations)

**Retirement policy:** pre-define when a strategy is paused (e.g., live max DD > backtest max DD × 1.5, or 3 consecutive months below the lower confidence band). Pause first, investigate after.

**Operational runbook:** a one-page doc for each failure type: feed down, broker API down, token expired, position mismatch, margin shortfall, exchange halt, circuit hit, power/internet loss, laptop theft/compromise.

## Module 28: Business, tax and ongoing learning (Week 40+)

- Maintain books: trade-wise ledger, charges, turnover, P&L statement, balance sheet items as your CA requires
- Advance tax planning; audit readiness
- Reinvestment rule: e.g., withdraw X% of profits monthly into index funds/emergency fund; this *is* your passive-income engine
- Keep learning: read papers, post-mortems, keep a research backlog
- Optionally add other uncorrelated income streams

### ✅ GATE F (steady-state "licence to operate")
- [ ] Runbook, risk policy and kill-switch tests current
- [ ] Monthly review cadence maintained for 3 consecutive months
- [ ] Taxes and books up to date

---

# 11. Realistic expectations at ₹50,000

| Question | Honest answer |
|---|---|
| Can ₹50k trade index options? | Only 1 lot of defined-risk spreads, and sometimes margin or contract-value rules make even that tight. Check current lot sizes/margins. |
| What return is plausible for a *good* systematic defined-risk strategy? | Low-to-mid single-digit % per month in good periods, with losing months and drawdowns of 10-25%. After costs and tax, less. |
| Net ₹1 lakh/month? | Would need roughly ₹20-50 lakh+ deployed at realistic returns, or a genuinely rare edge. Treat ₹50k as **tuition money** for a skill, plus a seed. |
| Best use of the first 12 months? | Learn, build, paper trade, and take the *tiny live* phase as a stress test of process, not an income plan. |
| Parallel plan | Systematically invest savings in index funds; use F&O profits (if any) to scale gradually. |

---

# 12. Repository plan for this project

```
fno-course/
  README.md                  <- this curriculum
  journal/                   <- learning + trade journal
  notebooks/
    01_costs_and_margin.ipynb
    02_payoffs_and_pricing.ipynb
    03_greeks_dashboard.ipynb
    04_volatility_study.ipynb
    05_backtest_lab.ipynb
  src/                       <- see Module 24 layout
  data/ (git-ignored)        <- raw + processed data
  docs/
    risk_policy.md
    runbook.md
    strategy_designs/
```

---

# 13. Resources

**Books**
- *Options, Futures, and Other Derivatives*, John Hull (the standard reference)
- *Option Volatility and Pricing*, Sheldon Natenberg (the practitioner's classic)
- *Dynamic Hedging*, Nassim Taleb (advanced; risk and hedging mindset)
- *Trading Options Greeks*, Dan Passarelli
- *Volatility Trading*, Euan Sinclair; *Positional Option Trading*, Euan Sinclair
- *Advances in Financial Machine Learning*, Marcos López de Prado (validation, backtest overfitting)
- *Quantitative Trading*, Ernest Chan; *Algorithmic Trading*, Ernest Chan
- *Evidence-Based Technical Analysis*, David Aronson (testing discipline)
- *Fooled by Randomness*, Nassim Taleb; *Thinking, Fast and Slow*, Daniel Kahneman
- *Trading in the Zone*, Mark Douglas (psychology)

**India-specific**
- NSE Academy / NISM modules: **NISM Series VIII (Equity Derivatives)** certification is highly recommended
- Zerodha Varsity (free; Options Theory, Futures Trading, Risk Management, Markets & Taxation modules)
- SEBI studies on individual F&O traders' profitability (read the primary PDFs)
- NSE circulars on lot sizes, expiries, margins; your broker's algo-trading policy

**Technical**
- `pandas`, `numpy`, `scipy`, `statsmodels`, `arch` (GARCH), `vectorbt`, `backtrader`, `py_vollib`, `QuantLib`
- Papers on backtest overfitting (Bailey & López de Prado), the Deflated Sharpe Ratio, variance risk premium literature

---

# 14. Weekly schedule (summary)

| Week | Module | Output |
|---|---|---|
| 1 | Market structure | Annotated option chain & order book |
| 2 | Derivatives 101 | Contract spec & basis spreadsheet |
| 3 | Costs, margin, tax, regulation | Cost model; CA-reviewed tax memo |
| 4 | Broker & setup | Repo + API "hello world" |
| 5 | Stage A capstone | **Gate A** |
| 6 | Option basics & payoffs | Payoff plots; put-call parity check |
| 7 | Pricing | BSM + IV solver |
| 8-9 | Greeks | Greeks dashboard; hedging sim |
| 10-11 | Volatility | VRP study; IV-rank table |
| 12-13 | Strategy catalogue | Strategy library (classes) |
| 14 | Adjustments & expiry | Trade-plan template; crash replay |
| 15 | Futures & OI analytics | Tested "indicator" claims |
| 16 | Stage B capstone | **Gate B** (+ paper trades logged) |
| 17 | Risk & sizing | Monte-Carlo ruin analysis |
| 18 | Psychology | Journal system |
| 19 | Stage C capstone | **Gate C** (Risk & Conduct Policy) |
| 20-21 | Python for trading | Tested data/pricing modules |
| 22 | Data | Clean 3-yr dataset |
| 23 | Statistics | Multiple-testing demo |
| 24-25 | Backtesting | Event-driven backtester; **Gate D** |
| 26 | Research process | 10 hypotheses → 3 survivors |
| 27-29 | Strategy families | 2 full design docs |
| 30-31 | Build & validate | Tear-sheets; acceptance decision |
| 32-33 | Architecture | Paper broker, risk layer, kill switches |
| 34 | Stage E capstone | **Gate E** (4 wks paper running) |
| 35-38 | Live ladder L1-L2 | Tiny-size live trading |
| 39-40+ | Monitoring & ops | Runbook; monthly review cadence; **Gate F** |

---

# 15. Red flags: when to stop and reassess

- You've broken your own risk rules twice in a month
- You're topping up capital to "recover" losses
- You're trading more because you're losing (or because you're winning)
- You're hiding trades or losses from people close to you
- A strategy "can't lose": check for look-ahead bias, bad fills, or tail risk you haven't seen yet
- Anyone promises guaranteed returns, sells "algo" subscriptions with fixed profit claims, or asks for your API keys or broker password

---

*This curriculum is educational material, not investment, tax or legal advice. Derivatives trading carries substantial risk of loss. Verify every [VERIFY] item against the latest exchange, SEBI and broker documentation.*
