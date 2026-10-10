# Module 1: How the Indian Market Works (Week 1)

**By the end of this module you can:**
- Name every institution that touches your trade and say what each one does.
- Trace one order from your phone to settlement.
- Read an order book and quantify slippage.
- Explain every common order type and when each can hurt you.
- Explain circuit breakers, price bands, freeze quantity and trading hours.

**Free companion reading:** Zerodha Varsity → *Introduction to Stock Markets* (all chapters on market structure, order types and the order book).

---

## 1.1 Why markets exist

A market lets people who want to buy meet people who want to sell, at a price both accept, **with confidence that the other side will honour the deal**. Everything in this module is machinery that makes those two things (matching and trust) work at scale.

In India, the two national exchanges are:

| | **NSE** (National Stock Exchange) | **BSE** (Bombay Stock Exchange) |
|---|---|---|
| Flagship index | Nifty 50 | Sensex |
| Derivatives | By far the largest derivatives exchange by contracts traded | Sensex/Bankex options, much smaller volume |
| Clearing corporation | NSE Clearing Ltd (NCL) | Indian Clearing Corporation Ltd (ICCL) |

Most retail F&O volume, and nearly all liquidity you'll care about for algos, is on **NSE**.

---

## 1.2 The cast of characters

```
        ┌───────────────────────────┐
        │   SEBI (the regulator)    │  makes rules, audits, punishes
        └─────────────┬─────────────┘
                      │ supervises
 ┌────────────────────┼─────────────────────────────┐
 ▼                    ▼                             ▼
Exchange (NSE)   Clearing Corp (NCL)         Depository (CDSL/NSDL)
matches orders   guarantees the trade,       holds your shares
publishes prices collects margins,           in electronic form
                 settles money
 ▲
 │ orders flow in
Broker (Zerodha/Upstox/…)  ← you (app, web, API)
```

| Entity | What it does | Why you care |
|---|---|---|
| **SEBI** | Capital markets regulator. Sets rules on brokers, exchanges, derivatives, algos, advice. | Its circulars change lot sizes, expiries, margins and what algos are allowed. They can invalidate a strategy overnight. |
| **Exchange (NSE/BSE)** | Runs the electronic order-matching engine. Publishes prices, contract specs, circulars. | Defines the contracts you trade and the rules (price bands, freeze qty, trading hours). |
| **Clearing corporation** | Becomes the **counterparty to every trade** (novation): to the buyer it is the seller, to the seller it is the buyer. Collects margin, marks positions to market daily, runs a default fund. | This is why you don't worry about *who* is on the other side. It's also why margins exist. |
| **Clearing member** | Large entity (often your broker) that is admitted to the clearing corporation. | Your broker's financial health matters indirectly. |
| **Broker (trading member)** | Gives you access to the exchange, holds your cash margin, enforces risk checks, charges brokerage. | Its app/API quality, outage record and risk rules affect every trade, especially on expiry day. |
| **Depository (CDSL/NSDL)** | Holds securities electronically (demat). | Matters for equity delivery and for **physical settlement** of stock F&O. |
| **Banks / UPI / payment systems** | Move your money. | Funding delays can cost you opportunity. |

💡 **Insight for algos:** When your order is rejected, it can be rejected by the *broker's risk system*, the *exchange* (price band, freeze quantity, invalid strike) or the *clearing corporation/margin* layer. Your algo must log which layer said no.

---

## 1.3 Primary vs secondary market (briefly)

- **Primary market:** companies issue new shares (IPOs, FPOs). Money goes to the company.
- **Secondary market:** investors trade existing securities among themselves on an exchange. Money goes seller → buyer. F&O is entirely a secondary-market activity.

**Derivatives** do not create shares or fund companies. They're contracts whose value *derives* from an underlying.

---

## 1.4 Market segments and timings

| Segment | What trades | Settlement |
|---|---|---|
| **Cash / Equity** | Shares, ETFs | **T+1** (trade day plus 1 working day) delivery |
| **F&O (Equity Derivatives)** | Index & stock futures and options | Daily MTM cash settlement; final settlement at expiry (cash for index, physical for stock) |
| **Currency, Commodity** | USD-INR etc.; MCX/NCDEX commodities | Separate exchanges/hours |

