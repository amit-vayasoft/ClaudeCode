# Module 5: Stage A Capstone and Gate (Week 5)

This week you consolidate. You produce the Capstone A report, take the Stage A exam, and clear **Gate A**. Do not skip this: gaps left in Stage A reappear later as expensive mistakes.

---

## 5.1 Plan for the week

| Day | Task | Time |
|---|---|---|
| 1 | Review: re-read your journal; list the 10 fuzziest concepts | 1.5 h |
| 2 | Fix the gaps: re-read notes, redo exercises you skipped | 2 h |
| 3 | Write the Capstone A report (sections 1-3) | 2 h |
| 4 | Write the Capstone A report (sections 4-6), CA/peer review | 2 h |
| 5 | Take the Stage A exam closed-book (timed, 90 min) and mark it | 2 h |
| 6-7 | Fix wrong answers; complete the Gate A checklist | 1-2 h |

---

## 5.2 Capstone A: "Market Structure and Cost Report"

A 5-8 page document (Markdown in your repo, `docs/capstone_a.md`) with the sections below. Write it as if for a smart friend who knows nothing about F&O. Teaching it is the best test of understanding.

### Section 1: How the market works (1 page)
- Diagram of the institutions and the life of an order.
- A paragraph on how an order can be rejected and at which layer.
- Trading hours, settlement cycle, circuit breakers, ban list.

### Section 2: Contract specification table (1 page)
Five instruments (Nifty future, Bank Nifty future, Nifty weekly option, one large-cap stock future, one mid-cap stock future). Columns: symbol, lot size, price, notional, margin, margin %, leverage, expiry, settlement type. Source and date for each number.

### Section 3: Futures mechanics (1 page)
- MTM table for a hypothetical 5-day position (your own numbers).
- Fair value and basis for the current-month future (show the formula, inputs and result) plus your 2-week basis log summary.
- A paragraph on what OI does and doesn't tell you.

### Section 4: Costs and breakeven (1-2 pages)
- Your broker's actual charges table.
- Full cost computation for: (a) 1 lot Nifty future round trip, (b) a single-leg option sell-and-buy-back, (c) a 4-leg iron condor in and out.
- Breakeven in points and as % of premium/credit for each.
- A paragraph estimating realistic slippage for each and the all-in breakeven.
- Link to your tested `fee_model.py`.

### Section 5: Margin and tax (1-2 pages)
- Margin table for three structures (naked straddle, iron butterfly, bull put spread): SPAN+exposure margin, max loss, feasibility at ₹50,000.
- Tax summary reviewed by a CA: category, ITR form, audit expectation, loss carry-forward, advance tax plan, expense records.

### Section 6: Operational risk register (1 page)
List **at least 10** things that could go wrong operationally, with detection and response for each. Seed ideas:

| # | Risk | Detect | Response |
|---|---|---|---|
| 1 | API token expired in the morning | Auth error at startup | Alert; manual login procedure |
| 2 | Order rejected (margin) | Order status = rejected | Log layer; do not retry blindly |
| 3 | Partial fill on one leg of a spread | Position mismatch check | Flatten the unintended leg |
| 4 | Lot size revised | Instrument master diff vs yesterday | Block trading until config updated |
| 5 | Stock enters F&O ban list | Daily ban list fetch | Reduce-only mode |
| 6 | Internet or laptop failure | Heartbeat | Manual square-off procedure (phone/app) |
| 7 | Wrong strike/expiry symbol | Pre-trade validation against instrument master | Reject order |
| 8 | Exchange halt / circuit | No ticks, stale data timer | Stand down; no new orders |
| 9 | Margin spike / peak-margin penalty | Margin utilisation > threshold | Reduce positions |
| 10 | Leaked API key | Unknown orders / alerts | Revoke key, audit trades |

### Capstone rubric (self-score, then have someone review)

| Criterion | 0 | 1 | 2 |
|---|---|---|---|
| Every number is sourced and dated | Few | Some | All |
| Cost calculations match broker calculator | No | Within ₹10 | Within ₹2 |
| Breakeven includes realistic slippage | Absent | Mentioned | Quantified |
| Tax section reviewed by a CA | No | Partially | Yes, with notes |
| Risk register is specific and actionable | Vague | Mixed | Concrete |
| Writing is clear to a non-expert | No | Partly | Yes |

**Pass: ≥ 10 of 12.**

---

## 5.3 Stage A exam (closed book, 90 minutes)

