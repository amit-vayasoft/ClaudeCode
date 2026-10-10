# Stage A Lesson Notes: Foundations (Weeks 1-5)

Detailed, self-study notes for Stage A of the [F&O course](../../README.md).

| File | Module | Week |
|---|---|---|
| [module-01-market-structure.md](module-01-market-structure.md) | How the Indian market works | 1 |
| [module-02-derivatives-101.md](module-02-derivatives-101.md) | Futures, options contracts, leverage, MTM, fair value | 2 |
| [module-03-costs-margin-tax-regulation.md](module-03-costs-margin-tax-regulation.md) | Costs, margins, taxation, regulation | 3 |
| [module-04-broker-and-setup.md](module-04-broker-and-setup.md) | Broker choice, option chain, Python/API setup | 4 |
| [module-05-capstone-and-gate.md](module-05-capstone-and-gate.md) | Capstone A, review checklist, Gate A | 5 |
| [glossary.md](glossary.md) | Every term used in Stage A | reference |

## How to study each module

1. **Read the notes once** without stopping (30-45 min).
2. **Do the "Try it" boxes** as you go. They are small, and they're where learning happens.
3. **Do the end-of-module exercises** in your own files (`notebooks/`, `journal/`).
4. **Answer the self-check questions *before* looking at the answer key.** Write your answers down.
5. **Write 5 lines in your journal:** what clicked, what's still fuzzy, what you'd teach someone else.
6. **Pair with outside material.** Each module lists free reading (Zerodha Varsity chapters, NSE pages). The notes here give the structure and the numbers; those give a second voice.

## Conventions used in these notes

- **₹** amounts and index levels are **illustrative** unless stated. Nifty is assumed at 24,000 and a Nifty lot at 65 units. Lot sizes, margins and tax rates change; every such number is tagged **[VERIFY]**. Before you rely on one, check the latest NSE circular, your broker's charges page, or the Income Tax portal.
- **Rates used in cost examples** (all **[VERIFY]**, plug in current values):

| Item | Rate used in examples |
|---|---|
| Brokerage | lower of ₹20 per executed order or 0.03% of turnover (many discount brokers) |
| Exchange transaction charge, futures | 0.002% of contract value |
| Exchange transaction charge, options | 0.035% of premium value |
| SEBI turnover fee | ₹10 per crore (0.0001%) |
| STT, futures | 0.02% on the **sell** side |
| STT, options | 0.1% of premium on the **sell** side |
| Stamp duty | 0.002% (futures), 0.003% (options), **buy** side |
| GST | 18% on (brokerage + exchange charges + SEBI fee) |

  STT and other charges have been **revised upward more than once recently**, so check the current table first.
- 🔧 **Try it** = a small hands-on task. 🧠 **Check** = a question you should be able to answer. ⚠️ **Trap** = a common beginner mistake. 💡 **Insight** = something that will matter later when you build algos.

## Time budget

| Week | Hours | Focus |
|---|---|---|
| 1 | 8-10 | Module 1 + exercises |
| 2 | 8-10 | Module 2 + spreadsheet |
| 3 | 10-12 | Module 3 (densest week) + CA meeting |
| 4 | 8-10 | Module 4 + environment setup |
| 5 | 6-8 | Capstone, review, Gate A |
