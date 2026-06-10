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
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "AWSCLIV2.msi"
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
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
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
Invoke-WebRequest -Uri "https://github.com/git-for-windows/git/releases/latest/download/Git-2.47.0-64-bit.exe" -OutFile "GitInstaller.exe"
```
> Replace `2.47.0` with the latest version from https://github.com/git-for-windows/git/releases

**Install silently:**
```powershell
Start-Process -Wait -FilePath "GitInstaller.exe" -ArgumentList "/VERYSILENT /NORESTART"
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
[Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\Program Files\Git\cmd", [EnvironmentVariableTarget]::Machine)
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
[Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\terraform", [EnvironmentVariableTarget]::Machine)
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
[Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\lacework", [EnvironmentVariableTarget]::Machine)
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


**Option A — Official MSI Installer (Recommended)**

1. Download the installer:
   ```powershell
   Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "AWSCLIV2.msi"
   ```
2. Run the installer silently:
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/I AWSCLIV2.msi /quiet'
   ```
3. Verify the installation:
   ```powershell
   aws --version
   ```

**Option B — winget**
```powershell
winget install -e --id Amazon.AWSCLI
```

---

## 2. Install Chocolatey (Windows equivalent of Homebrew)

Chocolatey is the Windows package manager equivalent of Homebrew on macOS.

1. Install Chocolatey:
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force
   [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```
2. Verify the installation:
   ```powershell
   choco --version
   ```
3. Close and reopen PowerShell as Administrator before continuing.

---

## 3. Install Git

**Option A — Official Installer (Recommended)**

1. Download the latest Git for Windows installer from:
   ```
   https://git-scm.com/download/win
   ```
   Or download directly via PowerShell:
   ```powershell
   Invoke-WebRequest -Uri "https://github.com/git-for-windows/git/releases/latest/download/Git-2.47.0-64-bit.exe" -OutFile "GitInstaller.exe"
   ```
   > **Note:** Replace `2.47.0` with the latest version from https://github.com/git-for-windows/git/releases

2. Run the installer:
   ```powershell
   Start-Process -Wait -FilePath "GitInstaller.exe" -ArgumentList "/VERYSILENT /NORESTART"
   ```
   This installs Git to `C:\Program Files\Git` and automatically adds it to your PATH.

3. Verify the installation:
   ```powershell
   git --version
   ```

**Option B — via winget**
```powershell
winget install -e --id Git.Git
```
> winget automatically adds Git to your PATH. Restart PowerShell after installation.

**Option C — via Chocolatey**
```powershell
choco install git -y
```

**Manual PATH setup (if Git is not found after install):**
```powershell
[Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\Program Files\Git\cmd", [EnvironmentVariableTarget]::Machine)
```
Then restart PowerShell and verify:
```powershell
git --version
```

---

## 4. Install Terraform


**Option A — via Chocolatey**
```powershell
choco install terraform -y
```

**Option B — via winget**
```powershell
winget install -e --id Hashicorp.Terraform
```

**Option C — Manual Download**
1. Download the latest Terraform zip from: https://developer.hashicorp.com/terraform/downloads
2. Extract the `terraform.exe` to a folder, e.g. `C:\terraform`
3. Add that folder to your system PATH:
   ```powershell
   [Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\terraform", [EnvironmentVariableTarget]::Machine)
   ```

Verify the installation:
```powershell
terraform --version
```

---

## 5. Install Lacework CLI

**Option A — via Chocolatey**
```powershell
choco install lacework-cli -y
```

**Option B — Manual Download**
1. Download the latest Windows release from: https://github.com/lacework/go-sdk/releases
2. Extract the `.exe` to a folder, e.g. `C:\lacework`
3. Add that folder to your system PATH:
   ```powershell
   [Environment]::SetEnvironmentVariable("Path", $Env:Path + ";C:\lacework", [EnvironmentVariableTarget]::Machine)
   ```

Verify the installation:
```powershell
lacework --version
```

---

## 6. Verify All Installations

Run the following commands in PowerShell to confirm every tool is installed correctly and check its version:

```powershell
# AWS CLI
aws --version
# Expected output: aws-cli/2.x.x Python/3.x.x Windows/x86_64

# Chocolatey
choco --version
# Expected output: 2.x.x

# Git
git --version
# Expected output: git version 2.x.x.windows.x

# Terraform
terraform --version
# Expected output: Terraform v1.x.x on windows_amd64

# Lacework CLI
lacework --version
# Expected output: lacework vx.x.x (sha:...)
```

To run all checks at once and see a clean summary:

```powershell
Write-Host "=== Tool Version Check ===" -ForegroundColor Cyan

Write-Host "`n[AWS CLI]" -ForegroundColor Yellow
aws --version

Write-Host "`n[Chocolatey]" -ForegroundColor Yellow
choco --version

Write-Host "`n[Git]" -ForegroundColor Yellow
git --version

Write-Host "`n[Terraform]" -ForegroundColor Yellow
terraform --version

Write-Host "`n[Lacework CLI]" -ForegroundColor Yellow
lacework --version

Write-Host "`n=== Done ===" -ForegroundColor Cyan
```

> If any command returns `'xxx' is not recognized as an internal or external command`, the tool is either not installed or its install path is not added to your PATH. Refer to the manual PATH setup steps in each section above.

---

## 7. Create a Lacework API Key

1. Open your browser and go to the Lacework portal:
   ```
   https://<your-account>.lacework.net
   ```
   > Replace `<your-account>` with your Lacework account name (e.g. `mycompany.lacework.net`).

2. Log in with your credentials.

3. Navigate to **Settings** → **API Keys** (found under the **Configuration** section in the left sidebar).

4. Click **+ Create New** to create a new API key.

5. Fill in the details:
   - **Name:** `dspm-integration`
   - **Description:** *(optional)* e.g. `API key for DSPM integration`

6. Click **Save**.

7. On the confirmation screen, click **Download** to download the API key file (`.json` format).

8. Save the downloaded file to your Desktop:
   - Default desktop path on Windows:
     ```
     C:\Users\<YourUsername>\Desktop\
     ```
   - Rename the file for clarity if needed, e.g.:
     ```
     dspm-integration-api-key.json
     ```

> **Important:** This is the only time the secret key will be shown. Do not close or navigate away before downloading. Keep this file secure and do not commit it to version control.

---

## 8. Log in to AWS from PowerShell

Open **PowerShell (Run as Administrator)** and run:

```powershell
aws login
```

- It will prompt you to select your **AWS region**
- It will then automatically open a **browser window** for you to log in with your AWS credentials
- Once logged in via the browser, return to PowerShell — your session will be authenticated

Verify the login was successful:
```powershell
aws sts get-caller-identity
```

---

## 9. Configure the Lacework CLI

After installing the Lacework CLI (Section 5) and downloading your API key (Section 7), configure the CLI using the key file saved on your Desktop.

Run the following command in PowerShell:

```powershell
lacework configure -j "C:\Users\$Env:USERNAME\Desktop\dspm-integration-api-key.json"
```

This will automatically read the API key, secret, and account details from the downloaded file.

Verify the CLI is connected and authenticated:
```powershell
lacework account list
```

> If successful, it will return the list of accounts your API key has access to.

---

## 10. Clone the Lacework DSPM Terraform Repository

Once all tools are installed and configured, clone the DSPM Terraform repository:

```powershell
git clone https://github.com/lacework/terraform-aws-dspm.git
```

Navigate into the cloned directory:
```powershell
cd terraform-aws-dspm
```

---

## 11. Edit the Main Terraform File

Navigate to the example directory:
```powershell
cd terraform-aws-dspm\examples\account-level-single-region
```

Open the `main.tf` file for editing.

**Option A — Using vim (via Git Bash)**
```bash
vim main.tf
```

**Option B — Using nano (via Git Bash)**
```bash
nano main.tf
```

> **Note:** `vim` and `nano` are available through **Git Bash** (installed with Git for Windows). To open Git Bash, right-click in the folder and select **"Git Bash Here"**, then run the command above.

**Option C — Using Notepad (built-in Windows)**
```powershell
notepad main.tf
```

**Option D — Using VS Code**
```powershell
code main.tf
```

Once the file is open, update the following fields:

### Fields to edit in `main.tf`

1. **AWS Account ID** — replace with your target AWS account ID:
   ```hcl
   account_id = "123456789012"    # Replace with your AWS Account ID
   ```

2. **AWS Region** — replace with the region you want to integrate:
   ```hcl
   region = "us-east-1"           # Replace with your target AWS region
   ```

> **Tip:** To find your AWS Account ID, run:
> ```powershell
> aws sts get-caller-identity --query Account --output text
> ```

Save the file:
- In **vim**: press `Esc`, then type `:wq` and hit `Enter`
- In **nano**: press `Ctrl+O` to save, then `Ctrl+X` to exit
- In **Notepad / VS Code**: press `Ctrl+S`

---

## 12. Run Terraform

Make sure you are in the correct directory before running any Terraform commands:
```powershell
cd terraform-aws-dspm\examples\account-level-single-region
```

### Step 1 — Initialise Terraform

Downloads the required providers and modules:
```powershell
terraform init
```

> Expected output: `Terraform has been successfully initialized!`

### Step 2 — Plan the Deployment

Previews all the AWS resources that will be created, without making any changes:
```powershell
terraform plan
```

> Review the output carefully. It will show a list of resources to be **added**, **changed**, or **destroyed**.

### Step 3 — Apply the Deployment

Applies the changes and deploys the resources to AWS:
```powershell
terraform apply
```

When prompted:
```
Do you want to perform these actions? yes/no
```
Type `yes` and press `Enter` to confirm.

> Expected output: `Apply complete! Resources: X added, 0 changed, 0 destroyed.`

---

## 13. Complete DSPM Setup in the Lacework Portal

1. Open your browser and go to the Lacework portal:
   ```
   https://<your-account>.lacework.net
   ```

2. Navigate to **Settings** → **DSPM** from the left sidebar.

3. You will see your integration listed with a status of **"Setup Required"** — click on it.

4. Configure the scan settings:
   - **S3 Buckets** — select the S3 buckets you want to include or exclude from scanning
   - **Maximum File Size** — enter the highest file size (in MB) you want to scan
   - **Scan Frequency** — set how often you want scans to run (e.g. daily, weekly)

5. Click **Save**.

6. The integration status will update to **Active** — this confirms the integration is successful.

7. Verify the integration from the portal by checking that the status shows as **Active** under **Settings** → **DSPM**.

> **Note:** After the integration is active, initial scan results will appear in the portal within **2–3 hours**.

---

## Summary

| Tool          | macOS Command              | Windows Equivalent                                      |
|---------------|----------------------------|---------------------------------------------------------|
| AWS CLI       | `curl` + `.pkg` installer  | `Invoke-WebRequest` + `.msi` installer or `winget`      |
| Package Mgr   | `brew` (Homebrew)          | `choco` (Chocolatey) or `winget`                        |
| Git           | pre-installed / `brew`     | Git for Windows `.exe` installer, `winget`, or `choco`  |
| Terraform     | `brew install terraform`   | `choco install terraform` or `winget`                   |
| Lacework CLI  | `brew install lacework-cli`| `choco install lacework-cli` or manual download         |

---

> **Note:** After installing any tool via the manual PATH method, restart PowerShell or run `refreshenv` (if using Chocolatey) for the changes to take effect.
