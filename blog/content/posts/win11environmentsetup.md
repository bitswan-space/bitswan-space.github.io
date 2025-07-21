---
title: "Setting up a local development environment for the Bitswan library on Windows 11"
date: 2025-06-20T15:48:08+01:00
author: "patrik_kiseda"
description: "This tutorial provides step-by-step guidance to set up a local development environment on Windows 11 for contributing to the Bitswan library. If you’re looking to build or modify the Bitswan codebase itself.   
  This guide shows how to install WSL, configure VSCode, create a Python virtual environment, and register Jupyter kernels.
"
tags: ["Tutorial", "Windows 11", "Automatons", "Wsl", "Environment"]
categories: ["Bitswan Workspace"]
draft: false
---

---

## Prerequisites
- Windows 11
- Administrator privileges

---

## Installation Scenarios

### Scenario 1: Fresh Installation
Follow these steps if you have neither WSL nor Visual Studio Code installed.  
(The simplest way to check if you have both of them installed you can just pressing the ```win+s``` shortcut and looking up the ```WSL``` and ```Visual Studio Code```/```vscode``` and seeing if they show up)

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
![PowerShell](/images/win11environmentsetup/win11environmentsetup-PowerShell.png)  

2. Run:
```powershell
wsl --install
```
![WSLInstallation](/images/win11environmentsetup/win11environmentsetup-WSLInstallation.png)  
3. Restart your PC when/if prompted.

### Installing VSCode
1. Download VSCode from [official site](https://code.visualstudio.com/).
2. Run the installer and follow the setup wizard.

### Installing Jupyter Extension
1. Open VSCode.
2. Click on **Extensions** (Ctrl+Shift+X).
3. Search for "Jupyter" and install it.
![gettingAJupyterNotebookExtension](/images/win11environmentsetup/win11environmentsetup-jupyterPlugin.png)

---

## Cloning Bitswan Repository
1. Launch WSL terminal and navigate to the desired directory where you want to store.
2. Clone repository:
```bash
git clone git@github.com:bitswan-space/BitSwan.git
cd BitSwan
```
**Best practice:** use your WSL home directory (~/projects) rather than /mnt/c to avoid permission issues (use the ```cd ~``` to drop into the wsl home directory).
![cloningARepo](/images/win11environmentsetup/win11environmentsetup-cloningARepository.png)


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
pip3 install -e ".[dev]"
```
![activatingEnvironment](/images/win11environmentsetup/win11environmentsetup-activatingTheEnvironment.png)


## Running Bitswan Pipelines
Run a pipeline example using:
```bash
bitswan-notebook examples/WebForms/main.ipynb
```
Optionally watch for changes:
```bash
bitswan-notebook examples/WebForms/main.ipynb --watch
```
![RunningAutomaton](/images/win11environmentsetup/win11environmentsetup-runningAnAutomaton.png)

You can test the basic functionality of some of the examples on **http://localhost:8080** in your browser:  
![localTesting](/images/win11environmentsetup/win11environmentsetup-localTest.png)

## Running Notebooks Cell-by-Cell

In addition to running the full pipeline via CLI, you can also open .ipynb files in VSCode and run them cell-by-cell using the built-in Jupyter notebook interface. This is really handy, if you want to learn, debug, and explore tre pipeline behavior step by step.

⚠️ Kernel Issues: If the Python kernel is incorrect or missing (indicated by errors like "Kernel not found" or "Incorrect Python environment"), it means the notebook doesn't know which Python interpreter (environment) to use. Register your virtual environment as a kernel by running:

```
pip install ipykernel
python -m ipykernel install --user --name=bitswan-env --display-name "Python (bitswan-env)"
```

Then, in VSCode, select this kernel (top-right corner of the notebook).
![selectingKernel](/images/win11environmentsetup/win11environmentsetup-activatingTheKernel.png)

## Testing Pipelines
Run pipeline tests:
```bash
bitswan-notebook examples/Testing/InspectError/main.ipynb --test
```

Automatically re-run tests on file changes:
```bash
bitswan-notebook examples/Testing/InspectError/main.ipynb --test --watch
```
![RunningTests](/images/win11environmentsetup/win11environmentsetup-runningTests.png)

---

## Troubleshooting & Common Issues

Command not found? Confirm venv activation and installation (```pip3 install -e ".[dev]"```).

Permission errors? Avoid Windows-mounted directories (/mnt/c) and use Linux-native directories.


## Conclusion
You now have a fully operational environment to run and test Bitswan pipelines on Windows 11 using WSL and VSCode.