**Equity & F&O trading hours:** 9:15 to 15:30 IST, Monday to Friday, excluding exchange holidays. There's a pre-open session before 9:15 for equity price discovery (approximately 9:00 to 9:15), and the F&O segment has its own pre-open arrangements **[VERIFY current timings and holiday calendar on nseindia.com]**.

💡 **Insight:** Use the **official exchange holiday calendar** programmatically in your code. Never hard-code "Mon-Fri". Muhurat trading, special sessions, and weekend sessions (e.g., disaster-recovery drills) exist.

---

## 1.5 Anatomy of an order: from tap to trade to settlement

```
1. You tap "SELL" in the app / your algo calls place_order()
2. Broker's Risk Management System (RMS) checks:
      - do you have margin? - quantity within limits? - price sane?
3. Order goes to the exchange via the broker's trading terminal (co-located / leased line)
4. Exchange matching engine:
      - checks price band, freeze qty, lot size, tick size
      - places it in the order book (if limit and not immediately matchable)
        or matches immediately against resting orders
5. Trade confirmation returns to broker → your app (milliseconds to seconds)
6. Trade details flow to the Clearing Corporation:
      - NCL becomes the counterparty (novation)
      - it computes your margin requirement and daily MTM
7. End of day: positions are marked to settlement price; MTM profit/loss moves
   between accounts via the clearing system; your broker updates your ledger.
8. At expiry: final settlement (cash for index, physical delivery for stock F&O).
```

### Price-time priority
The matching engine fills the **best price first**. At the same price, the **earliest order** first. This is why *queue position* matters for limit orders, and why tiny price improvements (one tick) jump the queue.

---

## 1.6 The order book: your most important picture

A simplified order book for one Nifty option (call at 24,000 strike):

```
        BIDS (buyers)                    ASKS (sellers)
   price   qty   orders             price   qty   orders
  149.85    130     3     │        150.00    65     1
  149.80    260     5     │        150.10    65     2
  149.70    195     4     │        150.30   130     3
  149.60    390     6     │        150.60   260     5
```

- **Best bid** = highest price a buyer will pay (149.85). **Best ask/offer** = lowest price a seller will accept (150.00).
- **Bid-ask spread** = 150.00 − 149.85 = **0.15**. This is a real cost: if you buy at the ask and immediately sell at the bid you lose the spread.
- **Mid price** = (149.85 + 150.00) / 2 = 149.925. Backtests that assume you trade at mid are **optimistic**.
- **Depth** = quantity available at each price level.
- **LTP** (last traded price) is *not* a price you can necessarily trade at now.

### Slippage: worked example
You send a **market order to buy 3 lots (195 units)**:

| Fill | Qty | Price | Value |
|---|---|---|---|
| Best ask | 65 | 150.00 | ₹9,750.00 |
| Next ask | 65 | 150.10 | ₹9,756.50 |
| Next ask | 65 | 150.30 | ₹9,769.50 |
| **Total** | **195** | | **₹29,276.00** |

Average price = 29,276 / 195 = **150.133**. You expected 150.00, so slippage = 0.133 per unit = **₹26** on the order. In a thin far-OTM option or a fast market the book can be much emptier, and slippage can exceed the whole premium you hoped to earn.

⚠️ **Trap:** Market orders in options are dangerous. Use limit orders with a sensible price (e.g., at/near the touch) and think about what happens if they don't fill.

🔧 **Try it:** On a live option chain, pick one liquid ATM option and one far-OTM option for the same expiry. Write down for each: bid, ask, spread, spread as % of price, and total depth at top 5 levels. Which would you rather trade, and why?

💡 **Insight for algos:** Your backtest's fill model (Module 20 later) must be built from exactly this: bid/ask, depth, spread widening at open/close/expiry. "I filled at LTP" is the most common reason a backtest lies.

---

## 1.7 Order types

