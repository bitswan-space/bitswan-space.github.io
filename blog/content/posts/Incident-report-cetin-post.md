---
title: "From Stock Automation to Daily Discord Updates with BitSwan"
date: 2025-07-28T15:00:00+01:00
author: "lukas_vecerka"
description: "Learn how to create and deploy a weather-checking automation using BitSwan and Cursor, an AI-powered coding assistant. This step-by-step guide shows how to use natural language to generate code, test locally with the BitSwan CLI, and deploy using the BitSwan VSCode extension. Ideal for developers looking to streamline automation with LLM-based tools."
tags: ["BitSwan","Tutorial", "Cursor","AI coding assistant","LLM automation","Weather API"]
categories: ["BitSwan Automations"]
draft: false
---

# Anomaly to Insight: A Real-World Root Cause Analysis

Services are stalling, devices are disappearing from dashboards, and metrics are inconsistent. The logs do not provide clear answers. Welcome to one of our recent real-world incidents in the telecommunications industry.  
We used BitSwan and Grafana to trace a fast-moving infrastructure failure as it unfolded. What began as a sudden spike in memory usage quickly snowballed—CPU load surged, services slowed, clients disconnected, and MongoDB lost its primary.  
By piecing together system metrics, timeline correlations, and engineering instincts, we unraveled the root cause. This analysis demonstrates how observability tools and focused debugging can transform fragmented symptoms into clear, actionable insights.

## Monitoring Data

### RAM Usage Patterns  
The first red flag appeared with a sudden spike in RAM usage across G1, G2, and G3 nodes. This wasn’t a gradual increase; it was a clear and immediate anomaly. Grafana detected it early, and the dashboards revealed that this wasn’t an isolated incident. Whatever caused it affected the cluster simultaneously.  
When we looked at previous trends, this behavior stood out as abnormal. We started examining logs and upstream dependencies, suspecting that something systemic was causing memory pressure.

### CPU Load Analysis  
As RAM usage increased, CPU load jumped just seconds later, especially on M1 and M2 nodes. This was not just high usage; it was the CPU being overwhelmed by load.  
From a developer's perspective, this kind of pattern typically indicates memory saturation that leads to GC loops. It was clear this situation required immediate action.

### Device Drop-Offs  
Then the edge devices began to disconnect. Monitoring alerted us to a sharp decline in connections, right when CPU and RAM peaked. Clients were either timing out or disconnecting.  
Our device management dashboards matched up perfectly with Grafana’s backend data. The backend couldn’t keep up, and the ripple effect severely impacted the clients.

![gettingAJupyterNotebookExtension](/images/Incident-report/MongoDB-Primary-Node-election.png)


### MongoDB Behavior  
Just as we were troubleshooting CPU and memory, MongoDB added to the confusion. Within minutes of the system becoming overloaded, MongoDB's replica set automatically triggered two primary elections, meaning that the database detected that its current primary node was too slow or unresponsive—likely due to high CPU and memory pressure—and attempted to recover by choosing a new one. The fact that it had to do this twice in quick succession was a strong signal that the entire cluster was under severe stress. That’s a concerning sign—it means Mongo lost trust in the current primary, likely due to responsiveness issues. The problem spread across layers.

## Correlation Analysis  
By mapping the sequence of events, we confirmed the root cause: a RAM spike across G-nodes caused a CPU overload, which led to degraded services and client disconnects. MongoDB, already under strain, switched its primary twice.  
Our Grafana timeline visualization made this cascade clear in hindsight. In real-time, it was more chaotic, but the data held all the clues.

## Solution  
To stabilize the system, we first reduced the GENIEACS_MAX_CONCURRENT_REQUESTS setting in the GenieACS CWMP component from 1000 to 250. This adjustment immediately helped reduce backend load and brought temporary stability to the application layer.  
However, we continued to observe recurring spikes in RAM usage, which closely aligned with high CPU usage on m1, the MongoDB node that had recently been elected as the new primary. Unlike the original primary node (b5-g*), which had no profiling enabled, m1 had MongoDB profiling set to level 2. This profiling configuration introduced significant overhead and degraded performance under load.  
To address this, we disabled profiling on all m* MongoDB nodes. Once this was done, both the RAM and CPU instability affecting the MongoDB primary were fully resolved.

## Mitigation and Recommendations  
Here’s what we changed after the situation calmed:  
- We set stricter limits for our memory-intensive services and isolated them in containers.  
- Our Grafana dashboards now track GC and MongoDB elections as essential metrics.  
- We adjusted alert thresholds and suppressed duplicates to reduce noise during high-load times.  
- Most importantly, we added this scenario to our runbooks so that the next time it happens, the response will be quicker.

