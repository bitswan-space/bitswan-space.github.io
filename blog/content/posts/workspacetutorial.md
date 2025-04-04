---
title: "Getting Started with Bitswan Workspace"
date: 2025-03-25T10:36:05+01:00
author: "dolezal"
description: "Setting Up Your Workspace with Bitswan on a VPS or Cloud Provider"
tags: ["Tutorial", "BitSwan", "Bitswan Workspace"]
categories: ["Bitswan Workspace"]
draft: false
---

This post will give you a brief introduction to setting up your workspace on your VPS, whether on your favorite cloud provider or locally!
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

```bash
bitswan workspace init --domain=workspace.test.io <gitops_name>
```

For local development run:

```bash
mkcerts --install
```

In this example we will use workspace.local domain for the local development

and then run the command with --mkcert:

```bash
bitswan workspace init --domain=workspace.local --mkcert <gitops_name>
```

Update your hosts file by adding this:

```bash
127.0.0.1 workspace.local
```
at `/etc/hosts`. Do this only when you want to run the workspaces locally. With the mkcerts and your domain in hosts file, you should be good to go. Without the certs you won't be able to open the editor later on using the https, which causes the editor to not work properly.

4. **Access the Editor**

After running the initialization command, the CLI will provide you with a URL and a password. If everything is configured correctly, you should see your VS Code editor successfully deployed.

Example output:
```bash
------------BITSWAN EDITOR INFO------------
Bitswan Editor URL: https://gitops_name-editor.workspace.test.io
Bitswan Editor Password: a1b2c3d4e5f6g7h8i9j0
------------GITOPS INFO------------
GitOps ID: jachym-gitops
GitOps URL: https://gitops_name-gitops.workspace.test.io
GitOps Secret: 321ab12b13k12jk312kjjk12
```

5. **Open The Editor**

Open the Editor URL in your browser and use the password to login. The site should look like this:
![BitswanEditorLoggingScreen](/devblog/images/webforms/login_page.png)

Give it the `Bitswan Editor Pasword` from the output.

After logging in, you should see the editor with the Bitswan extension installed.

![BitswanEditorVSCScreen](/devblog/images/webforms/vsc_page.png)

7. **Start developing your automations**
Copy an example automation folder from the examples/ directory into your workspace/ folder within the editor.

8. **Deploy your first automation**
Click on Deploy Automation in the top-center Jupyter panel of the editor.

9. **Manage your automations**
Use the Bitswan VS Code extension (automatically installed via the CLI) to monitor automation status, view logs, restart, or stop processes as needed.

### Conclusion
You have successfully set up your Bitswan workspace on a VPS or cloud provider. You're now ready to begin developing and deploying your own automations directly within the browser-based environment! Let me know if you want a shortened version or one with extra formatting like collapsible sections or callouts.


If you want to get started with the editor to develop and deploy your first automation, check out our blog post: *Getting Started with Bitswan in the VS Code Editor*.
