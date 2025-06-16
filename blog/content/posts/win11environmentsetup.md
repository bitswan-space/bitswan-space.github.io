---
title: "Setting up a development environment for BitSwan on Windows 11"
date: 2025-06-13T09:46:05+01:00
author: "patrik_kiseda"
description: "This tutorial provides step-by-step guidance to set up your environment on Windows 11 for running and testing Bitswan pipelines. Using the Windows Subsystem for Linux (WSL), Visual Studio Code (VSCode), Python virtual environments, and Jupyter Notebook extensions, you'll establish an efficient workspace capable of executing Bitswan pipelines seamlessly.
"
tags: ["Tutorial", "Windows 11", "Automatons", "Wsl", "Environment"]
categories: ["Bitswan Workspace"]
draft: false
---


## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Scenarios](#installation-scenarios)
  - [Scenario 1: Fresh Installation (No WSL, No VSCode)](#scenario-1)
  - [Scenario 2: WSL and VSCode Installed (No Jupyter Extension)](#scenario-2)
  - [Scenario 3: Complete Setup (WSL, VSCode, Jupyter Extensions Installed)](#scenario-3)
- [Cloning Bitswan Repository](#cloning-bitswan-repository)
- [Setting up Python and Virtual Environment](#setting-up-python-and-virtual-environment)
- [Running Bitswan Pipelines](#running-bitswan-pipelines)
- [Testing Pipelines](#testing-pipelines)
- [Conclusion](#conclusion)

---

## Prerequisites
- Windows 11
- Administrator privileges

---

## Installation Scenarios

### Scenario 1: Fresh Installation
Follow these steps if you have neither WSL nor Visual Studio Code installed.

1. [Install Windows Subsystem for Linux (WSL)](#installing-wsl)
2. [Install Visual Studio Code](#installing-vscode)
3. [Install Jupyter Notebook Extensions in VSCode](#installing-jupyter-extension)

### Scenario 2: WSL and VSCode Installed
If you have WSL and VSCode installed but need the Jupyter extensions:

1. [Install Jupyter Notebook Extensions in VSCode](#installing-jupyter-extension)

### Scenario 3: Complete Setup
If WSL, VSCode, and Jupyter extensions are already installed, jump directly to:

1. [Cloning Bitswan Repository](#cloning-bitswan-repository)

---

### Installing WSL
1. Open **PowerShell** as Administrator.  
![PowerShell](\images\win11environmentsetup\win11environmentsetup-PowerShell.png)  

2. Run:
```powershell
wsl --install
```
![WSLInstallation](\images\win11environmentsetup\win11environmentsetup-WSLInstallation.png)  
3. Restart your PC when prompted.

### Installing VSCode
1. Download VSCode from [official site](https://code.visualstudio.com/).
2. Run the installer and follow the setup wizard.

### Installing Jupyter Extension
1. Open VSCode.
2. Click on **Extensions** (Ctrl+Shift+X).
3. Search for "Jupyter" and install it.
![gettingAJupyterNotebookExtension](\images\win11environmentsetup\win11environmentsetup-jupyterPlugin.png)

---

## Cloning Bitswan Repository
1. Launch WSL terminal.
2. Clone repository:
```bash
git clone git@github.com:bitswan-space/BitSwan.git
cd BitSwan
```
![cloningARepo](\images\win11environmentsetup\win11environmentsetup-cloningARepository.png)

---

## Setting up Python and Virtual Environment
1. Ensure Python 3 is installed:
```bash
sudo apt update
sudo apt install python3 python3-venv python3-pip
```
2. Create and activate a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
```
3. Install dependencies:
```bash
pip3 install -e "[dev]"
```
![activatingEnvironment](\images\win11environmentsetup\win11environmentsetup-activatingTheEnvironment.png)


## Running Bitswan Pipelines
Run a pipeline example using:
```bash
bitswan notebook examples/WebForms/main.ipynb
```
Optionally watch for changes:
```bash
bitswan notebook examples/WebForms/main.ipynb --watch
```
![RunningAutomaton](/images/win11environmentsetup/win11environmentsetup-runningAnAutomaton.png)

You can test the basic functionality of some of the examples on **http://localhost:8080** in your browser:  
![localTesting](\images\win11environmentsetup\win11environmentsetup-localTest.png)


## Testing Pipelines
Run pipeline tests:
```bash
bitswan notebook examples/Testing/InspectError/main.ipynb --test
```

Automatically re-run tests on file changes:
```bash
bitswan notebook examples/Testing/InspectError/main.ipynb --test --watch
```
![RunningTests](/images/win11environmentsetup/win11environmentsetup-runningTests.png)

---

## Conclusion
You now have a fully operational environment to run and test Bitswan pipelines on Windows 11 using WSL and VSCode.


