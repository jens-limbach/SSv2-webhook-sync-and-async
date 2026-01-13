# Project Setup Guide

## Initial Git Repository Setup

### 1. Initialize Git Repository

```bash
cd "C:\Users\D057136\VSCode Projects\crm-webhook-service"
git init
```

### 2. Stage All Files

```bash
git add .
```

### 3. Create Initial Commit

```bash
git commit -m "Initial commit: CRM Webhook Service with sync/async patterns"
```

### 4. Add Remote Repository

```bash
git remote add origin <your-repository-url>
```

### 5. Push to Remote

```bash
git push -u origin main
```

## What Gets Committed

✅ **Included in repository:**
- `server.js` - Main application code
- `package.json` - Dependencies and scripts
- `package-lock.json` - Locked dependency versions
- `README.md` - Complete documentation
- `SETUP.md` - This setup guide
- `env-template.txt` - Environment variable template
- `manifest.yml` - Cloud Foundry deployment config
- `.gitignore` - Git ignore rules

❌ **Excluded from repository:**
- `.env` - Your credentials (never commit!)
- `node_modules/` - Dependencies (install with npm)
- `.vscode/` - IDE settings
- Log files

## Cloning on a New Machine

### 1. Clone Repository

```bash
git clone <your-repository-url>
cd crm-webhook-service
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment

```bash
# Windows
copy env-template.txt .env

# macOS/Linux
cp env-template.txt .env
```

Edit `.env` with your credentials:
```env
CRM_BASE_URL=https://your-tenant.crm.cloud.sap
CRM_USERNAME=your-username
CRM_PASSWORD=your-password
PORT=3000
```

### 4. Run Locally

```bash
npm run dev
```

### 5. Deploy to Cloud Foundry

```bash
# Login to Cloud Foundry
cf login

# Set environment variables
cf set-env crm-webhook-service CRM_BASE_URL "https://your-tenant.crm.cloud.sap"
cf set-env crm-webhook-service CRM_USERNAME "your-username"
cf set-env crm-webhook-service CRM_PASSWORD "your-password"

# Deploy
cf push

# Verify
cf app crm-webhook-service
```

## Security Checklist

Before pushing to repository:

- [ ] `.env` file is listed in `.gitignore`
- [ ] No credentials in code or configuration files
- [ ] `env-template.txt` contains placeholders only
- [ ] `README.md` doesn't contain real credentials
- [ ] Test that `.env` is not staged: `git status`

## Useful Git Commands

```bash
# Check what will be committed
git status

# See changes
git diff

# Undo changes to a file
git checkout -- <file>

# Create a new branch
git checkout -b feature/new-feature

# View commit history
git log --oneline

# Update from remote
git pull origin main
```
