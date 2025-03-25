---
title: "Getting Started with Bitswan Workspace"
date: 2025-03-25T10:36:05+01:00
author: "dolezal"
description: "Setting Up Your Workspace with Bitswan on a VPS or Cloud Provider"
tags: ["Tutorial", "BitSwan", "Bitswan Workspace"]
categories: ["Bitswan Workspace"]
draft: true
---

This post will give you a brief introduction into how to setup your workspace on your VPS or your favorite cloud provider!

### Prerequisities

- `git`, `linux/macos` environment, `docker`, and `bitswan-workspace-cli`

---

### Tutorial

1. **Install Bitswan Workspace CLI**

You can find the installation instructions for the Bitswan Workspace CLI [here](https://github.com/bitswan-space/bitswan-workspaces)

```bash
curl -L https://github.com/bitswan-space/bitswan-gitops-cli/releases/latest/download/bitswan-gitops-cli_Linux_x86_64.tar.gz | tar -xz
```

3. **Setup DNS**
Setup your DNS to have a domain `test.io` for example and a subdomain for your workspace `workspace.test.io` and a wildcard for subdomains `*.workspace.test.io`.

3. **Create a Workspace**

Connect to your VPS or cloud provider and create a workspace using the Bitswan Workspace CLI.

```bash
bitswan-workspace-cli init --domain=<your_domain> gitops_name
```

4. **Access the Editor**

After running the initialization command, the CLI will provide you with a URL and a password. If everything is configured correctly, you should see your VS Code-based editor successfully deployed.

Example output:
```bash
------------BITSWAN EDITOR INFO------------
Bitswan Editor URL: https://gitops_name-editor.workspace.test.io
Bitswan Editor Password: a1b2c3d4e5f6g7h8i9j0
------------GITOPS INFO------------
```

5. **Open The Editor**

Open the URL in your browser and use the password to login.
<!-- TODO fix images -->
![BitswanEditorLoggingScreen](/static/images/webforms/url.png)

After logging in, you should see the editor with the Bitswan extension installed.

7. **Start developing your automations**
Copy any example automation folders from the examples/ directory into your workspace/ folder within the editor.

8. **Deploy your first automation**
Click on Deploy Automation in the top-center Jupyter panel of the editor.

9. **Manage your automations**
Use the Bitswan VS Code extension (automatically installed via the CLI) to monitor automation status, view logs, restart, or stop processes as needed.

### Conclusion
You have successfully set up your Bitswan workspace on a VPS or cloud provider. You're now ready to begin developing and deploying your own automations directly within the browser-based environment! Let me know if you want a shortened version or one with extra formatting like collapsible sections or callouts.
