# FRED_Data_Analysis

# YipitData AI Engineer Take-Home Exercise

## What you're doing

We hand you a real customer question and a small set of files. You spend **4 to 6 hours** on it. You send back **three things**:

* a notebook
* a one-page memo
* a prompt log

We read what you sent. Then we talk it through with you on a Zoom call.

The question is closer to what we actually do here than it is to a textbook problem. There is no single right answer.

---

# The customer's question

Read this brief carefully. It is the actual question.

> “We track the monthly retail-sales series from FRED for our subsector. We are thinking about using it as a leading indicator of quarterly revenue for Walmart.
>
> Does it predict Walmart’s revenue better than a naive baseline?
>
> If yes, by how much, and what should we worry about?
>
> If no, what evidence would change our minds?”

A **leading indicator** is a signal that moves before the thing you care about.

A **naive baseline** is the simplest possible guess.

We’ll show you a few examples of each below.

---

# What you send back

Three files, packed into one zip:

## 1. Notebook

A Python notebook called:

* `analysis.ipynb`

Your full analysis lives here.

Requirements:

* code should run end to end without errors
* add comments where helpful for human readers

---

## 2. Memo

A one-page memo called:

* `memo.md`
* or `memo.pdf`

Frame the question, give the answer, and list the things that worry you.

Imagine the reader is a portfolio manager who took stats in college but has not done much of it since.

---

## 3. Prompt log

A file called:

* `prompts.md`

Include:

* prompts you sent to your LLM assistant
* or exported prompt history / compressed folder

Also add a short note (**under 200 words**) about:

* what the assistant got right
* where you had to push back
* how you checked its output

---

# How long this should take

Plan for **4 to 6 hours**.

Hard cap at **6 hours**.

If you go over, tell us.

We read your work for judgment, not for stamina.

---

# Tools you can use

Python is required.

Beyond that, work the way you actually work.

You may use any LLM coding assistant:

* Claude Code
* Cursor
* GitHub Copilot
* ChatGPT
* etc.

We expect that you will.

We are testing your judgment about LLMs, not your willingness to do without them.

The prompt log is the artifact that shows us how you used the tool.

---

# The data

Pull and rename two CSV files using the directions below.

## Required files

### `retail_sales_fred.csv`

Monthly U.S. retail-sales index from FRED:

* series: `RSXFS`
* from 2010 through last month

### `walmart_revenue.csv`

Quarterly Walmart revenue from SEC filings:

* ticker: `WMT`
* from 2010 through most recent reported quarter

You can use these files as-is. That is the easy path.

If you want fresher data or a different retailer, optional API instructions are included below.

---

# Free API Key #1: FRED (Federal Reserve Economic Data)

FRED is the public data archive run by the St. Louis Fed.

Signup takes about a minute.

## Steps

1. Go to:
   `https://fred.stlouisfed.org/docs/api/api_key.html`
2. Register for an account
3. Confirm your email
4. Request an API key
5. Copy the key

---

## Example FRED pull

```python
"""monthly U.S. retail-sales index (RSXFS), Jan 2010 through last released month."""

from pathlib import Path

import pandas as pd
import requests

FRED_KEY = "paste_your_key_here"

url = "https://api.stlouisfed.org/fred/series/observations"

params = {
    "series_id": "RSXFS",
    "api_key": FRED_KEY,
    "file_type": "json",
    "observation_start": "2010-01-01",
}

r = requests.get(url, params=params, timeout=30)
r.raise_for_status()

df = pd.DataFrame(r.json()["observations"])[["date", "value"]]

df["date"] = pd.to_datetime(df["date"])
df["value"] = pd.to_numeric(df["value"], errors="coerce")

df = (
    df.dropna()
      .sort_values("date")
      .reset_index(drop=True)
)

Path("data").mkdir(exist_ok=True)

df.to_csv("data/retail_sales_fred.csv", index=False)
```

Wrappers like:

* `fredapi`
* `pandas-datareader`

are also fine.

---

# No key needed: yfinance (Yahoo Finance)

The `yfinance` package can pull quarterly financials directly from Yahoo.

```python
import yfinance as yf

wmt = yf.Ticker("WMT")

quarterly = wmt.quarterly_financials

revenue = quarterly.loc["Total Revenue"]
```

### Important

`yfinance` scrapes public webpages and occasionally breaks when Yahoo changes layouts.

If it fails:

```bash
pip install -U yfinance
```

---

# No key needed: SEC EDGAR

EDGAR is the SEC filings archive.

No API key required.

You **must** send a `User-Agent` header.

---

## Example SEC revenue extraction

