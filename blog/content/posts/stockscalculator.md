---
title: "Getting Started with BitSwan Jupyter Automations: Creating Your First Automation"
date: 2025-03-25T10:36:05+01:00
author: "dolezal"
description: "Learn how to build your first automation using BitSwan Jupyter with a WebForm source. This step-by-step tutorial covers the basics of form creation, event handling, and pipeline execution."
tags: ["Automations","Tutorial", "BitSwan","Jupyter"]
categories: ["BitSwan Automations"]
draft: false
---

This post introduces how to use **BitSwan Jupyter Automations** to create your first automation using a **WebForm source**. You’ll learn how to set up a simple automation pipeline, define a web form, and handle the data submitted through it.

We will follow an automation that is present in our examples: [StocksCalculator](https://github.com/bitswan-space/BitSwan/tree/feat/webform-blogpost-example/examples/StocksCalculator). Feel free to experiment yourself, but this following tutorial will go to detail how each component works.

This tutorial will give you an example of an automation where you can select a date and the pipeline will display a graph of the several stocks and their performance at that day!

The result automation will look like this one: ![automation](https://jachymdolezal-stockscalculator.jachymdolezal.bswn.io/)

## Jupyter Automation

If you have a BitSwan workspace with the **VS Code editor** and the **BitSwan extension** installed, you’re ready to get started. Below is an example automation script. We’ll go through it cell by cell and explain each part of the pipeline.

### 1. Import Required Modules
```python
from bspump.jupyter import *
from bspump.http.web.server import *
import requests
import pandas as pd
from datetime import datetime, timedelta, date
import time
import io

STOCKS = ["AAPL", "MSFT", "GOOGL", "AMZN", "TSLA", "NVDA", "META", "JPM", "NFLX", "AMD"]

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
}
```

The first cell as usual imports the necessary modules.
- `bspump.jupyter` module is used to run and handle the automation in the jupyter notebook.
- The `bspump.http.web.server` module is used to create a web server that will serve the web form.

The rest of libraries are the usual libraries you might know using python.

- `pandas` is for data processing
- `requests` for sending requests via http/https
- `datetime` working with dates etc.

We also define constants we will use later in the example

### 2. Define an Initial Event (Optional)
```python
event = {
    "form" : {"textfile": "text"}
}
```
This optional cell defines an `event` dictionary that is an optional object that can be used during development, but has no effect on the pipeline during the production.

### 3. Define functions for this particular example

These are just normal python functions we will use in this case
```python
def fetch_yahoo_price(ticker, start_date, end_date):
    start_ts = int(time.mktime(start_date.timetuple()))
    end_ts = int(time.mktime(end_date.timetuple()))

    url = f"https://query1.finance.yahoo.com/v8/finance/chart/{ticker}"
    params = {
        "period1": start_ts,
        "period2": end_ts,
        "interval": "1d"
    }

    try:
        resp = requests.get(url, params=params, headers=HEADERS)
        data = resp.json()

        result = data["chart"]["result"][0]
        timestamps = result["timestamp"]
        closes = result["indicators"]["quote"][0]["close"]

        dates = [datetime.fromtimestamp(ts).strftime('%Y-%m-%d') for ts in timestamps]
        return pd.Series(closes, index=dates)
    except Exception as e:
        print(f"Error for {ticker}: {e}")
        return None


def get_price_changes(tickers, date_str):
    date_obj = datetime.strptime(date_str, "%Y-%m-%d")
    day_before = date_obj - timedelta(days=1)
    day_after = date_obj + timedelta(days=1)

    changes = {}
    for ticker in tickers:
        series = fetch_yahoo_price(ticker, day_before, day_after)
        print(series)
        if series is not None and date_str in series and day_before.strftime("%Y-%m-%d") in series:
            prev = series[day_before.strftime("%Y-%m-%d")]
            curr = series[date_str]
            if prev and curr:
                change = ((curr - prev) / prev) * 100
                changes[ticker] = change
    return changes

def plot_with_pandas(changes, date_str):
    df = pd.DataFrame(list(changes.items()), columns=["Ticker", "Change (%)"])
    df.sort_values("Change (%)", ascending=False, inplace=True)

    ax = df.plot.bar(
        x="Ticker",
        y="Change (%)",
        title=f"Stock Change (%) on {date_str}",
        xlabel="Ticker",
        ylabel="Change (%)",
        grid=True,
        figsize=(10, 5)
    )

    buf = io.BytesIO()
    ax.figure.savefig(buf, format="jpeg")
    return buf
    print(f"✅ Chart saved as 'stock_changes_{date_str}.jpeg'")
```

Feel free to copy them.

### 4. Define the Automation Pipeline
```python
auto_pipeline(
    source=lambda app, pipeline: WebFormSource(app, pipeline, route="/",
    form_intro="""<p> Select date range and click submit to see a visualization""",
    fields=[
        IntField("Day",default=20),
        IntField("Month",default=2),
        IntField("Year",default=2025),
    ]),
    sink=lambda app, pipeline: WebSink(app, pipeline)
)
```
This is the **core of the automation**. We use `auto_pipeline` to define our pipeline with minimal boilerplate:

- Source: `WebFormSource` renders a form at the root path (`/`) with a set of fields.
- Sink: `WebSink` handles the final processed output of the automation. and displays it on the web page. In this case it will render an jpeg of a graph.

The form can include:
- Text fields (some with defaults or hidden)
- An integer input
- A dropdown (choice field)
- A checkbox

In this case we are using three fields for day, month and year.

You can mix and match different sources and sinks to create your desired automation.

TODO: [Add a link to the documentation with a full list of available Sources and Sinks.]

### 5. Handle Form Submission

```python
form = event["form"]
parsed_date = date(form["Year"], form["Month"], form["Day"]).strftime("%Y-%m-%d")
try:
    changes = get_price_changes(STOCKS, parsed_date)
    if not changes:
        print("⚠️ No data found. Market may have been closed.")
        return
    top_stock = max(changes, key=changes.get)
    print(f"📈 Most increasing stock on {parsed_date}: {top_stock} ({changes[top_stock]:.2f}%)")
    img_data = plot_with_pandas(changes, parsed_date)
    event["response"] = img_data.getvalue()
    event["content_type"] = "image/jpeg"

except Exception as e:
    print(f"❌ Error: {e}")
```
Cell below the `automation_pipeline` definition are executed when the pipeline is triggered.

In this case:
- Once the form is sumitted, `WebFormSource` creates an event.
- The `event` object includec the submitted `form` data.
- We consturct a new `response` field in the event using the selected form input values.

If the evnt is defined in the last cell of the notebook, it will be **automatically sent to the `WebSink`**.

It is important to mention that your automation can have any functionality you desire. The `WebFormSource` can only be used just as a trigger for your automation, what you do with jupyter cells and the event data is up to you.

For achieve our goal we simply date form data from the event and create `parsed_date` which we can plug in into our implementation of the stocks retrieval and `plot_with_pandas` function. The image is returned as a buffer which we can then set as the response in the event and by changing the `content_type` to `image/json` we can render it in our browser.

Wow! You've just created your first automation using a WebForm source. 🎉

### 6. Run the automation

### Conclusion

In this guide, we walked through the process of creating a **WebForm-based automation** using BitSwan Jupyter Automations. You learned how to:

- Import and set up required modules

- Define a form-based source

- Process and structure incoming form data

- Output the final result via a sink

This example is a great starting point. From here, you can expand your automations with custom logic, additional sources and sinks, and other powerful BitSwan capabilities.

If you have any questions or need assistance, feel free to reach out — we're here to help!

Happy automating! 🚀