Use a calculator and the "illustrative rates" table in `lessons/stage-a/README.md` where cost rates are needed. Nifty lot = 65 unless stated.

**Section I: Concepts (2 marks each)**

1. Which entity becomes the counterparty to both the buyer and the seller after a trade, and what is this process called?
2. Name the three layers at which an order can be rejected and give one example of each.
3. What is the difference between volume and open interest? Give an example in which a trade of 10 contracts leaves OI unchanged.
4. State the settlement type (cash or physical) for Nifty options and for Reliance stock options. Why does it matter near expiry?
5. Which two statutory charges apply only to the sell side, and which to the buy side, in an F&O round trip? (Use the rates table.)
6. Name three recent SEBI measures aimed at F&O retail participants.
7. What is the tax classification of F&O income, and which ITR form is normally used?
8. Why is an LTP-based backtest fill unrealistic?

**Section II: Calculations (4 marks each)**

9. You buy 2 lots of Nifty futures at 24,100 and sell at 24,250. (a) Gross P&L? (b) Total charges using the rates table? (c) Net P&L?
10. Nifty future at 24,250: (a) notional of 1 lot; (b) leverage if margin is ₹1,60,000.
11. Spot 24,200, r = 7%, q = 1.2%, 20 days to expiry. Compute the fair future and basis in points.
12. A long futures position: entry 24,000 (2 lots). Daily settlement prices: 24,040, 23,980, 24,100. Compute daily MTM and total.
13. The ask side of an option shows 80.00 × 65 and 80.20 × 65. You market-buy 2 lots. Average fill and slippage in ₹ versus the best ask.
14. Bid 6.0, ask 8.0. Compute the spread, mid, and spread as % of mid.
15. Four futures trades close with P&L: +9,000, −5,000, −2,500, +1,500. Compute net P&L and F&O turnover (ICAI-style).
16. A strategy captures 5 points gross on average per Nifty futures round trip. Using charges breakeven of 7.2 points, what's the expected net per trade (before slippage), in points and ₹?

**Section III: Applied (6 marks each)**

17. You have ₹50,000. A friend proposes selling 1 lot of a naked ATM Nifty straddle. Using what you learned about margin, leverage and risk, write a short assessment (6-8 lines).
18. Explain how F&O losses are treated for tax and why filing the return on time matters even in a loss year.
19. Your algo placed four orders for an iron condor and only three filled. Describe the risk, the order in which legs should have been sent, and the system check that would catch this.
20. Describe three things you'd check in a broker's API documentation before building an algo on it, and why each matters.

### Answer key

