# 🛡️ Lacework DSPM — Windows Setup Guide

> **Data Security Posture Management (DSPM)** integration with AWS via Lacework, deployed using Terraform on Windows.

---

## 📋 Table of Contents

| # | Step |
|---|------|
| 1 | [Install AWS CLI](#1--install-aws-cli) |
| 2 | [Install Chocolatey](#2--install-chocolatey) |
| 3 | [Install Git](#3--install-git) |
| 4 | [Install Terraform](#4--install-terraform) |
| 5 | [Install Lacework CLI](#5--install-lacework-cli) |
| 6 | [Verify All Installations](#6--verify-all-installations) |
| 7 | [Create a Lacework API Key](#7--create-a-lacework-api-key) |
| 8 | [Log in to AWS from PowerShell](#8--log-in-to-aws-from-powershell) |
| 9 | [Configure the Lacework CLI](#9--configure-the-lacework-cli) |
| 10 | [Clone the DSPM Terraform Repository](#10--clone-the-dspm-terraform-repository) |
| 11 | [Edit the Main Terraform File](#11--edit-the-main-terraform-file) |
| 12 | [Run Terraform](#12--run-terraform) |
| 13 | [Complete DSPM Setup in Portal](#13--complete-dspm-setup-in-the-lacework-portal) |

---

> ⚠️ **All commands must be run in PowerShell (Run as Administrator)** unless otherwise noted.

---

## 1 · Install AWS CLI

<details>
<summary><strong>Option A — Official MSI Installer (Recommended)</strong></summary>

**Download:**
```powershell
Invoke-WebRequest `
  -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" `
  -OutFile "AWSCLIV2.msi"
```

**Install silently:**
```powershell
Start-Process msiexec.exe -Wait -ArgumentList '/I AWSCLIV2.msi /quiet'
```

</details>

<details>
<summary><strong>Option B — winget</strong></summary>

```powershell
winget install -e --id Amazon.AWSCLI
```

</details>

**✅ Verify:**
```powershell
aws --version
```

---

## 2 · Install Chocolatey

Chocolatey is the Windows equivalent of Homebrew — used to install and manage CLI tools.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
$sp = [System.Net.ServicePointManager]
$sp::SecurityProtocol = $sp::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

> 🔄 **Close and reopen PowerShell as Administrator** before continuing.

**✅ Verify:**
```powershell
choco --version
```

---

## 3 · Install Git

<details>
<summary><strong>Option A — Official Installer (Recommended)</strong></summary>

**Download from:** https://git-scm.com/download/win

Or download directly via PowerShell:
```powershell
$gitUrl = "https://github.com/git-for-windows/git/releases/latest/download/Git-2.47.0-64-bit.exe"
Invoke-WebRequest -Uri $gitUrl -OutFile "GitInstaller.exe"
```
> Replace `2.47.0` with the latest version from https://github.com/git-for-windows/git/releases

**Install silently:**
```powershell
Start-Process -Wait `
  -FilePath "GitInstaller.exe" `
  -ArgumentList "/VERYSILENT /NORESTART"
```
Installs to `C:\Program Files\Git` and automatically adds to PATH.

</details>

<details>
<summary><strong>Option B — winget</strong></summary>

```powershell
winget install -e --id Git.Git
```
> Restart PowerShell after installation.

</details>

<details>
<summary><strong>Option C — Chocolatey</strong></summary>

```powershell
choco install git -y
```

</details>

<details>
<summary><strong>⚙️ Manual PATH fix (if Git is not found)</strong></summary>

```powershell
$newPath = $Env:Path + ";C:\Program Files\Git\cmd"
[Environment]::SetEnvironmentVariable("Path", $newPath, [EnvironmentVariableTarget]::Machine)
```
Restart PowerShell after running.

</details>

**✅ Verify:**
```powershell
git --version
```

---

## 4 · Install Terraform

<details>
<summary><strong>Option A — Chocolatey (Recommended)</strong></summary>

```powershell
choco install terraform -y
```

</details>

<details>
<summary><strong>Option B — winget</strong></summary>

```powershell
winget install -e --id Hashicorp.Terraform
```

</details>

<details>
<summary><strong>Option C — Manual Download</strong></summary>

1. Download the latest zip from: https://developer.hashicorp.com/terraform/downloads
2. Extract `terraform.exe` to `C:\terraform`
3. Add to PATH:
```powershell
$newPath = $Env:Path + ";C:\terraform"
[Environment]::SetEnvironmentVariable("Path", $newPath, [EnvironmentVariableTarget]::Machine)
```

</details>

**✅ Verify:**
```powershell
terraform --version
```

---

## 5 · Install Lacework CLI

<details>
<summary><strong>Option A — Chocolatey (Recommended)</strong></summary>

```powershell
choco install lacework-cli -y
```

</details>

<details>
<summary><strong>Option B — Manual Download</strong></summary>

1. Download the latest Windows release from: https://github.com/lacework/go-sdk/releases
2. Extract the `.exe` to `C:\lacework`
3. Add to PATH:
```powershell
$newPath = $Env:Path + ";C:\lacework"
[Environment]::SetEnvironmentVariable("Path", $newPath, [EnvironmentVariableTarget]::Machine)
```

</details>

**✅ Verify:**
```powershell
lacework --version
```

---

## 6 · Verify All Installations

Run this script to check all tools at once:

```powershell
Write-Host "`n============================" -ForegroundColor Cyan
Write-Host "   Tool Version Check" -ForegroundColor Cyan
Write-Host "============================`n" -ForegroundColor Cyan

Write-Host "[1] AWS CLI" -ForegroundColor Yellow
aws --version

Write-Host "`n[2] Chocolatey" -ForegroundColor Yellow
choco --version

Write-Host "`n[3] Git" -ForegroundColor Yellow
git --version

Write-Host "`n[4] Terraform" -ForegroundColor Yellow
terraform --version

Write-Host "`n[5] Lacework CLI" -ForegroundColor Yellow
lacework --version

Write-Host "`n=============================" -ForegroundColor Green
Write-Host "   All checks complete ✓" -ForegroundColor Green
Write-Host "=============================`n" -ForegroundColor Green
```

| Tool | Expected Output |
|------|----------------|
| `aws --version` | `aws-cli/2.x.x Python/3.x.x Windows/x86_64` |
| `choco --version` | `2.x.x` |
| `git --version` | `git version 2.x.x.windows.x` |
| `terraform --version` | `Terraform v1.x.x on windows_amd64` |
| `lacework --version` | `lacework vx.x.x (sha:...)` |

> ❗ If any command returns `'xxx' is not recognized`, the tool is not installed or its path is missing. Refer to the manual PATH fix in each section above.

---

## 7 · Create a Lacework API Key

1. Go to your Lacework portal:
   ```
   https://<your-account>.lacework.net
   ```

2. Navigate to **Settings** → **API Keys**

3. Click **+ Create New** and fill in:
   | Field | Value |
   |-------|-------|
   | **Name** | `dspm-integration` |
   | **Description** | `API key for DSPM integration` *(optional)* |

4. Click **Save**

5. Click **Download** on the confirmation screen to save the `.json` key file

6. Save the file to your Desktop:
   ```
   C:\Users\<YourUsername>\Desktop\dspm-integration-api-key.json
   ```

> 🔐 **Security:** This is the only time the secret key is shown. Download it before navigating away. Never commit this file to version control.

---

## 8 · Log in to AWS from PowerShell

```powershell
aws login
```

- It will prompt you to select your **AWS region**
- It will open a **browser window** for you to log in with your AWS credentials
- Once authenticated in the browser, return to PowerShell — your session is ready

**✅ Verify:**
```powershell
aws sts get-caller-identity
```

---

## 9 · Configure the Lacework CLI

Use the API key JSON file downloaded in Step 7:

```powershell
lacework configure -j "C:\Users\$Env:USERNAME\Desktop\dspm-integration-api-key.json"
```

This reads the API key, secret, and account details automatically from the file.

**✅ Verify:**
```powershell
lacework account list
```

> If successful, it returns the list of accounts your API key has access to.

---

## 10 · Clone the DSPM Terraform Repository

```powershell
git clone https://github.com/lacework/terraform-aws-dspm.git
cd terraform-aws-dspm
```

---

## 11 · Edit the Main Terraform File

Navigate to the example directory:
```powershell
cd terraform-aws-dspm\examples\account-level-single-region
```

Open `main.tf` with your preferred editor:

| Editor | Command |
|--------|---------|
| vim *(Git Bash)* | `vim main.tf` |
| nano *(Git Bash)* | `nano main.tf` |
| Notepad | `notepad main.tf` |
| VS Code | `code main.tf` |

> 💡 For vim/nano, open **Git Bash** by right-clicking in the folder and selecting **"Git Bash Here"**.

### ✏️ Fields to update in `main.tf`

```hcl
account_id = "123456789012"   # Replace with your AWS Account ID
region     = "us-east-1"      # Replace with your target AWS region
```

> 💡 To find your AWS Account ID, run:
> ```powershell
> aws sts get-caller-identity --query Account --output text
> ```

**Save the file:**

| Editor | Save Command |
|--------|-------------|
| vim | `Esc` → `:wq` → `Enter` |
| nano | `Ctrl+O` then `Ctrl+X` |
| Notepad / VS Code | `Ctrl+S` |

---

## 12 · Run Terraform

Make sure you are in the correct directory:
```powershell
cd terraform-aws-dspm\examples\account-level-single-region
```

### Step 1 — Initialise
Downloads all required providers and modules:
```powershell
terraform init
```
> ✅ Expected: `Terraform has been successfully initialized!`

### Step 2 — Plan
Previews what will be created — no changes are made:
```powershell
terraform plan
```
> Review the output carefully for resources to be **added**, **changed**, or **destroyed**.

### Step 3 — Apply
Deploys all resources to AWS:
```powershell
terraform apply
```
When prompted, type `yes` and press `Enter`:
```
Do you want to perform these actions? yes
```
> ✅ Expected: `Apply complete! Resources: X added, 0 changed, 0 destroyed.`

---

## 13 · Complete DSPM Setup in the Lacework Portal

1. Go to your Lacework portal:
   ```
   https://<your-account>.lacework.net
   ```

2. Navigate to **Settings** → **DSPM**

3. Find your integration listed as **"Setup Required"** and click on it

4. Configure the scan settings:

   | Setting | Description |
   |---------|-------------|
   | **S3 Buckets** | Select buckets to include or exclude from scanning |
   | **Maximum File Size** | Enter the largest file size (MB) to scan |
   | **Scan Frequency** | Set how often scans run (e.g. daily, weekly) |

5. Click **Save**

6. The status will update to ✅ **Active** — integration is complete

7. Confirm under **Settings** → **DSPM** that the status shows **Active**

> ⏱️ Initial scan results will appear in the portal within **2–3 hours** after activation.

---

## 🗂️ Quick Reference

| Tool | Install Method | Default Path | Verify Command |
|------|---------------|--------------|----------------|
| AWS CLI | MSI / winget | Auto-added to PATH | `aws --version` |
| Chocolatey | PowerShell script | Auto-added to PATH | `choco --version` |
| Git | `.exe` / winget / choco | `C:\Program Files\Git\cmd` | `git --version` |
| Terraform | choco / winget / manual | `C:\terraform` | `terraform --version` |
| Lacework CLI | choco / manual | `C:\lacework` | `lacework --version` |

---

> 💡 **Tip:** After any manual PATH change, run `refreshenv` (Chocolatey) or restart PowerShell for changes to take effect.
