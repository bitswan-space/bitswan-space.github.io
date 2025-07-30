---
title: "From Stock Automation to Daily Discord Updates with BitSwan"
date: 2025-07-28T15:00:00+01:00
author: "lukas_vecerka"
description: "Learn how to create and deploy a weather-checking automation using BitSwan and Cursor, an AI-powered coding assistant. This step-by-step guide shows how to use natural language to generate code, test locally with the BitSwan CLI, and deploy using the BitSwan VSCode extension. Ideal for developers looking to streamline automation with LLM-based tools."
tags: ["BitSwan","Tutorial", "Cursor","AI coding assistant","LLM automation","Weather API"]
categories: ["BitSwan Automations"]
draft: false
---



# How to Create and Deploy a BitSwan Automation with Cursor and AI

## Introduction to Cursor

Cursor is a smart coding assistant integrated into a full-featured editor. It looks and feels like VSCode because it’s built on top of it, but its true strength comes from its **LLM-based** features. It understands your project and helps you write, fix, and explain the code.

Think of Cursor as a pair programming partner powered by AI. You can type a comment describing what you want, and Cursor will turn it into code. If you need to change the existing logic, just edit it in plain English. This makes it an ideal partner for automation platforms like BitSwan, where projects often have repeatable structures.

If you're building pipelines, scraping data, or using notebooks, Cursor eliminates the extra work and keeps you focused on the main logic. There's no need to search for the import syntax or wonder where to put that configuration. You describe, and it writes.

## Prompting Cursor

Let’s start by giving Cursor a specific prompt. Since we’re asking it to build an automation from scratch, we’ll be detailed but not overly technical. Cursor will fill in the details.

**Prompt:**

> I would like to create a BitSwan automation that will have a web form with an input for the date. The automation should get the current temperature in Brno for that date and display it nicely. Examples of BitSwan automations are in @/examples. You can get ideas from @/StocksCalculator. Create this pipeline in examples/WeatherChecker.

This prompt clearly sets the context: user input, external data, and visual output. It also directs Cursor to the existing examples it can use or modify. With that done, hit Enter and let it work.

![Cursor](/images/automation-with-cursor/Weather-in-Brno.png)

## Generated BitSwan Automation

Cursor will respond by creating a new folder: `examples/WeatherChecker`. Inside, you'll find a working pipeline in the form of a Jupyter notebook named `main.ipynb`. This notebook does four things:

1. Renders a simple web form for the user to enter a date.  
2. Fetches the temperature for that day in Brno using a weather API.  
3. Processes and cleans the data.  
4. Displays the result nicely, typically as a graph or chart.

The code structure will likely follow BitSwan’s modular style. You’ll see sources, logic, and sinks laid out clearly, making future tweaks easy.

## Running the Automation Locally

To test it, use the BitSwan CLI. This launches the notebook with the required settings:


bitswan notebook examples/WeatherChecker/main.ipynb -c examples/WeatherChecker/pipelines.conf


This runs the pipeline in the development mode, with settings like API keys and input types loaded from the configuration file. Ensure you have the necessary Python packages installed; Cursor can usually suggest those that are missing.

![Cursor](/images/automation-with-cursor/Weather-in-Brno-0.png)

We recommend working in a virtual environment or a Docker container for a smoother experience. You can also test just the form and the input logic first before fetching live weather data.

## Visual Overview

Let’s make this automation more relatable. You can include:

- A screenshot of the form asking for the date.  
- A chart showing the temperature (e.g., a bar, a line, or a gauge).  
- A pipeline diagram illustrating the flow from the user input to the output.

Visuals are especially helpful for readers new to BitSwan or AI tools like Cursor. They also help convey the value of automating such tasks.

## Deployment with the BitSwan VSCode Extension

Once you’re satisfied with the local results, it’s time to deploy. The good news is that the BitSwan VSCode extension works within Cursor too. This means you won’t have to switch contexts.

### Prerequisites

Before deploying, make sure:

- Your project is open in Cursor or VSCode.  
- The BitSwan extension is installed.  
- Your automation code is saved and staged if using Git.  
- You have a **BitSwan workspace setup locally, in our cloud, or on your VPS**.  

### Deploying

> In order to deploy this pipeline, we need to create a new BitSwan PRE with `matplotlib`. The `Dockerfile` is created by Cursor too, so we just need to use the BitSwan extension to build the image and use it in deployment.

To deploy:

1. Open the notebook (`main.ipynb`).  
2. Use the command palette: **BitSwan: Deploy Notebook**.  
3. **Make sure the workspace is added to the BitSwan extension and activated.**  
4. Select your environment: **local, Docker, or remote**.

Done. The pipeline is now live.

Want to automate the deployment? For more details, check out the BitSwan Deployment Tutorial.

## Conclusion

That’s it! You’ve just used natural language to generate, test, and deploy a weather-checking automation. Cursor handled the coding, BitSwan ran the pipeline, and you stayed focused on the results.

This approach scales well—not just for weather—but for any automation that combines user input, external APIs, and output logic. Begin with a simple idea and grow from there. The less code you write manually, the more time you have for innovation.

If you’re excited about what you’ve created, share it or remix it. AI-assisted development is now a reality—and very practical. Give it a try and enjoy coding your next project.