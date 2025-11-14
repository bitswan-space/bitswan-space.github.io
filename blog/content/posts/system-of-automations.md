---
title: "System of simple Automations: Designing Maintainable BitSwan Pipelines"
date: 2025-10-09T9:25:00+01:00
author: "patrik_kiseda" 
description: "Practical guide to building maintainable automations with BitSwan’s auto_pipeline: how to decompose problems, choose right sources and sinks, move quickly with cell-by-cell execution, while keeping it all secured."
tags: ["Tutorial", "Automation", "BitSwan", "Cron", "WebForms", "Secrets", "CouchDB", "Google Docs"]
categories: ["Automations"]
draft: false
---

---

## Why this guide

When you build automations, code size and complexity can build up fast. BitSwan’s `auto_pipeline` keeps each automation small, streamlined and testable. This article explores patterns and pitfalls we met while building a real, multi‑part automation (this example: web form + weekly document update + Discord notification) and generalizes them for the Automation Operations Centre (AOC) context.

---

## A design choice: modular vs. monolithic

To keep things maintainable, easy to edit, and reusable, keep automations modular. In BitSwan, this translates to:

- Separate pipelines per concern (e.g., form pipeline, scheduled pipeline). Longer code is harder to debug and operate.
- Keep secrets external and portable between AOC and local dev.
- Make behavior both triggerable on demand while still testable with jupyter notebook cell‑by‑cell execution.

The practical outcome: one pipeline serves a protected web form and stores input; another pipeline runs on a `CronSource` that utilizes a time trigger to execute the pipeline's code at a given time. It reads the latest document, updates a Google Doc, and notifies Discord. Either pipeline can be run, tested, and deployed independently.

---

## Architecture 101: Sources, processing cells, and Sinks

- **Sources**: These define how and when your code runs—they trigger the execution of your pipeline. In other words, a source is where events come from *and* what starts the processing in your notebook. Examples: 
-`WebFormSource`(simple source useful for testing), -
`ProtectedWebFormSource`(a secure version of the `WebFormSource`), 
-`CronSource`( time-triggered source).
- **Processing code**: Cells after `auto_pipeline(...)` are executed when an event flows through; they update the `event` or interact with services.
- **Sinks**: Where results go. Examples: `WebSink` (HTTP response), `PPrintSink` (logs), or a custom `Sink`.

**Note:** Helper code note: You can define helper functions/classes above the `auto_pipeline(...)` cell and call them from processing cells. You can also import helpers from other files (standard Python modules). For web UIs, serve JS/CSS from stable paths (e.g., `/styles/`, `/scripts/`) and reference them in field HTML.

Minimal skeleton:

```python
from bspump.jupyter import *
from bspump.http.web.server import *
import json

event = {"form": {"subject": "Hello"}} # If testing the cell-by-cell execution

auto_pipeline(
    source=lambda app, pipeline: WebFormSource(app, pipeline, route="/", fields=[
        TextField("subject"),
    ]),
    sink=lambda app, pipeline: WebSink(app, pipeline),
    name="FormPipeline"
)

# processing cell: runs when the form is submitted
event["response"] = json.dumps({"echo": event["form"].get("subject", "")})
event["content_type"] = "application/json"
```

---

## Plan first: divide and conquer

Break bigger goals into small pipelines:

- **Form with a database save**: A protected form pipeline to collect data and save it (for example: CouchDB). Reusable and safe to work on without compromising other module functionality.
- **Scheduled work**: A cron‑driven pipeline (that is, a pipeline triggered automatically at specific times or intervals) that reads the latest data and acts (for example: updating a doc on a given time, sending a timed message).

---

## Secrets in AOC and locally

- In AOC, secrets live under `/home/coder/workspace/secrets/` and are loaded into environment variables at runtime.
- If working locally, you can keep `secrets/` next to your notebook and load with `python-dotenv`.

Example local load:

```python
import os
from pathlib import Path
from dotenv import load_dotenv

base = Path.cwd()
secrets_dir = base / "secrets"
for name in ["webform", "couchdb", "google-docs", "discord"]:
    p = secrets_dir / name
    if p.exists():
        load_dotenv(p, override=True)
```

---

## Imports that simplify notebooks

- `from bspump.jupyter import *` gives you `auto_pipeline`, async helpers, and testing ergonomics expected by BitSwan notebooks.
- `from bspump.cron import CronSource` for cron scheduling.
- Import sinks/sources explicitly when reading code clarity matters (e.g., `from bspump.http.web.server import WebFormSource, WebSink`).

