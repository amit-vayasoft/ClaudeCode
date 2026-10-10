# Module 2: Derivatives 101 (Week 2)

**By the end of this module you can:**
- Define a derivative and say who uses them and why.
- Read the full contract specification of any NSE F&O instrument.
- Compute notional value, margin as a percent of notional, and effective leverage.
- Walk through daily mark-to-market (MTM) cash flows for a futures position.
- Compute the fair value of a future and explain basis, contango and backwardation.
- Explain open interest, and why OI and volume are different things.
- State clearly what an option is (a full treatment of payoffs comes in Module 6).

**Free companion reading:** Zerodha Varsity → *Futures Trading* (contract, margin, MTM, leverage chapters) and the first chapters of *Options Theory for Professional Trading*.

---

## 2.1 What is a derivative?

A **derivative** is a contract whose value is *derived* from something else (the **underlying**): an index, a stock, a currency, a commodity, an interest rate. You are not buying the underlying; you are buying or selling a *contract about* it.

### Why derivatives exist (three uses)

| Use | Who | Example |
|---|---|---|
| **Hedging** | Anyone who holds underlying risk and wants to reduce it | A mutual fund with ₹100 crore of stocks sells Nifty futures to protect against a market fall. A buyer of a protective put pays a premium for insurance. |
| **Speculation** | Traders who take a view with a small capital outlay | Buying a Nifty call because you expect a rally. |
| **Arbitrage** | Traders who exploit price mismatches between related instruments | Buying the stock in the cash market and selling its future when the future is too expensive. |

⚠️ **Trap:** Most retail activity is speculation, not hedging. Speculation is a zero-sum game before costs (one side's gain is the other's loss) and **negative-sum after costs**. Keep this in mind: for you to win, someone else has to lose more than costs.

---

## 2.2 Forward vs future

A **forward** is a private agreement to buy/sell something at a fixed price on a future date. It's flexible but carries counterparty risk and is illiquid.

A **future** is a **standardised forward traded on an exchange**: fixed lot size, fixed expiry, guaranteed by the clearing corporation, marked to market daily. Standardisation gives liquidity and removes counterparty risk. This is what retail traders access.

---

## 2.3 Underlyings and contract families

| Family | Underlyings | Settlement at expiry |
|---|---|---|
| **Index futures & options** | Nifty 50, Bank Nifty, Fin Nifty, Midcap Nifty, Nifty Next 50 (NSE); Sensex, Bankex (BSE) **[VERIFY which are currently listed and which have weekly expiry]** | **Cash-settled** |
| **Stock futures & options** | About 180-220 liquid stocks eligible for F&O **[VERIFY, list changes]** | **Physically settled** (delivery of shares) |

Important: the set of products allowed weekly expiries has been reduced by SEBI: in the current framework each exchange may offer weekly expiry on **one** benchmark index **[VERIFY]**. Monthly (and quarterly/half-yearly for some) contracts exist for the others.

**European vs American exercise:** In India, F&O options are **European-style** (exercisable only at expiry), not American. This matters for pricing (Black-Scholes applies cleanly; Module 7) and for how you think about assignment: no one can exercise a short option against you early.

---

## 2.4 Contract specifications: how to read one

Every F&O contract on NSE is defined by a spec. For example (illustrative, **[VERIFY]** all numbers on NSE):

| Field | Nifty 50 Futures | Nifty 50 Options | Stock Options (e.g., RELIANCE) |
|---|---|---|---|
| **Underlying** | Nifty 50 index | Nifty 50 index | Stock |
| **Lot size** | ~65 **[VERIFY]** | ~65 | e.g., 250-500, varies |
| **Tick size** | ₹0.05 on futures price (check) | ₹0.05 on premium | ₹0.05 |
| **Expiry cycle** | Monthly (last Tuesday-type rule; day-of-week changes occur) | Weekly + monthly | Monthly |
| **Strike interval** | n/a | e.g., 50 points near ATM, wider far out | varies by price |
| **Settlement** | Cash | Cash | Physical |
| **Style** | n/a | European | European |
| **Price quote** | Index points | Premium in points | Premium in ₹ per share |
| **Multiplier** | Lot size × price | Lot size × premium | Lot size × premium |