1. The clearing corporation (NSE Clearing), via **novation**: it becomes the buyer to every seller and the seller to every buyer.
2. (a) Broker RMS: insufficient margin. (b) Exchange: price outside band / over freeze quantity / invalid lot. (c) Clearing/regulatory rule: fresh position in a banned stock (also accept: margin/peak-margin penalties).
3. Volume counts every contract traded; OI counts contracts still outstanding. Example: A (long 10) sells 10 to a new participant C (C opens, A closes) → volume +10, OI unchanged.
4. Nifty options: **cash-settled**. Reliance options: **physically settled** (delivery of shares). Near expiry, ITM stock positions create delivery obligations, margin rises, and brokers may square off; index positions simply settle in cash.
5. Sell side only: **STT**. Buy side only: **stamp duty**. (Brokerage, exchange charges, SEBI fee and GST apply to both sides.)
6. Any three of: larger minimum contract size; upfront collection of option premium; single weekly expiry per exchange; removal of calendar-spread benefit on expiry day; intraday position-limit monitoring; higher ELM on expiry day; rationalised strikes. **[VERIFY]**
7. Non-speculative business income; **ITR-3**.
8. LTP is a past trade price. You buy at the ask and sell at the bid and large orders move through the book, so you pay spread plus depth impact on every trade; LTP ignores both.
9. Qty = 130. (a) Gross = (24,250 − 24,100) × 130 = **₹19,500**. (b) Charges: brokerage 40.00 + exchange 125.71 + SEBI 6.29 + GST 30.96 + STT 630.50 + stamp 62.66 = **≈ ₹896.11**. (c) Net = 19,500 − 896.11 = **≈ ₹18,603.89**.
10. (a) 24,250 × 65 = **₹15,76,250**. (b) 15,76,250 / 1,60,000 ≈ **9.85×**.
11. T = 20/365 = 0.054795. Exponent = (0.07 − 0.012) × 0.054795 = 0.0031781. Fair = 24,200 × e^0.0031781 ≈ **24,277.0**; basis ≈ **+77 points**.
12. Qty = 130. Day 1: (24,040 − 24,000) × 130 = **+5,200**. Day 2: (23,980 − 24,040) × 130 = **−7,800**. Day 3: (24,100 − 23,980) × 130 = **+15,600**. Total = **+13,000** = (24,100 − 24,000) × 130 ✓.
13. 65 @ 80.00 + 65 @ 80.20 → total = 5,200 + 5,213 = ₹10,413; average = 10,413 / 130 = **80.10**. Slippage vs 80.00 = 0.10 × 130 = **₹13**.
14. Spread = **2.0**. Mid = **7.0**. Spread % of mid = 2/7 ≈ **28.6%**.
15. Net = 9,000 − 5,000 − 2,500 + 1,500 = **+₹3,000**. Turnover = 9,000 + 5,000 + 2,500 + 1,500 = **₹18,000**.
16. Net = 5 − 7.2 = **−2.2 points** per trade = −2.2 × 65 ≈ **−₹143** per lot per trade, before slippage. It loses every time on average.
17. A strong answer covers: naked short straddle margin is large relative to ₹50,000 (often more than the whole account), so it may not even be allowed or leaves no buffer; unlimited-risk exposure on both sides; a 1-2% Nifty move creates losses of ₹15,000-30,000 per lot, which is 30-60% of capital; margin rises with volatility, so a forced square-off at the worst moment is likely; the account fails the "1-2% risk per trade" rule; a defined-risk spread (or paper trading) is more consistent with the risk policy.
18. F&O income is non-speculative business income; losses can be set off against other income heads except salary in the same year and carried forward up to 8 years against business income, but **only if the return is filed by the due date**. Late filing forfeits the carry-forward. An audit may also be required in loss years.
19. Risk: an unintended naked short leg with large or undefined risk (e.g., short call without its protective long call). Send the **long (protective) legs first**, then the short legs, ideally via a broker spread/basket order that handles it, and verify each fill. Check: a post-order **position reconciliation** that compares actual positions to the intended structure within seconds and auto-flattens/alerts on mismatch.
20. Any three well-justified: rate limits (orders/sec; ban risk and design of throttling); websocket instrument limits and reconnect behaviour (data completeness); historical data availability (research); token expiry and login flow (daily operations); order types supported incl. multi-leg (execution design); sandbox availability (safe testing); static-IP/algo-tagging requirements (compliance); outage history (reliability).

**Scoring:** Section I: 16 marks. Section II: 32 marks. Section III: 24 marks. Total 72. **Pass: ≥ 54 (75%), with ≥ 85% on Section II.**

---

## 5.4 Gate A checklist

Tick every box honestly. If a box can't be ticked, return to the relevant module.

- [ ] I can compute any trade's all-in cost and breakeven by hand and match my broker's calculator within ₹2.
- [ ] My `fee_model.py` has passing tests.
- [ ] I can explain SPAN, exposure and extreme-loss margin, and why margin changes.
- [ ] I can compute notional, margin %, leverage, MTM cash flows, fair value and basis.
- [ ] I can explain OI vs volume and trace OI through a ledger.
- [ ] I know my tax category, ITR form and audit expectations (CA-confirmed).
- [ ] I have summarised SEBI's recent F&O measures and the retail algo framework, with sources.
- [ ] I have chosen a broker with reasons, and my read-only API script works.
- [ ] My repo has a safe secrets setup (`.env` ignored, `.env.example` committed).
- [ ] I know the manual kill procedure for my broker.
- [ ] Capstone A report scores ≥ 10/12 and has been reviewed by another person.
- [ ] Stage A exam passed (≥ 75%, ≥ 85% on calculations).

**If all boxes are ticked → proceed to Stage B (Options & Futures mastery).**
**If not → close the gaps first. Time spent here is the cheapest time in the whole course.**

---

## 5.5 What's next: Stage B preview

You will learn how option prices are formed and how P&L actually behaves day to day:
- Option payoffs, moneyness, put-call parity
- Black-Scholes and implied volatility (coded from scratch)
- The Greeks, one at a time
- Volatility: realised, implied, VIX, skew, term structure, variance risk premium
- The strategy catalogue and when each makes sense
- Adjustments, expiry-day mechanics and event risk
- Futures analytics and honest testing of popular indicators

Start a "Stage B questions" file now: write down anything from Stage A you wish you understood better (for example, "why does the OTM put cost more than the OTM call?") and bring it to Module 6.