---

## Events: testing before the pipeline runs

In notebooks, you are able to define a sample `event` yourself, before the `auto_pipeline` call to test processing cells without hitting the real source:

```python
event = {"form": {"subject": "Hello"}}
```

You can run cells one by one in the notebook and test iteratively this way.

---

Common source categories (not exhaustive):

- **Web & HTTP**: web forms, webhook/HTTP endpoints, protected variants for production
- **Time‑based**: cron/time triggers
- **Files & FS**: file tailers, directory scanners, CSV/JSON readers
- **Message & Streams**: Kafka/AMQP/MQTT/topic consumers
- **Databases & Change Feeds**: pollers or change streams (e.g., DB `_changes` feeds)
- **Cloud/APIs**: HTTP polling of REST/Graph endpoints
- **Custom**: lightweight `Source` subclasses for domain‑specific producers

For this use‑case:

- Use `ProtectedWebFormSource` to create a secure form (local demos can use `WebFormSource`).
- Use `CronSource` to run weekly on a schedule and process the latest stored data.
- Optionally add an HTTP poller/webhook source if you need to ingest external updates before the scheduled run.

Current limitations to keep in mind:

- Long‑running steps belong in processing or external workers, not in sources.
- Cron is time‑based, not event‑based; if you need idempotency, implement it in processing (e.g., compare last processed timestamp/document id).

---

## Sinks: where work completes

- **WebSink**: Returns an HTTP response from a form pipeline.
- **PPrintSink**: Good for debugging—prints events to logs.
- **Custom Sink**: Encapsulate actions like posting to Discord or updating a Google Doc. 

Example custom sink (synchronous request):

```python
from bspump.abc.sink import Sink
import requests

class DiscordSink(Sink):
    def process(self, context, event):
        url = os.getenv("DISCORD_WEBHOOK_URL", "")
        try:
            requests.post(url, json={"content": event.get("message", "")}, timeout=10)
        except Exception ...
        return None
```

---

## What `auto_pipeline` does behind the scenes

Conceptually:

- Registers your `source(...)` and `sink(...)` with the BitSwan app.
- Wraps post‑`auto_pipeline` cells into an async processing step that receives `(context, event)` and runs for every event.
- Ensures your notebook can be run headless via `bitswan-notebook` and interactively cell‑by‑cell.

Implications for your code:

- Keep in mind that post‑`auto_pipeline` cells can be run multiple times(as many times as triggered) and beware of side effects of that.
- Always set `event["content_type"]` when returning a body; prefer JSON for structured responses.
- Use clear try/except blocks and return reasonable fallbacks.

---

## Cell‑by‑cell execution: speed and confidence

Advantages:

- Shorten feedback loops: test parsing, formatting, and external calls with mocked events.
- Safer refactors: small cells surface errors early.
- Easier onboarding: teammates can run just the relevant cells to understand flow.

---

## Common pitfalls and how to avoid them

- **Time handling**: BitSwan currently works with offset‑naive datetimes. Stick to naive times throughout triggers to avoid mismatches.
- **Secrets**: In AOC, secrets are mounted (e.g., under `/home/coder/workspace/secrets/`) and loaded into environment variables—treat this as the primary mode. If developing locally, you can for example keep a sibling `secrets/` directory and load it via `python-dotenv`.
- **Third‑party deps**: If you need extra packages, you can use `pip install` directly in notebooks to make sure everything you need is available.


---

## Example: split Form triggered and Cron triggered pipelines

```python
# form pipeline: collect + store
auto_pipeline(
    source=lambda app, pipeline: ProtectedWebFormSource(app, pipeline, route="/", fields=[
        TextField("subject"),
    ], config={"secret": os.getenv("WEBFORM_SECRET", "")} ),
    sink=lambda app, pipeline: WebSink(app, pipeline),
    name="FormPipeline"
)

# ... cells that parse event["form"], save to DB, and respond JSON

# cron pipeline: read latest + act
auto_pipeline(
    source=lambda app, pipeline: CronSource(app, pipeline, config={"when": "40 14 * * 1"}),
    sink=lambda app, pipeline: PPrintSink(app, pipeline),
    name="WeeklyPipeline"
)

# ... cells that read last document and perform actions (e.g., update Google Doc, send Discord message)
```

---

## Conclusion

Treat each automation as a small, observable unit: one clear source, a few explicit processing cells, and a sink. Keep secrets portable, test with sample `event` data, and split responsibilities across pipelines. With these patterns, BitSwan automations remain easy to reason about—and easy to run in AOC.