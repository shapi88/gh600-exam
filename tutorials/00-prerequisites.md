# Lab 00: Prerequisites — One-Time Environment Setup

Complete these steps once before starting any lab. Every subsequent lab assumes this environment is in place.

---

## What you will set up

- A GitHub account and a practice repository
- GitHub CLI (`gh`) installed and authenticated
- Node.js and Python (for MCP examples in Lab 02)
- GitHub Actions enabled on the practice repo
- A fine-grained personal access token (PAT) with the minimum required scopes

---

## Step 1 — Create a GitHub account

If you already have a GitHub account, skip to Step 2.

1. Go to [https://github.com/signup](https://github.com/signup).
2. Enter a username, email address, and password.
3. Complete the email verification step.

---

## Step 2 — Create a practice repository

This repository is used for all labs. Name it `gh600-practice`.

**Via GitHub UI:**

1. Click the **+** icon in the top-right corner → **New repository**.
2. Set:
   - **Repository name:** `gh600-practice`
   - **Visibility:** Private (recommended) or Public
   - **Initialize this repository with:** check **Add a README file**
3. Click **Create repository**.

**Via GitHub CLI (after Step 3):**

```bash
gh repo create gh600-practice --private --add-readme --clone
cd gh600-practice
```

---

## Step 3 — Install and authenticate the GitHub CLI

### macOS

```bash
brew install gh
```

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install gh
```

### Windows (winget)

```powershell
winget install --id GitHub.cli
```

### Authenticate

```bash
gh auth login
# Select: GitHub.com → HTTPS → Authenticate with a web browser
# Follow the one-time code prompt in your browser
```

**Verify:**

```bash
gh auth status
# Expected output:
# ✓ Logged in to github.com as <your-username>
# ✓ Git operations for github.com configured to use https protocol
```

---

## Step 4 — Clone the practice repository locally

```bash
git clone https://github.com/<your-username>/gh600-practice.git
cd gh600-practice
```

---

## Step 5 — Install Node.js (for MCP examples in Lab 02)

### macOS / Linux (via nvm — recommended)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# Restart your terminal, then:
nvm install --lts
node --version   # expected: v20.x.x or higher
npm --version    # expected: 10.x.x or higher
```

### Windows

Download and run the installer from [https://nodejs.org/en/download](https://nodejs.org/en/download). Choose the **LTS** version.

---

## Step 6 — Install Python 3 (for MCP examples in Lab 02)

### macOS

```bash
brew install python
python3 --version   # expected: 3.11.x or higher
```

### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install python3 python3-pip
python3 --version
```

### Windows

Download from [https://www.python.org/downloads/](https://www.python.org/downloads/). Check **Add Python to PATH** during install.

---

## Step 7 — Enable GitHub Actions on the practice repo

GitHub Actions is enabled by default on new repositories. Confirm it is on:

1. Open `https://github.com/<your-username>/gh600-practice`.
2. Click the **Actions** tab.
3. If prompted with "Get started with GitHub Actions", Actions is enabled and you are ready.

If Actions was previously disabled:

1. Go to **Settings** → **Actions** → **General**.
2. Under **Actions permissions**, select **Allow all actions and reusable workflows**.
3. Click **Save**.

---

## Step 8 — Create a fine-grained personal access token

A fine-grained PAT lets you authenticate API calls in labs without exposing a broad token.

1. Go to [https://github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta).
2. Click **Generate new token**.
3. Set:
   - **Token name:** `gh600-labs`
   - **Expiration:** 30 days
   - **Resource owner:** your personal account
   - **Repository access:** Only select repositories → `gh600-practice`
4. Under **Permissions**, grant:

   | Permission | Level |
   | --- | --- |
   | Actions | Read and write |
   | Contents | Read and write |
   | Issues | Read and write |
   | Pull requests | Read and write |
   | Environments | Read and write |
   | Metadata | Read-only (required) |

5. Click **Generate token** and copy the token immediately — it is shown only once.
6. Store it in your shell for use in labs:

```bash
export GH_PAT="github_pat_<paste-your-token-here>"
```

> **Never commit this token to a repository.** If you accidentally expose it, go to your token settings and click **Regenerate** immediately.

---

## Step 9 — Configure git identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## Verification checklist

Run this block to confirm your environment is ready:

```bash
echo "--- gh CLI ---"
gh auth status

echo "--- git ---"
git --version

echo "--- Node.js ---"
node --version

echo "--- Python ---"
python3 --version

echo "--- Practice repo ---"
gh repo view gh600-practice --json name,visibility
```

Expected output (values will differ):

```
--- gh CLI ---
✓ Logged in to github.com as myusername
✓ Git operations for github.com configured to use https protocol
--- git ---
git version 2.43.0
--- Node.js ---
v20.12.0
--- Python ---
Python 3.12.0
--- Practice repo ---
{
  "name": "gh600-practice",
  "visibility": "PRIVATE"
}
```

---

## Next step

All prerequisites are complete. Start with [Lab 01 — Agent Architecture & SDLC](01-agent-architecture-lab.md).
