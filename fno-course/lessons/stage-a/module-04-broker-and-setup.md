# Module 4: Broker, Platform and Workflow Setup (Week 4)

This module is practical. By the end you will have chosen a broker, learned to read an option chain the way a professional does, set up a clean Python project with safe secret handling, and written a **read-only** script that talks to your broker's API.

**By the end of this module you can:**
- Choose a broker using criteria that matter to an algo trader.
- Read every column of an option chain and say what each tells you (and what it doesn't).
- Tell a payoff-at-expiry curve from a "today" (T+0) curve.
- Create a reproducible Python environment and a repo that cannot leak secrets.
- Authenticate to a broker API and read margins and live quotes without placing an order.

**Free companion reading:** your broker's API documentation (Kite Connect, Upstox, Angel One SmartAPI, Dhan, Fyers, etc.); Zerodha Varsity → *Option Chain* chapter in *Options Theory*.

---

## 4.1 Choosing a broker

You will use this broker for data, execution and margin for years, and it's the single point of failure on expiry day. Choose on facts, not on ads.

### Criteria for an algo trader

| Criterion | Why it matters | How to check |
|---|---|---|
| **API availability & cost** | Some brokers charge a monthly fee for the full API (live data + history); some are free | Pricing page; community forums |
| **API stability on expiry/volatile days** | An API that times out in a crash is worse than none | Developer forums, GitHub issues, past outage reports |
| **Rate limits** | Caps on orders/sec and requests/sec; SEBI's framework adds thresholds **[VERIFY]** | API docs |
| **Websocket streaming** | Ticks for many instruments | Max instruments per connection, reconnect behaviour |
| **Historical data** | For research. Intraday depth, options history, how far back | Docs; test a download |
| **Order types & multi-leg support** | Basket orders, spread margin benefit, GTT | Docs; margin calculator |
| **Margin & charges** | Brokerage structure, margin calculator API, pledge facilities | Charges page |
| **Sandbox / paper trading** | Safe testing | Docs (many brokers have none, so you'll write your own `PaperBroker`) |
| **Compliance posture** | Static-IP setup, algo-tagging, SEBI framework readiness **[VERIFY]** | Support/docs |
| **Support & community** | When things break at 9:14 am | Forums, quality of responses |
| **Reliability of the retail app** | You'll need the app when the API fails (manual kill) | Reviews; try it |

### Common choices (verify current offerings yourself)
| Broker | API note (illustrative, **[VERIFY]**) |
|---|---|
| **Zerodha (Kite Connect)** | Mature, well-documented, large community; paid plan for data; daily token login |
| **Upstox** | Free/low-cost API tier; v2 APIs with SDKs |
| **Angel One (SmartAPI)** | Free API; TOTP-based login |
| **Dhan** | Developer-friendly API; data APIs |
| **Fyers** | API v3 with websockets; free tier |
| **Shoonya (Finvasia)** | Free API; a smaller ecosystem |

🧠 **Decision exercise:** Pick two candidates. Read each one's API docs for 30 minutes. Write down: how login works, how many requests per second, how to get historical option data, what happens to your token overnight, whether there's a paper/sandbox mode. Choose, and note why.

⚠️ **Trap:** Don't choose on brokerage alone. A ₹10 saving per trade is irrelevant next to a failed stop-loss order on expiry day. Don't use unregistered or foreign-only "algo platforms" that ask for your broker password or sell "guaranteed" bots.

---

## 4.2 Anatomy of an option chain

An **option chain** lists all strikes for one underlying and one expiry, calls on the left, puts on the right, strikes in the middle.

```
               CALLS                         |  STRIKE |                PUTS
 OI    ΔOI   Vol   IV    LTP   Bid   Ask     |         |  Bid   Ask   LTP    IV   Vol   ΔOI    OI
 …
 8.2L  +1.1L 12L  13.8  215   214.9 215.4    | 24000   | 148.0 148.5 148.2  13.2  18L  +2.0L  9.5L
 …
```

| Column | Meaning | Use | Limits |
|---|---|---|---|
| **Strike** | Agreed price | The ATM strike is closest to spot | |
| **LTP** | Last traded price | Orientation | Can be stale for illiquid strikes |
| **Bid / Ask (and qty)** | Best prices and depth | **The real prices you trade at** | Only top of book on free views |
| **IV** | Implied volatility of that option | Compare strikes (skew) and expiries (term structure) | Calculation conventions differ between platforms |
| **Volume** | Contracts traded today | Liquidity | Includes both opening and closing trades |
| **OI** | Open interest | Where positions are concentrated | Not directional (Module 2) |
| **ΔOI (change in OI)** | New positions opened vs closed today | Shows where activity is building | Sellers and buyers both counted |
| **Greeks (Δ Γ Θ V)** | Sensitivities | Position risk | Model-based; depends on IV input |
| **PCR** | Put OI ÷ Call OI | Sentiment heuristic | Weak predictive value (tested later) |

### Reading the chain: five habits
1. **Find ATM, then look at the straddle price** (ATM call + ATM put). It approximates the market's expected move to expiry (≈ straddle ÷ spot).
2. **Check bid-ask spreads** before anything else. If spread > ~2-3% of price, think hard.
3. **Look at OI clusters** as *potential* reference levels (support/resistance folklore), but treat them as hypotheses.
4. **Compare IV across strikes**: puts below spot usually have higher IV (skew). That's normal in equity indices.
5. **Check the days to expiry and event calendar** (RBI, budget, results) before reading IV as "cheap" or "expensive".

### Example: the market's "expected move"
Spot 24,000. ATM call = 150, ATM put = 148 → straddle = 298. Expected move ≈ 298/24,000 ≈ **1.24%** (≈ ±298 points) by expiry. If a strategy needs more than that, the market doesn't see it as likely. (The straddle price is roughly 0.8 × σ√T × spot.)

🔧 **Try it:** Take a screenshot of today's Nifty option chain. Mark: spot, ATM strike, straddle price, expected move %, the two strikes with highest call OI and put OI, the spread at ATM and at ±500 points. Write three sentences about what you can and can't conclude.

---

## 4.3 Tools for analysing positions

| Tool type | Examples | What for |
|---|---|---|
| **Option chain & analytics** | NSE website, broker apps, Sensibull, Opstra, Streak | Chain, OI, IV, strategy builder, payoff curves |
| **Charting** | TradingView, broker charts | Price/volatility charts |
| **Spreadsheets** | Excel/Google Sheets | Quick calculations, trade log |
| **Your own code** | Python + Jupyter | Everything repeatable |

### Two payoff curves: don't confuse them
- **Payoff at expiry:** what the position is worth at the *end*. Has sharp corners (e.g., a spread's hockey-stick). Useful for max profit/loss/breakeven.
- **T+0 (today) curve:** estimated value *now* using a pricing model (Black-Scholes) with current IV and time remaining. Smooth. Shows how P&L changes with price **before expiry** and with IV changes.

Most losses come from the gap between these two: you're comfortable with the expiry picture, but time, IV and gamma make your P&L today look very different.

---

## 4.4 Set up your development environment

### 4.4.1 Tools
- **Python 3.11+** (check with `python3 --version`)
- **Git** and a GitHub account
- **VS Code** (or PyCharm) with the Python extension
- **Jupyter** for notebooks

### 4.4.2 Project skeleton
```
fno-course/
  .gitignore
  .env.example         <- names of variables, no secrets
  pyproject.toml       <- dependencies
  src/fno/             <- your package: config.py, brokers/, pricing/, costs/ ...
  tests/
  notebooks/
  data/                <- ignored by git
  journal/
```

### 4.4.3 Create the environment
```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install pandas numpy scipy matplotlib jupyter pytest python-dotenv ruff
# broker SDK of your choice, e.g.:
pip install kiteconnect            # Zerodha
pip freeze > requirements.txt      # or use pyproject.toml
```

### 4.4.4 `.gitignore` (minimum)
```
.venv/
__pycache__/
.ipynb_checkpoints/
.env
*.pem
*.key
data/
*.log
```

### 4.4.5 Secrets: the rules
1. **Never commit API keys, secrets, access tokens, TOTP seeds or passwords.** One leaked key can place orders on your account.
2. Keep them in a local `.env` (git-ignored) or an OS/cloud secret manager. Commit only `.env.example`:
   ```
   BROKER_API_KEY=
   BROKER_API_SECRET=
   BROKER_ACCESS_TOKEN=
   ```
3. Use **separate keys** for research (read-only if the broker supports it) and for trading.
4. Turn on **2FA** on the broker and email accounts.
5. If you ever commit a secret by mistake: **revoke/rotate the key immediately**. Deleting the commit isn't enough, because history and caches persist.
6. Add a secret scanner to catch mistakes, e.g., `pip install pre-commit detect-secrets` and a pre-commit hook.

### 4.4.6 Config loader (tested example)
`src/fno/config.py`:
```python
import os
from dataclasses import dataclass

from dotenv import load_dotenv


@dataclass(frozen=True)
class Settings:
    api_key: str
    api_secret: str
    access_token: str | None


def load_settings() -> Settings:
    """Read credentials from environment / .env. Never hard-code secrets."""
    load_dotenv()
    missing = [k for k in ("BROKER_API_KEY", "BROKER_API_SECRET") if not os.getenv(k)]
    if missing:
        raise RuntimeError(f"Missing environment variables: {', '.join(missing)}")
    return Settings(
        api_key=os.environ["BROKER_API_KEY"],
        api_secret=os.environ["BROKER_API_SECRET"],
        access_token=os.getenv("BROKER_ACCESS_TOKEN"),
    )
```
A test for it:
```python
import pytest
from fno.config import load_settings

def test_missing_env_raises(monkeypatch):
    monkeypatch.delenv("BROKER_API_KEY", raising=False)
    monkeypatch.delenv("BROKER_API_SECRET", raising=False)
    monkeypatch.setattr("fno.config.load_dotenv", lambda: None)
    with pytest.raises(RuntimeError):
        load_settings()

def test_settings_loaded(monkeypatch):
    monkeypatch.setenv("BROKER_API_KEY", "k")
    monkeypatch.setenv("BROKER_API_SECRET", "s")
    monkeypatch.setattr("fno.config.load_dotenv", lambda: None)
    s = load_settings()
    assert (s.api_key, s.api_secret, s.access_token) == ("k", "s", None)
```

---

## 4.5 Your first broker API script (read-only!)

**Rule for this week: no order-placing functions in your code at all.** You are only authenticating, reading margins and quotes, and downloading the instrument list.

### 4.5.1 The typical login flow (daily)
Most Indian broker APIs use an OAuth-style flow where the **access token expires every day** (often early morning):
```
1. You open the broker's login URL (with your api_key) in a browser.
2. You log in with user id, password and 2FA/TOTP.
3. The broker redirects to your registered redirect URL with a short-lived request_token.
4. Your script exchanges request_token + api_secret for an access_token.
5. You use access_token for all API calls that day.
```
Some brokers allow automated TOTP login. Check their terms. Automating 2FA may breach policy and weakens security. A common safe pattern: log in manually (or via an approved flow) each morning and paste the token in `.env`.

### 4.5.2 Illustrative script (Zerodha Kite Connect SDK; adapt for your broker)
> The method names below follow the Kite Connect Python client at the time of writing **[VERIFY against current docs]**. I have not run this against a live account.

```python
# scripts/hello_broker.py: READ-ONLY. Do not add order placement here.
from kiteconnect import KiteConnect
from fno.config import load_settings


def main() -> None:
    s = load_settings()
    kite = KiteConnect(api_key=s.api_key)

    if s.access_token is None:
        print("Open this URL, log in, and copy the request_token from the redirect:")
        print(kite.login_url())
        request_token = input("request_token: ").strip()
        session = kite.generate_session(request_token, api_secret=s.api_secret)
        print("Put this in .env as BROKER_ACCESS_TOKEN (valid today only):")
        print(session["access_token"])
        return

    kite.set_access_token(s.access_token)

    margins = kite.margins()
    print("Available cash:", margins["equity"]["available"]["live_balance"])

    quote = kite.quote(["NSE:NIFTY 50"])
    print("Nifty last price:", quote["NSE:NIFTY 50"]["last_price"])


if __name__ == "__main__":
    main()
```

### 4.5.3 The instrument master
Brokers publish a daily file (CSV/API call) listing every tradeable instrument: **token, trading symbol, expiry, strike, instrument type (CE/PE/FUT), lot size, tick size, exchange.** You'll use it to:
- map symbols to tokens (for websockets and orders),
- find lot sizes, **and keep a dated history of them**,
- list all strikes/expiries for an underlying.

With Kite: `kite.instruments("NFO")` returns the F&O list **[VERIFY]**. Save a dated copy each day to `data/instruments/YYYY-MM-DD.csv`. This builds the lot-size history you'll need for honest backtests (Module 2).

### 4.5.4 Logging
Use Python's `logging` from day one: timestamped lines to a file, with no secrets in the log text. In Stage E you'll use logs to reconstruct what your system did and why.

```python
import logging
logging.basicConfig(
    filename="data/hello_broker.log",
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)
```

---

## 4.6 Habits to build now

1. **Version everything**, with small commits and clear messages.
2. **Write the test before trusting the number**, especially for costs and P&L.
3. **Never run untested code against real money.**
4. **Keep a daily log:** what you did, what broke, what you learned (`journal/YYYY-MM-DD.md`).
5. **Back up** your repo and data.
6. **Have a manual fallback:** know how to square off positions through the broker's app/website/phone if your code or laptop dies.

---

## 4.7 Exercises

1. **Broker decision memo (1-2 hours).** One page: candidates, criteria table filled in, final choice and reasons.
2. **Option-chain annotation (1 hour).** As in the "Try it" in 4.2. Add a table: strikes ±500 points from ATM with bid, ask, spread, spread %.
3. **Environment setup (1-2 hours).** Create the repo skeleton, virtual environment, `.gitignore`, `.env.example`, config loader and tests (`pytest` passes). Confirm `git status` shows `.env` as untracked-and-ignored.
4. **Hello-broker script (1-2 hours).** Get the read-only script running: margins and Nifty quote printed. If your broker lacks a free API tier, do the exercise with the broker's free tier/sandbox or simulate with a stub that returns fixed values.
5. **Instrument snapshot (45 min).** Download the F&O instrument list, save it dated, and produce a table of Nifty lot size, available expiries and strike step. Add a one-line comment on how you'd store lot-size changes over time.
6. **Manual kill drill (20 min).** Find, in your broker's app/website, exactly how to: view all open positions, square off everything, and cancel all pending orders. Write the steps down. You'll need them one day under stress.

---

## 4.8 Self-check questions

1. Why is an API that is down on expiry day more costly than an extra ₹10 brokerage per trade?
2. What's the difference between volume and OI in an option chain, and which one tells you about liquidity?
3. The ATM straddle costs ₹298 with spot at 24,000. What move does the market roughly expect by expiry, in % terms?
4. What's the difference between the payoff-at-expiry curve and the T+0 curve? When does it matter most?
5. List three rules for handling API secrets.
6. Why should the first API script be read-only?
7. Why store dated copies of the instrument master?
8. You accidentally pushed your `.env` to a public repo and deleted it in the next commit. Are you safe? What do you do?

### Answer key

1. A failed or delayed stop-loss during a fast move on expiry day can cost thousands of rupees in one trade, which dwarfs small per-trade savings.
2. Volume = contracts traded today (both opening and closing). OI = contracts still outstanding. Liquidity (ease of entering/exiting) is best judged by **bid-ask spread and depth**, then volume; OI shows positioning, not tradability.
3. 298 / 24,000 ≈ **1.24%**, i.e., about ±300 points by expiry.
4. At-expiry shows terminal P&L; T+0 shows current value given model, IV and time left. It matters for any position held before expiry (most of them), where IV and time can swing P&L by far more than the expiry picture suggests, especially near expiry (high gamma).
5. Any three: never commit keys; use `.env`/secret manager; separate research and trading keys; 2FA on; rotate on leak; secret scanner in pre-commit; don't share keys with third parties.
6. A bug in a read-only script can't lose money. You verify authentication, data and plumbing safely before adding the dangerous part.
7. Lot sizes, strikes available, tokens and symbols change over time. Historical simulation needs the values valid on each date, and brokers' instrument files only show the current state.
8. No. Anyone who cloned or scraped the repo in the interval has the key, and Git history retains it. **Revoke/rotate the API key and secret immediately**, check your account for unauthorised activity, and then clean the history (and add a scanner).

---

## 4.9 Further reading
- Your broker's API docs and changelog; their developer forum.
- NSE: *Option chain* page and *Live equity derivatives* data.
- Python: official tutorial; `pandas` 10-minute intro; `pytest` getting-started.
- Git: *Pro Git* book chapters 1-3 (free online).