| Order type | What it does | When it helps | How it can hurt |
|---|---|---|---|
| **Market** | Execute immediately at best available prices | Urgent exit in a liquid contract | Walks the book (slippage); may be rejected for options by exchange rules, since market orders in illiquid options can be blocked or converted **[VERIFY broker]** |
| **Limit** | Execute only at your price or better | Control cost; passive entry | May never fill; you can miss the move |
| **SL (Stop-loss limit)** | When trigger price is touched, a limit order is placed | Controls max price paid on exit | In a gap/fast move the limit may not fill, leaving you in the position |
| **SL-M (Stop-loss market)** | When trigger touched, a market order is placed | Ensures exit | Can fill far from trigger; some segments restrict SL-M for options |
| **IOC (Immediate or Cancel)** | Fill what you can immediately, cancel the rest | Algos that avoid resting orders | Partial fills |
| **Day** | Valid till end of day | Default | n/a |
| **GTT / GTC** | Order persists beyond a day (broker feature, not an exchange order type) | Long-term triggers | Not guaranteed; trigger fires a fresh order |
| **Bracket / Cover orders** | Entry + stop-loss (+ target) bundled | Intraday discipline | Many brokers have restricted or retired these **[VERIFY]**; margin benefits have been reduced |
| **Basket / Multi-leg** | Send several legs together | Spreads/condors | Not atomic: legs can fill at different times → leg risk |
| **Iceberg / slicing** | Splits a big order into pieces | Large size | Complexity; broker/exchange rules |

### Important concepts

- **Tick size:** the smallest price increment (₹0.05 for most options; futures have their own tick). Prices must be on the tick.
- **Lot size:** you trade only in multiples of the lot.
- **Freeze quantity:** the exchange rejects a single order larger than a set maximum quantity per contract. Larger orders must be sliced into several orders. Algos need slicing logic **[VERIFY current freeze quantities in NSE circulars]**.
- **Order price protection:** the exchange/broker rejects limit prices far from the market (e.g., a buy limit at 10× LTP), to prevent fat-finger errors.

⚠️ **Trap (multi-leg orders):** A condor sent as four separate orders can fill 3 legs and miss the 4th, leaving you with unintended, **undefined risk**. Algo rule: send the **long (protective) legs first**, then the short legs; or use broker-provided spread orders where available and verify fills.

---

## 1.8 Circuit breakers, price bands and safety rails

1. **Index-level market-wide circuit breakers:** If a main index (Nifty/Sensex) moves by 10%, 15% or 20% from the previous close, trading halts for a period depending on the time of day. These are rare but do occur (e.g., March 2020). Your algo must survive a halt: no data, no orders, then a violent restart.
2. **Stock-level price bands/circuit limits:** Individual stocks have daily limits (2%, 5%, 10%, 20% bands). Stock F&O contracts use **dynamic price bands** around reference prices **[VERIFY rules]**.
3. **Dynamic price bands for derivatives:** orders outside a band are rejected; contracts in "ban" (see below) can only be reduced.
4. **F&O ban list:** if a stock's total OI exceeds 95% of the market-wide position limit, it enters the **ban period**: no fresh positions allowed, only reducing existing ones. A stock can exit the ban when OI falls below a threshold **[VERIFY thresholds]**.
5. **Trade-to-trade, ASM/GSM surveillance** on cash stocks (less relevant for F&O).

🧠 **Check:** You are short a stock future and the stock enters the F&O ban list. What can and can't you do?

---

## 1.9 Settlement, margin and the daily cycle