```python
"""
quarterly Walmart revenue (fiscal Q1-Q4),
Jan 2010 through most recent reported quarter.

Important details:
- Walmart switches accounting concepts after fiscal 2018
- Q4 must be derived from FY totals
"""

from pathlib import Path

import pandas as pd
import requests

USER_AGENT = "Your Name your.email@example.com"

frames = []

for concept in (
    "Revenues",
    "RevenueFromContractWithCustomerExcludingAssessedTax"
):

    url = (
        "https://data.sec.gov/api/xbrl/companyconcept/"
        f"CIK0000104169/us-gaap/{concept}.json"
    )

    r = requests.get(
        url,
        headers={"User-Agent": USER_AGENT},
        timeout=30
    )

    if r.status_code == 404:
        continue

    r.raise_for_status()

    units = r.json().get("units", {}).get("USD", [])

    if units:
        frames.append(pd.DataFrame(units))

raw = pd.concat(frames, ignore_index=True)

raw["start"] = pd.to_datetime(raw["start"])
raw["end"] = pd.to_datetime(raw["end"])
raw["filed"] = pd.to_datetime(raw["filed"])

raw["days"] = (raw["end"] - raw["start"]).dt.days

raw = raw[
    raw["days"].between(80, 100)
    | raw["days"].between(355, 375)
].copy()

raw["kind"] = raw["days"].apply(
    lambda d: "Q" if d <= 100 else "FY"
)

raw = (
    raw.sort_values("filed")
       .drop_duplicates(
           ["start", "end", "kind"],
           keep="last"
       )
)

q_facts = (
    raw[raw["kind"] == "Q"][["end", "val"]]
    .rename(columns={"end": "date", "val": "value"})
)

q4_rows = []

for _, fy in raw[raw["kind"] == "FY"].iterrows():

    in_fy = q_facts[
        (q_facts["date"] > fy["start"])
        & (q_facts["date"] < fy["end"])
    ]

    if len(in_fy) == 3:

        q4_rows.append({
            "date": fy["end"],
            "value": (
                float(fy["val"])
                - float(in_fy["value"].sum())
            )
        })

revenue = pd.concat(
    [q_facts, pd.DataFrame(q4_rows)],
    ignore_index=True
)

revenue["value"] = revenue["value"].astype(float)

revenue = (
    revenue.drop_duplicates("date")
           .sort_values("date")
           .query("date >= '2010-01-01'")
           .reset_index(drop=True)
)

Path("data").mkdir(exist_ok=True)

revenue.to_csv("data/walmart_revenue.csv", index=False)
```

Walmart CIK:

```text
0000104169
```

Company lookup:
`https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany`

---

# Optional APIs

You do not need these.

## Alpha Vantage

* Free key:
  `https://www.alphavantage.co/support/#api-key`
* Limits:

  * 5 calls/minute
  * 500/day

---

## Financial Modeling Prep (FMP)

* Free key:
  `https://site.financialmodelingprep.com/developer/docs`
* Limits:

  * 250/day

Both expose finance data through REST APIs.

---

# How to think about the work

Here is one possible starting point.

It is not the only approach.

It is also not the best approach.

---

# A simple, partly wrong starting approach

```python
import pandas as pd
import statsmodels.api as sm

retail = pd.read_csv(
    "retail_sales_fred.csv",
    parse_dates=["date"]
)

revenue = pd.read_csv(
    "walmart_revenue.csv",
    parse_dates=["date"]
)

# resample retail sales monthly -> quarterly
retail_q = (
    retail.set_index("date")
          .resample("QE")["value"]
          .sum()
          .reset_index()
)

# year-over-year growth
retail_q["yoy"] = retail_q["value"].pct_change(4)
revenue["yoy"] = revenue["value"].pct_change(4)

merged = pd.merge(
    retail_q,
    revenue,
    on="date",
    suffixes=("_retail", "_rev")
)

X = sm.add_constant(merged["yoy_retail"])
y = merged["yoy_rev"]

model = sm.OLS(y, X, missing="drop").fit()

print(model.summary())
```

This gives you a first pass.

It is also wrong in at least four ways.

---

# Why the simple approach is wrong

## 1. No baseline

“How well does X predict Y?” is meaningless without something simple to compare against.

You need a naive baseline, for example:

> “Next quarter’s revenue equals last year’s same quarter plus average growth.”

An R² of 0.7 sounds good until a seasonal-naive model gets 0.85.

---

## 2. In-sample evaluation

Fitting on all historical data and reporting R² only tells you how well the model explains the past.

It says nothing about future prediction quality.

You need an out-of-sample setup:

* train through year X
* predict X+1
* roll forward
* repeat

---

## 3. Look-ahead bias

Walmart Q3 revenue is reported during Q4.

If your model assumes Q3 revenue was known at quarter-end, you leaked future information.

Be explicit about what data would actually have been available at prediction time.

---

## 4. No causal story

Even if the signal predicts well:

* why does it work?
* is it stable?
* did the relationship break during COVID?
* is it causal or just correlated?

---

# What “good” looks like

A solid submission will likely:

* choose a real baseline
* beat it (or honestly fail to)
* use proper time-series validation
* clearly state caveats
* use one or two clean figures
* run end to end without errors

---

# What “great” looks like

A great submission might also:

* identify when the signal works vs fails
* catch subtle LLM mistakes
* make a clear falsifiable claim

Example:

> “The signal beats the seasonal-naive baseline by X% on out-of-sample MAPE, but only outside recession periods.”

---

# What happens after submission

If your work clears the bar, you will do a 60-minute Zoom interview.

## First 30 minutes

You walk through your analysis.

## Second 30 minutes

We challenge one of your decisions.

We are evaluating:

1. Technical reasoning
2. Ability to update calmly under disagreement

---

# What we do NOT care about

* magazine-quality plots
* trying every model ever invented
* hiding LLM usage
* exceeding the time cap to perfect things

Clean and thoughtful beats overengineered.

---

# How to submit

Send one zip file:

```text
firstname_lastname_yipitdata.zip
```

Structure:

```text
firstname_lastname_yipitdata/
├── analysis.ipynb
├── memo.md (or memo.pdf)
├── prompts.md
└── data/
    ├── retail_sales_fred.csv
    └── walmart_revenue.csv
```

If you used additional Python packages beyond:

* pandas
* numpy
* matplotlib
* statsmodels
* scikit-learn

include:

```text
requirements.txt
```

---

# Questions

If anything is unclear, email Recruiting.

We would rather answer one question than read ten guesses.

---

# Good luck

Have fun with it.