### Lot size: why it matters
The **lot size** is the minimum tradeable unit. Exchanges set lot sizes so that the *contract value* (index level × lot size) sits within a prescribed band. The SEBI framework from late 2024 raised the minimum contract value for new derivatives contracts to around **₹15 lakh** (with a target range of roughly ₹15-20 lakh) **[VERIFY]**, which is why lot sizes were revised upward. Lots are reviewed periodically (semi-annually) and can **change mid-life**; a historical backtest must use the lot size valid on each date.

💡 **Insight for algos:** Store a table `(symbol, effective_date, lot_size)`. Never hard-code a single lot size. Position sizing, margin, and P&L conversions depend on it.

### Contract naming
Example: `NIFTY25OCT24000CE` = Nifty, expiry month Oct 2025 (weekly formats include date), strike 24,000, **CE** = call (European). `PE` = put. Futures look like `NIFTY25OCTFUT`. Your broker's **instrument master** file maps these symbols to exchange tokens; your code should use the instrument master, never hand-built strings.

---

## 2.5 Notional value, margin and leverage

- **Notional (contract) value** = price × lot size.
- **Margin** = the good-faith deposit required to hold the position (not a fee; returned when you close).
- **Leverage** = notional ÷ margin.

### Worked example: 1 lot Nifty future
- Nifty future price: ₹24,000. Lot size: 65.
- Notional = 24,000 × 65 = **₹15,60,000**.
- Suppose margin ≈ ₹1,60,000 **[VERIFY with broker's margin calculator]**.
- Leverage = 15,60,000 / 1,60,000 ≈ **9.75×**.

### What leverage does to P&L
Per-lot P&L = (price change in points) × 65.

| Nifty move | Points | P&L per lot (long) | As % of ₹1.6L margin |
|---|---|---|---|
| +1% | +240 | +₹15,600 | +9.75% |
| −1% | −240 | −₹15,600 | −9.75% |
| −5% | −1,200 | −₹78,000 | −48.75% |
| −10% | −2,400 | −₹1,56,000 | −97.5% (margin nearly gone) |

A 1% index move changes your margin capital by nearly 10%. **Leverage is a magnifier, not a source of edge.**

⚠️ **Trap (₹50,000 account):** One Nifty future lot needs roughly 3× your whole capital in margin. This is why small accounts end up in option spreads. Even there, a single bad day is a meaningful part of your capital. Take this seriously when choosing instruments.

🔧 **Try it:** Use your broker's margin calculator for: (a) 1 lot Nifty future, (b) short 1 lot ATM call (naked), (c) a bull put spread (sell 24,000 PE, buy 23,800 PE), (d) buy 1 lot ATM call. Note margin as a percent of notional for each. Why is (c) so much cheaper than (b)?

---

## 2.6 Mark-to-market (MTM): daily settlement of futures

Futures profit and loss do not wait until you close. Every day the exchange computes a **settlement price** and credits/debits the difference to your account.

### Worked example (long 1 lot at 24,000; lot 65)

| Day | Event | Price used | Daily MTM | Cumulative |
|---|---|---|---|---|
| 0 | Buy 1 lot at 24,000 | entry 24,000 | | |
| 0 close | Settlement price 24,100 | (24,100 − 24,000) × 65 | **+₹6,500** | +6,500 |
| 1 close | Settlement price 23,950 | (23,950 − 24,100) × 65 | **−₹9,750** | −3,250 |
| 2 | Sell at 24,050 | (24,050 − 23,950) × 65 | **+₹6,500** | **+3,250** |

Check: total = (24,050 − 24,000) × 65 = ₹3,250 ✓. MTM doesn't change the total, only **when cash moves**. Losses are collected daily, so if your account lacks funds you receive a **margin call** or the broker squares you off.

💡 **Insight:** For algos, "available cash" changes with MTM during the day. Your risk module should use *live* margin and P&L, and re-check before placing orders.

---

## 2.7 Fair value of a future, basis, contango and backwardation

### Cost of carry (the idea)
Instead of buying a future, you could buy the underlying now, financing it at interest rate *r* and receiving dividends *q*. So a future shouldn't trade far from the cost of that alternative:

```
Fair future ≈ Spot × e^((r − q) × T)
Basis       = Future − Spot
```
where *T* = time to expiry in years, *r* = financing rate, *q* = dividend yield.

### Worked example
- Spot Nifty = 24,000, r = 6.5%, q = 1.3%, 30 days to expiry (T = 30/365).
- Fair future = 24,000 × e^((0.065 − 0.013) × 30/365) = 24,000 × e^0.004274 ≈ **24,102.8**
- Fair basis ≈ **+102.8 points**, an annualised premium of 102.8/24,000 × 365/30 ≈ **5.2%**.

### Contango and backwardation
- **Contango:** future > spot (normal: carry cost > dividends). Futures price converges *down* to spot at expiry if spot doesn't move.
- **Backwardation:** future < spot (unusual for indices; signals stress, heavy hedging demand or dividend/ex-date effects).

At expiry, **future = spot** (convergence). This is the engine of cash-futures arbitrage.

### Cash-futures arbitrage (why retail rarely wins)
If the future trades at 24,150 when fair value is 24,102.8: sell the future, buy the stock basket, hold to expiry. Profit ≈ 24,150 − 24,102.8 ≈ 47 points before costs, locked in. The catch: you need to buy all 50 Nifty stocks in proper weights (costly, impractical for small capital), pay costs on both legs, and compete with firms with better pricing and execution. Use stock-level cash-future arbitrage in individual names for understanding.

⚠️ **Trap:** "Futures premium is high so the market is bullish" is a misreading. Basis mostly reflects interest rates, dividends and time. Changes in *annualised basis* relative to its own history are what carry information.

🔧 **Try it:** Compute the annualised basis for the current-month Nifty future every day for two weeks in a spreadsheet. Compare with the repo rate. Does it move toward zero as expiry approaches (it should in absolute points)?

---

## 2.8 Open interest and volume

| Term | Meaning |
|---|---|
| **Volume** | Number of contracts traded during the period (counts every trade) |
| **Open interest (OI)** | Number of contracts currently outstanding (not yet closed or expired) |

### How OI changes: worked example
Follow a ledger of who holds what. Every contract has exactly one long and one short, so total longs always equal total shorts, and that common number is the OI.

| Step | Trade | Longs outstanding | Shorts outstanding | OI |
|---|---|---|---|---|
| 1 | A buys 10 from B (both open) | A:10 | B:10 | 10 |
| 2 | C buys 5 from A (A closes 5, C opens 5) | A:5, C:5 = 10 | B:10 | 10 (unchanged: A's close, C's open) |
| 3 | B buys 4 from C (B closes 4 shorts, C closes 4 longs) | A:5, C:1 = 6 | B:6 | **6** (both closing → OI falls) |
| 4 | E buys 3 from F (both open) | 6 + 3 = 9 | 6 + 3 = 9 | **9** |

Rule: **OI rises when both sides open, falls when both sides close, and is unchanged when one opens while the other closes.** Total longs always equal total shorts: every contract has a buyer and a seller.

### How to read OI (and its limits)
OI by itself says nothing about direction. Because every contract has two sides, "OI rose" doesn't tell you whether bulls or bears are stronger. The popular four-way grid (price up + OI up = "long build-up", etc.) is a **heuristic**, not a law. We test such claims in Module 12 and often find weak or no predictive power.

---

## 2.9 Options: first look (details in Module 6)

An **option** gives the **buyer** a *right* (not obligation) and the **seller (writer)** an *obligation*:

| | Call option | Put option |
|---|---|---|
| **Buyer's right** | Buy the underlying at the strike | Sell the underlying at the strike |
| **Buyer pays** | Premium upfront | Premium upfront |
| **Seller's duty** | Sell at the strike if exercised | Buy at the strike if exercised |
| **Buyer profits when** | Underlying rises well above strike + premium | Underlying falls well below strike − premium |
| **Buyer's max loss** | Premium paid | Premium paid |
| **Seller's max profit** | Premium received | Premium received |
| **Seller's max loss** | Unlimited (call), large (put) | Large (strike × lot − premium) |

Key vocabulary: **strike** (agreed price), **premium** (option price), **expiry**, **ITM/ATM/OTM** (in/at/out of the money), **intrinsic value** (value if exercised now), **time value** (premium − intrinsic).

### Tiny worked example
Nifty at 24,000. You buy a 24,000 call for ₹150 (lot 65): cost = ₹9,750.
- Nifty expires at 24,300 → intrinsic 300 → value 300 × 65 = ₹19,500 → profit ₹9,750 (before costs).
- Nifty expires at 23,900 → option expires worthless → loss ₹9,750.
- Breakeven at expiry = strike + premium = 24,150.

The asymmetry between buyers (limited loss) and sellers (limited profit, potentially huge loss) is why options feel attractive to retail buyers and why sellers "win often, lose big." We study that trade-off properly from Module 6.

---

## 2.10 Exercises

1. **Contract-spec spreadsheet (90 min).** For five instruments: Nifty future, Bank Nifty future, Nifty weekly option, one large-cap stock future (e.g., Reliance), one mid-cap stock future. Columns: symbol, lot size, price, **notional**, margin (from broker calculator), **margin %**, **leverage**, expiry date, settlement type.
2. **Basis tracker (daily, 10 min, 2 weeks).** Record spot, current-month future, days to expiry, basis (points), annualised basis (%), repo rate. Plot.
3. **MTM simulator (45 min).** In Python/Sheets, write a function that takes entry price, a list of daily settlement prices, lot size, and quantity, and returns daily MTM cash flows and cumulative P&L. Verify against the 2.6 table.
4. **OI ledger (30 min).** Reproduce the 2.8 table in a spreadsheet and invent a 5th trade where OI is unchanged. Explain why.
5. **Fair-value calculator (30 min).** Function `fair_future(spot, r, q, days)`. Compare to the live future; investigate why differences exist (dividend dates, liquidity, expectations).

---

## 2.11 Self-check questions

1. If Nifty is 24,000 and the lot size is 65, what is the notional of one lot? If margin is ₹1.5 lakh, what is the effective leverage?
2. A future trades at a premium to spot. Is that a "bullish signal"? Why or why not?
3. You're long 2 lots of Nifty future at 24,000 (lot 65). Settlement prices on three days: 24,060, 23,900, 24,120. What are the daily MTM flows and total?
4. Why can OI rise while the price falls, and what does the "short build-up" label assume that may be wrong?
5. What are the three settlement/exercise facts that make stock options riskier to hold into expiry than index options?
6. Spot = 24,000, r = 6%, q = 1%, T = 45 days. Compute the fair future.
7. Explain why a long call has limited loss but a short call has unlimited loss. Which one needs margin?
8. Why can't you use one hard-coded lot size in a multi-year backtest?

### Answer key

1. Notional = 24,000 × 65 = ₹15,60,000. Leverage = 15,60,000 / 1,50,000 = **10.4×**.
2. No. A normal futures premium reflects interest rate minus dividends over time (cost of carry). Only a premium that is unusual versus its own history and the carry model adds information, and even that needs testing.
3. Qty = 130 units. Day 1: (24,060 − 24,000) × 130 = **+₹7,800**. Day 2: (23,900 − 24,060) × 130 = **−₹20,800**. Day 3: (24,120 − 23,900) × 130 = **+₹28,600**. Total = **+₹15,600** = (24,120 − 24,000) × 130 ✓.
4. Price down + OI up means new positions are being opened on both sides while price falls. Whether new shorts are aggressive bears or hedgers (e.g., funds hedging long equity holdings) isn't revealed. Every contract has a buyer and seller, so OI does not identify direction.
5. (a) Stock F&O is physically settled; ITM positions become delivery obligations. (b) Margins rise sharply near expiry. (c) Brokers may auto-square-off or charge penalties/higher STT on delivery; lower liquidity in many stocks widens spreads. **[VERIFY details]**
6. T = 45/365 = 0.12329. Fair = 24,000 × e^((0.06−0.01)×0.12329) = 24,000 × e^0.0061644 ≈ 24,000 × 1.006183 ≈ **24,148.4**.
7. A long call's worst case is losing the premium paid, since you're not obliged to exercise. A short call is obligated to sell at the strike and the underlying can rise without limit, so losses are unlimited. The short call needs margin; the long call needs only the premium.
8. The exchange revises lot sizes periodically. The P&L of a lot depends on the lot size at that date, so using today's lot size on old data misstates P&L, margin and capital usage.

---

## 2.12 Further reading
- Zerodha Varsity: *Futures Trading* chapters on margin, MTM, pricing; *Options Theory* first chapters.
- NSE: *Contract specifications* pages for index/stock derivatives; *Lot size revisions* circulars.
- NISM Series VIII (Equity Derivatives) workbook: contract design and settlement chapters.
