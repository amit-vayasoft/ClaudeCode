# Module 3: Costs, Margins, Taxation and Regulation (Week 3)

This is the densest module of Stage A and the one that most separates people who survive from people who don't. A strategy that looks profitable before costs is often a guaranteed loser after them, and a trader who ignores margin rules or tax can be forced out of a good position or blindsided at year-end.

**By the end of this module you can:**
- List every charge on an F&O trade and compute it by hand.
- Compute breakeven in index points for futures and options.
- Explain SPAN, exposure and extreme-loss margin, and why margins change.
- Explain how F&O income is taxed, what turnover means, and when an audit is needed.
- Summarise the recent SEBI measures on F&O and the retail algo framework.

**Free companion reading:** Zerodha Varsity → *Markets and Taxation*; your broker's *Charges* page and brokerage calculator; Income Tax Department FAQs on business income; SEBI circulars (search "Measures to strengthen equity derivatives").

> ⚠️ **Rates and rules change.** Every percentage in this module is illustrative **[VERIFY]**. STT, exchange charges and rules have been revised several times in recent years. Always check your broker's charges page and the latest circulars. Tax advice must come from a qualified Chartered Accountant (CA).

---

## 3.1 The full cost stack of an F&O trade

Every executed order incurs:

| # | Charge | Who gets it | Charged on | Notes |
|---|---|---|---|---|
| 1 | **Brokerage** | Your broker | Per executed order | Many discount brokers: flat ₹20, or 0.03% of turnover, **whichever is lower**. Full-service brokers charge more. |
| 2 | **Exchange transaction charge** | NSE/BSE | Turnover | Different rates for futures and options; rate differs by exchange. For options, applied to **premium** value. |
| 3 | **SEBI turnover fee** | SEBI | Turnover | ₹10 per crore. Tiny. |
| 4 | **STT (Securities Transaction Tax)** | Government | **Sell side** for futures (on contract value) and options (on premium) | Usually the **largest single statutory charge**, especially for futures. Revised repeatedly **[VERIFY]**. |
| 5 | **Stamp duty** | State governments | **Buy side** | Small. |
| 6 | **GST (18%)** | Government | Brokerage + exchange charges + SEBI fee (not on STT or stamp duty) | |
| 7 | **Slippage and spread** | The market (other traders) | Every trade | Not a line item on your contract note, but **often the biggest cost**. |

Also remember: data/API fees, platform subscriptions, and your time.

### Where each charge appears
Your broker's **contract note** (and the daily P&L statement) lists items 1-6 per trade. Item 7 shows up only as the gap between the price you expected and the price you got.

---

## 3.2 Worked examples (illustrative rates from the stage README)

Assumptions: Nifty lot = 65. Rates as in the table in `lessons/stage-a/README.md`.

### Example A: 1 lot Nifty future, buy and sell at the same price (24,000)
Contract value per side = 24,000 × 65 = ₹15,60,000.

| Charge | Calculation | ₹ |
|---|---|---|
| Brokerage | 2 orders × ₹20 (0.03% of 15.6L = ₹468, so flat ₹20 applies) | 40.00 |
| Exchange txn | 0.002% × (15.6L + 15.6L) | 62.40 |
| SEBI fee | 0.0001% × 31.2L | 3.12 |
| GST | 18% × (40 + 62.40 + 3.12) | 18.99 |
| STT (sell only) | 0.02% × 15.6L | 312.00 |
| Stamp duty (buy only) | 0.002% × 15.6L | 31.20 |
| **Total** | | **≈ ₹467.71** |

**Breakeven** = 467.71 / 65 ≈ **7.2 Nifty points** (before slippage). If your strategy expects to capture 5 points per trade on average, it loses money.

### Example B: ATM option, sell at ₹150 and buy back at ₹150 (1 lot)
Premium value per side = 150 × 65 = ₹9,750.

| Charge | Calculation | ₹ |
|---|---|---|
| Brokerage | 2 orders × min(₹20, 0.03% × 9,750 = ₹2.93) | 5.85 |
| Exchange txn | 0.035% × 19,500 | 6.83 |
| SEBI fee | 0.0001% × 19,500 | 0.02 |
| GST | 18% × (5.85 + 6.83 + 0.02) | 2.29 |
| STT (sell) | 0.1% × 9,750 | 9.75 |
| Stamp (buy) | 0.003% × 9,750 | 0.29 |
| **Total** | | **≈ ₹25.02** |

