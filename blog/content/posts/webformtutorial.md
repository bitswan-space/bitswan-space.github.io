---
title: "Getting Started with BitSwan Jupyter Automations: Creating Your First WebForm Automation"
date: 2025-03-25T10:36:05+01:00
author: "dolezal"
description: "Learn how to build your first automation using BitSwan Jupyter with a WebForm source. This step-by-step tutorial covers the basics of form creation, event handling, and pipeline execution."
tags: ["Automations","Tutorial", "BitSwan","Jupyter"]
categories: ["BitSwan Automations"]
draft: false
---

This post introduces how to use **BitSwan Jupyter Automations** to create your first automation using a **WebForm source**. You’ll learn how to set up a simple automation pipeline, define a web form, and handle the data submitted through it.


## Jupyter Automation

If you have a BitSwan workspace with the **VS Code editor** and the **BitSwan extension** installed, you’re ready to get started. Below is an example automation script. We’ll go through it cell by cell and explain each part of the pipeline.

### 1. Import Required Modules
```python
from bspump.jupyter import *
from bspump.http.web.server import *
```

The first cell as usual imports the necessary modules.
- `bspump.jupyter` module is used to run and handle the automation in the jupyter notebook.
- The `bspump.http.web.server` module is used to create a web server that will serve the web form.

### 2. Define an Initial Event (Optional)
```python
event = {
    "form" : {"textfile": "text"}
}
```
This optional cell defines an `event` dictionary that will be used throughout the automation to store and access data. It can help structure how data flows through the pipeline.

### 3. Define the Automation Pipeline
```python
auto_pipeline(
    source=lambda app, pipeline: WebFormSource(app, pipeline, route="/", fields=[
        TextField("foo", display="Foo"),
        TextField("la", readonly=True, default="laa"),
        IntField("n"),
        TextField("le", hidden=True),
        ChoiceField("lol", choices=["a","b","c"]),
        CheckboxField("checkbox"),
    ]),
    sink=lambda app, pipeline: WebSink(app,pipeline)
)
```
This is the **core of the automation**. We use `auto_pipeline` to define our pipeline with minimal boilerplate:

- Source: `WebFormSource` renders a form at the root path (`/`) with a set of fields.
- Sink: `WebSink` handles the final processed output of the automation. and displays it on the web page.

The form includes:
- Text fields (some with defaults or hidden)
- An integer input
- A dropdown (choice field)
- A checkbox

👉 You can mix and match different sources and sinks to create your desired automation.

TODO: [Add a link to the documentation with a full list of available Sources and Sinks.]

### 4. Handle Form Submission

```python
event["response"] = {
            "msg": event["form"]["foo"],
            "checkbox": event["form"]["checkbox"],
            "n": event["form"]["n"],
            "lol": event["form"]["lol"]
        }
```
Cell below the `automation_pipeline` definition are executed when the pipeline is triggered.

In this case:
- Once the form is sumitted, `WebFormSource` creates an event.
- The `event` object includec the submitted `form` data.
- We consturct a new `response` field in the event using the selected form input values.

If the evnt is defined in the last cell of the notebook, it will be **automatically sent to the `WebSink`**.

It is important to mention that your automation can have any functionality you desire. The `WebFormSource` can only be used just as a trigger for your automation, what you do with jupyter cells and the event data is up to you.


Wow! You've just created your first automation using a WebForm source. 🎉

### Conclusion

In this guide, we walked through the process of creating a **WebForm-based automation** using BitSwan Jupyter Automations. You learned how to:

- Import and set up required modules

- Define a form-based source

- Process and structure incoming form data

- Output the final result via a sink

This example is a great starting point. From here, you can expand your automations with custom logic, additional sources and sinks, and other powerful BitSwan capabilities.

If you have any questions or need assistance, feel free to reach out — we're here to help!

Happy automating! 🚀