- **Cash equity:** you pay/receive on **T+1**.
- **Futures & options:** positions are **marked to market (MTM) every day**. Profit and loss are cash-settled daily between you and the clearing corporation (via your broker). Options buyers pay full premium upfront; sellers post margin.
- **At expiry:**
  - **Index F&O → cash-settled** at the final settlement price (a price derived from the underlying index at expiry; check the exchange's definition **[VERIFY]**).
  - **Stock F&O → physically settled:** ITM positions held at expiry become delivery obligations (long ITM call = you must pay for and receive shares; short ITM call = deliver shares, etc.). This is a **major trap for small accounts**: margin spikes in the last days and the broker may square you off.

💡 **Insight:** Your algo needs an **expiry calendar** per product, and a rule like "flatten all stock-F&O positions at least one day before expiry."

---

## 1.10 Participants and who is on the other side

| Participant | Typical behaviour | Why it matters |
|---|---|---|
| **Retail** | Small, often option buyers; heavily concentrated in expiry-day trades | Large share of volume; SEBI data shows most lose money |
| **Proprietary traders / HFT / market makers** | Fast, large, systematic; capture spreads; arbitrage | Your counterparty on many trades; faster and better capitalised |
| **FIIs (foreign institutional investors)** | Large directional/hedging positions in index futures/options | Their positioning is published daily; many "indicators" are derived from it |
| **DIIs, mutual funds** | Hedging, long-only, some arbitrage | Provide flows that dampen or amplify moves |
| **Hedgers (corporates, funds)** | Offset risk | The reason derivatives exist |

💡 **Insight:** Before trading any pattern, ask: *"Who is on the other side, and why are they willing to lose?"* If you can't answer, you probably don't have an edge.

---

## 1.11 Exercises

1. **Order-book annotation (60 min).** Open a live option chain (NSE website or your broker). Screenshot one liquid and one illiquid contract. Annotate: best bid/ask, spread, depth, OI, volume, LTP. Compute spread as % of mid-price for both.
2. **Slippage calculator (45 min).** In Python or a spreadsheet, write `avg_fill_price(book_side, qty)` that walks a list of `(price, qty)` levels. Test with the example in 1.6 (answer 150.1333). Add a function returning slippage in ₹ versus the best price.
3. **Paper-order experiment (30 min).** In a broker simulator or on paper, record what you expect to happen for: (a) a market buy of 1 lot ATM option, (b) a limit buy at the bid, (c) a limit buy at the ask, (d) an SL-M sell. For each, write when it fills and the worst fill you can imagine.
4. **Calendar (30 min).** List the next 3 monthly expiries and next 4 weekly expiries for Nifty and Sensex, the expiry weekday for each, and the next 5 exchange holidays. Source: NSE/BSE circulars.
5. **Ban-list watch (15 min/day for a week).** Check the F&O ban list daily. Note which stocks appear, and look up their OI/market-wide position limit percentage.

---

## 1.12 Self-check questions

1. Why does a clearing corporation exist, and what protects you from the counterparty defaulting?
2. What is freeze quantity and why might your algo hit it?
3. What happens to a long stock-futures position that is held to expiry?
4. What's the difference between the bid-ask spread and slippage?
5. Why is "I'll set a stop-loss and I'm safe" not a guarantee?
6. List three ways an order can be rejected, and which institution rejects it in each case.
7. Why do backtests that fill at LTP overstate returns?
8. A stock enters the F&O ban list while you hold a long position. What can you do?

### Answer key

1. The clearing corporation steps in as the counterparty to both sides (novation). It collects margin, marks positions to market daily and maintains a default fund and risk-management framework, so each trader's risk is the clearing corporation's credit and not an unknown counterparty's.
2. The exchange-set maximum quantity per order for a contract. Orders above it are rejected. Large algo orders must be sliced into multiple orders.
3. Stock futures are physically settled at expiry (as of current rules **[VERIFY]**): a long position results in a delivery obligation, i.e., you must pay for and take delivery of shares. Margins also rise ahead of expiry. Close or roll before expiry unless you want delivery.
4. Spread = the gap between best bid and best ask (a property of the book). Slippage = the difference between the price you expected and the price you actually got (depends on your order size, speed, and book depth). Spread is one component of expected slippage.
5. Stop-loss orders trigger after a price touch, then become market/limit orders. In gaps, fast markets, halts, or illiquid options the fill can be far worse, or not happen at all (SL limit).
6. e.g., (a) broker RMS: insufficient margin; (b) exchange: price outside band, over freeze quantity, wrong lot/tick; (c) clearing/margin layer or ban rules: fresh positions in banned stock.
7. LTP is a past trade price. You may not be able to buy at LTP (you pay the ask) or sell at LTP (you receive the bid); costs spread plus depth impact on every trade.
8. Only reduce or close the position (e.g., sell your long). You cannot add to it.

---

## 1.13 Further reading
- Zerodha Varsity: *Introduction to Stock Markets*, *Futures Trading* (first chapters on contract mechanics).
- NSE website: *Products → Equity Derivatives → Contract specifications*, *Circulars*, *F&O ban list*, *Trading holidays*.
- SEBI website: *Legal → Circulars* (search "derivatives", "lot size", "algorithmic").