**Breakeven** = 25.02 / 65 ≈ **0.38 premium points**. Look at the brokerage line: because the 0.03% cap is lower than ₹20, cheap options carry far less brokerage than the flat fee suggests. (Check your own broker's rule.)

### Example C: Iron condor (8 orders in, out)
Sell 24,300 CE @ 40, buy 24,400 CE @ 20, sell 23,700 PE @ 40, buy 23,600 PE @ 20, closed later at the same prices (to isolate costs). Net credit = (40 − 20 + 40 − 20) = 40 points = ₹2,600 per lot.
Total charges ≈ **₹20.02** → about **0.8%** of the credit. Charges look tiny here. Now add the real cost:

### Example D: slippage dwarfs charges
Suppose each leg gives up just **1 point** versus mid-price on entry and exit. That's 1 point × 65 × 2 (in and out) = **₹130 per leg-pair**, per option position. For a 4-leg condor, an extra 1 point per leg round trip = **₹520**, which is **20% of the ₹2,600 credit**.

> 💡 **Insight:** For options, *spread and slippage* are the dominant cost, not statutory charges. For futures, *STT* is the dominant statutory charge. A realistic backtest models both.

🔧 **Try it:** For your broker, find the real charges page. Replace the rates in the table, recompute Examples A and B by hand, and compare with the broker's own brokerage calculator. They should agree to within a rupee or two.

---

## 3.3 A reference implementation (try writing your own first)

Write your own `fee_model.py` first. Then compare. This version reproduces the examples above.

```python
"""Illustrative F&O charge calculator. All rates are placeholders: VERIFY and update."""
from dataclasses import dataclass


@dataclass(frozen=True)
class Rates:
    brokerage_flat: float = 20.0          # Rs per executed order
    brokerage_pct_cap: float = 0.0003     # or 0.03% of turnover, whichever is lower
    txn_fut: float = 0.00002              # 0.002% of contract value
    txn_opt: float = 0.00035              # 0.035% of premium value
    sebi: float = 0.000001                # Rs 10 per crore
    stt_fut_sell: float = 0.0002          # 0.02%, sell side
    stt_opt_sell: float = 0.001           # 0.1% of premium, sell side
    stamp_fut_buy: float = 0.00002        # 0.002%, buy side
    stamp_opt_buy: float = 0.00003        # 0.003%, buy side
    gst: float = 0.18                     # on brokerage + txn + sebi


def leg_cost(kind: str, side: str, price: float, qty: int, r: Rates = Rates()) -> dict:
    """Charges for one executed order. kind: 'FUT' or 'OPT'; side: 'BUY' or 'SELL'."""
    assert kind in ("FUT", "OPT") and side in ("BUY", "SELL")
    turnover = price * qty
    brokerage = min(r.brokerage_flat, r.brokerage_pct_cap * turnover)
    txn = turnover * (r.txn_fut if kind == "FUT" else r.txn_opt)
    sebi = turnover * r.sebi
    gst = r.gst * (brokerage + txn + sebi)
    if side == "SELL":
        stt = turnover * (r.stt_fut_sell if kind == "FUT" else r.stt_opt_sell)
        stamp = 0.0
    else:
        stt = 0.0
        stamp = turnover * (r.stamp_fut_buy if kind == "FUT" else r.stamp_opt_buy)
    return dict(brokerage=brokerage, txn=txn, sebi=sebi, gst=gst, stt=stt, stamp=stamp)


def total_cost(legs, r: Rates = Rates()) -> dict:
    """legs: iterable of (kind, side, price, qty). Returns component totals plus TOTAL."""
    out: dict = {}
    for kind, side, price, qty in legs:
        for k, v in leg_cost(kind, side, price, qty, r).items():
            out[k] = out.get(k, 0.0) + v
    out["TOTAL"] = sum(out.values())
    return out


def breakeven_points(legs, qty: int, r: Rates = Rates()) -> float:
    """Price move per unit needed to cover charges (excludes slippage and spread)."""
    return total_cost(legs, r)["TOTAL"] / qty
```

A matching test file (`test_fee_model.py`):

```python
import pytest
from fee_model import total_cost, breakeven_points

LOT = 65

def test_future_round_trip():
    legs = [("FUT", "BUY", 24000, LOT), ("FUT", "SELL", 24000, LOT)]
    t = total_cost(legs)
    assert t["stt"] == pytest.approx(312.0)
    assert t["TOTAL"] == pytest.approx(467.71, abs=0.01)
    assert breakeven_points(legs, LOT) == pytest.approx(7.20, abs=0.01)

def test_option_round_trip():
    legs = [("OPT", "SELL", 150, LOT), ("OPT", "BUY", 150, LOT)]
    t = total_cost(legs)
    assert t["stt"] == pytest.approx(9.75)
    assert t["brokerage"] == pytest.approx(5.85)        # 0.03% cap beats Rs20 on small premiums
    assert t["TOTAL"] == pytest.approx(25.02, abs=0.01)

def test_stt_only_on_sell_and_stamp_only_on_buy():
    buy = total_cost([("OPT", "BUY", 100, LOT)])
    sell = total_cost([("OPT", "SELL", 100, LOT)])
    assert buy["stt"] == 0 and buy["stamp"] > 0
    assert sell["stamp"] == 0 and sell["stt"] > 0
```

Run with `pytest`. The tests encode the hand calculations in 3.2, so if you change rates, update the expected numbers deliberately.

💡 **Insight:** This function becomes the **cost model of your backtester** (Stage D). Every backtest result in this course will be quoted *net of this function plus slippage*.

---

## 3.4 Margins: what you must deposit and why it changes

**Margin** is collateral that protects the clearing corporation (and your broker) against you not covering a loss. It isn't a fee, it's tied up capital.

### Components for futures and short options
1. **SPAN margin:** a risk-based estimate of the worst one-day loss across a set of scenarios (price up/down, volatility up/down) for your whole portfolio of one underlying. Hedged positions (e.g., spreads) attract much lower SPAN than naked ones.
2. **Exposure margin:** an additional buffer (a % of contract value) on top of SPAN.
3. **Extreme Loss Margin (ELM):** extra margin to cover tail events. It is raised on **expiry day** for contracts expiring that day under SEBI's recent measures **[VERIFY current %]**.
4. **Additional/ad-hoc margins:** exchanges and brokers may add margin around events or volatility.

### Options buying vs selling
- **Buyers** pay the **full premium upfront**, and that's their maximum loss. No further margin.
- **Sellers** (writers) must post margin because losses can be large. Naked sellers need the most; spreads need far less because the long leg caps the loss.

### Recent structural changes (rolled out in phases from late 2024 into 2025; **[VERIFY details and dates]**)
- Option premium must be **collected upfront from buyers** (no more intraday leverage on buying options).
- **Calendar-spread margin benefit removed** for positions expiring on that day.
- **Intraday monitoring of position limits** for index derivatives (not just end-of-day).
- Higher margin (ELM) on short options on expiry day.
- Higher minimum contract value (larger lots).

### Peak margin reporting
Brokers must report to the clearing corporation the **maximum margin you had at any point in the day**, and penalties apply for shortfalls. Practical consequences: you can't rely on end-of-day margin; keep a buffer; a position that looks fine on the screen at 3:00 pm may have breached margin at 9:30 am.

### Margin changes with volatility
When VIX or the underlying moves a lot, SPAN scenarios widen and margin rises, sometimes while you are in a losing position, causing a **margin call or forced square-off**. This is a hidden risk of short-option strategies at low capital.

⚠️ **Trap (₹50,000 capital):** If you use more than ~30-40% of your capital as margin, a margin rise plus a losing mark-to-market can trigger forced liquidation at the worst time. Keep ≥ 50% free.

🔧 **Try it:** Using your broker's margin calculator, compute the margin for:
1. Short 1 lot ATM Nifty straddle.
2. Short 24,000 CE + short 24,000 PE + long 24,500 CE + long 23,500 PE (iron butterfly).
3. Bull put spread: short 23,800 PE, long 23,600 PE.
Note margin for each and the **max loss** of each. Margin vs max loss tells you how the exchange views the risk.

---

## 3.5 Taxation of F&O income in India

> ⚠️ **Not tax advice.** Rules and thresholds change in each Budget. Use this section to *prepare questions for your CA*.

### 3.5.1 How F&O income is classified
- F&O gains and losses are treated as **non-speculative business income** (not capital gains). They are reported under **"Profits and Gains from Business or Profession"**.
- **Intraday equity (cash) trading** is treated as *speculative* business income; F&O is not (a common confusion).
- Because it is business income you generally file **ITR-3**.

### 3.5.2 How tax is computed
- Net business income = trading profit − allowable business expenses.
- It is added to your other income and taxed at **slab rates** (not a flat rate), so your effective tax rate depends on your total income and chosen regime (old vs new). Surcharge and 4% cess apply on top.
- **Allowable expenses (typically):** brokerage, STT, exchange charges, GST on brokerage, internet, data/software subscriptions, a share of electricity/computer costs, depreciation on equipment, interest on money borrowed for trading. Keep records. **[CA to confirm what applies to you]**

### 3.5.3 Turnover (a special calculation)
For F&O, "turnover" is **not** the traded notional. Under ICAI's guidance, it is based on the **absolute sum of profits and losses** from your closed trades (the "favourable and unfavourable differences"); how option premiums and reverse trades are treated has specific rules **[ask CA, since interpretations differ]**.

Worked example (futures, 4 closed trades): +₹10,000, −₹4,000, +₹3,000, −₹2,000.
- Net profit = ₹7,000.
- Turnover (sum of absolute values) = 10,000 + 4,000 + 3,000 + 2,000 = **₹19,000**.

Why turnover matters: it decides whether you need a **tax audit**.

### 3.5.4 Tax audit (Section 44AB), in plain terms **[VERIFY thresholds every year]**
Roughly, an audit by a CA may be required if:
- Turnover exceeds the audit limit (₹10 crore if cash transactions are very small; otherwise ₹1 crore), **or**
- You are on or opt out of the presumptive scheme and report profit below the prescribed percentage of turnover, and your income exceeds the basic exemption limit. This is the clause that catches many **loss-making or low-profit F&O traders**, even with small turnover.

Penalties for missing a required audit can be significant. **Ask your CA at the start of every financial year.**

### 3.5.5 Losses
- **Non-speculative business loss** can be set off in the same year against other income heads except salary (e.g., capital gains, other business income).
- Unabsorbed loss can be **carried forward for 8 years**, but only against *business income*, and **only if you file your return by the due date**.
- Losses must be reported even if there's no profit: filing is how you keep them.
- Speculative losses (intraday cash) are separate and carried forward only 4 years against speculative income.

### 3.5.6 Compliance calendar (indicative)
| Item | Indicative date |
|---|---|
| Advance tax (if liability > ₹10,000) | 15 Jun, 15 Sep, 15 Dec, 15 Mar (cumulative %) |
| ITR filing, non-audit | 31 July after the FY end |
| ITR filing, audit case | 31 October |
| Tax audit report | 30 September |

### 3.5.7 Documents to keep
- Broker's **Tax P&L statement** and contract notes (for every FY).
- Ledger and bank statements, brokerage/charges summary.
- Records of expenses claimed.
- AIS/26AS to match reported transactions.

### 3.5.8 Post-tax returns
Required pre-tax profit = target post-tax profit / (1 − effective tax rate). For example, if your marginal rate (slab + cess) is about 31.2%, then ₹1 lakh *after* tax needs about ₹1.45 lakh *before* tax. This is why the goal "₹1 lakh/month net of tax" is a very ambitious target on ₹50,000 of capital.

---

## 3.6 Regulation you must know

### 3.6.1 SEBI's F&O measures for retail protection (announced Oct 2024; phased through 2025; **[VERIFY each]**)
- Increased minimum contract size (larger lots).
- Upfront collection of option premium.
- One weekly expiry per exchange (benchmark index).
- Removal of calendar-spread benefit on expiry day.
- Intraday monitoring of position limits.
- Higher ELM on short options on expiry day.
- Rationalised strike-price availability.

SEBI's own studies show that the majority of individual F&O traders incur net losses, with aggregate losses in the tens of thousands of crores per year **[read the primary SEBI study PDFs]**. Regulation is deliberately making F&O harder for small traders.

### 3.6.2 SEBI's retail algo framework (issued 2025; implementation was phased and extended, so check the current status **[VERIFY]**)
Broad ideas you should expect (confirm with your broker):
- Brokers are accountable ("principal") for algos run through their APIs.
- API access tied to **registered static IP addresses**; shared/unknown IPs can be blocked.
- Orders per second above a threshold require the algo to be **registered/tagged with the exchange**; below it, simple personal API use is typically allowed without registration.
- **Algo tagging** of orders for audit trails.
- Rules for third-party algo providers and "black-box" strategies (empanelment, research analyst/ registration requirements).
- Personal-use algos are generally for **your own account** only. Selling signals or running money for others needs the right SEBI registration.

### 3.6.3 Other things to know
- **Advice and tips:** giving paid trading advice requires SEBI registration. Beware unregistered "finfluencers", Telegram tips and "guaranteed return" algos. Check registrations on SEBI's website.
- **Position limits and bans:** market-wide position limits for stocks; F&O ban periods.
- **Grievances:** broker complaints → exchange → SEBI SCORES portal.
- **Contract notes and statements:** you have the right to a daily contract note and periodic statements.

---

## 3.7 Putting it together: "is this strategy even possible?"

Take a hypothetical bot that scalps Nifty futures for an average of 4 points gross per trade:
- Breakeven from charges alone = 7.2 points (Example A).
- Add 1 point of slippage each way = 9.2 points.
- A 4-point gross edge → net expected loss of ≈ 5 points per trade, **guaranteed**, no matter how good the entries are.

And a theta-harvesting iron condor selling ₹2,600 credit:
- Charges ≈ ₹20 (Example C), slippage maybe ₹300-500, so costs are 12-20% of the maximum profit.
- It needs a win rate and win/loss ratio that survive *those* costs plus the occasional large loss.

**Rule for the rest of the course:** if a result is not reported **net of charges + slippage + tax**, it isn't a result.

---

## 3.8 Exercises

1. **Cost model (2-3 hours).** Implement your own `fee_model.py` from the rates on your broker's website. Write tests reproducing 3.2 Examples A and B (adjusted to your rates).
2. **Breakeven table (45 min).** For Nifty future, ATM Nifty option (premium 100, 150, 300), and an iron condor (credit 40 points): compute charges and breakeven in points at lot size 65, and as % of premium or credit.
3. **Margin matrix (60 min).** For the three structures in 3.4, record SPAN+exposure margin from your broker, max loss from the payoff, and the ratio. Which structures are feasible for ₹50,000?
4. **Tax memo and CA meeting (1 hour + meeting).** Write a one-page note: your tax category, ITR form, whether you expect an audit, advance-tax plan, expense records you'll keep. Book 30 minutes with a CA and reconcile.
5. **Regulatory brief (60 min).** Download the latest SEBI circular(s) on F&O measures and on retail algos. Write a half-page for each: what changed, effective date, how it affects *your* trading plan.

---

## 3.9 Self-check questions

1. Why can a strategy with a +₹3 per lot gross edge be a guaranteed loser net of costs?
2. Which charge is largest for futures and which is largest in practice for options?
3. In Example A, why isn't GST applied to STT?
4. A friend says "I don't need to file ITR-3, I only made losses." What's wrong with this?
5. What is the difference between SPAN margin and exposure margin?
6. Why might your broker square off a position you thought was safe on margin?
7. Four futures trades close with P&L: +8,000, −12,000, +5,000, −1,000. Compute net P&L and turnover (ICAI-style).
8. Your target is ₹1 lakh/month net of tax. Your marginal tax rate is 31.2%. What pre-tax monthly profit do you need?
9. Why does a cheap far-OTM option with a 2-point spread (on a 6-point premium) hurt more than the statutory charges?

### Answer key

1. Charges on a futures round trip are about ₹468 per lot (Example A), so a ₹3 edge is wiped out many times over.
2. Futures: STT (₹312 of ₹468). Options: the real cost is the **bid-ask spread and slippage**; the statutory stack is small (Example B).
3. GST is levied on the *services* (brokerage and exchange/SEBI fees). STT and stamp duty are taxes/duties, not services.
4. F&O losses are business losses and filing ITR-3 by the due date is what allows you to **carry them forward for 8 years**. Without filing, you lose that benefit. An audit may also be required in loss years.
5. SPAN is a risk-model-based margin for the portfolio's worst scenario; exposure margin is an additional percentage buffer on top of it.
6. Margin requirements can rise (volatility, expiry-day ELM, event margins) while MTM losses reduce your free cash; if the shortfall isn't funded, the broker's RMS auto-squares off positions. Peak-margin penalties can also apply.
7. Net = 8,000 − 12,000 + 5,000 − 1,000 = **₹0**. Turnover = 8,000 + 12,000 + 5,000 + 1,000 = **₹26,000**.
8. 1,00,000 / (1 − 0.312) = 1,00,000 / 0.688 ≈ **₹1,45,349 per month** pre-tax.
9. Buying at the ask and later selling at the bid loses the whole 2-point spread, which is 2/6 ≈ 33% of the premium. Even if you manage to trade at mid, you give up about 1 point (≈17%) per side. The statutory charges on a trade this small are a rounding error by comparison.

---

## 3.10 Further reading
- Your broker's *Charges* and *Margin calculator* pages.
- Zerodha Varsity: *Markets and Taxation* (tax chapters), *Futures Trading* (margin chapters).
- SEBI: study on profitability of individual traders in equity F&O (FY22 onwards); circulars on F&O measures and retail algo trading.
- Income Tax India e-filing portal: ITR-3 instructions; Section 44AB FAQs.
- ICAI Guidance Note on Tax Audit (turnover computation for F&O).
