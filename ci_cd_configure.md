# GitHub Actions CI/CD Pipeline - Complete Documentation

## Table of Contents
1. [Overview](#overview)
2. [Challenges Faced](#challenges-faced)
3. [Core Concepts & Root Causes](#core-concepts--root-causes)
4. [Solutions Implemented](#solutions-implemented)
5. [Complete Setup Flow](#complete-setup-flow)
6. [Architecture & Component Behavior](#architecture--component-behavior)
7. [Workflow Explanation](#workflow-explanation)
8. [Troubleshooting Guide](#troubleshooting-guide)

---

## Overview

This document explains the complete GitHub Actions CI/CD pipeline setup for the **Zodiack Backend** project. It covers automated building, testing, and deployment to an EC2 server with PM2 process management.

**Project**: Zodiack Backend (Node.js + Express)  
**Deployment Target**: AWS EC2 (Ubuntu)  
**Process Manager**: PM2  
**Build Tool**: npm  
**Node Version**: v20.x

---

## Challenges Faced

### Challenge 1: "Too many levels of symbolic links" Error
```
bash: line 12: /usr/bin/npm: Too many levels of symbolic links
```

**When it occurred**: During the build step in GitHub Actions

**What happened**: The workflow tried to run `/usr/bin/npm install`, but this path was a broken symlink chain that never resolved to the actual npm executable.

---

### Challenge 2: "npm: command not found" Error
```
bash: line 12: npm: command not found
```

**When it occurred**: During the deployment step on EC2 server via SSH

**What happened**: Even after sourcing NVM, npm wasn't available in the SSH session because GitHub Actions uses non-interactive shells that don't load shell profiles.

---

### Challenge 3: PM2 Process Not Found
```
[ERROR] Process or Namespace zodiack-backend not found
```

**When it occurred**: After npm build failures

**What happened**: PM2 tried to reload a process that didn't exist because the previous build step failed, so the app was never deployed.

---

### Challenge 4: Script Not Found
```
Script not found: /home/ubuntu/zodiack-backend/dist/index.js
```

**When it occurred**: When PM2 tried to start the compiled JavaScript file

**What happened**: The TypeScript build never completed (due to npm error), so `dist/index.js` was never generated, but PM2 tried to start it anyway.

---

## Core Concepts & Root Causes

### 1. Symlink Issues (Challenge 1)

**Core Concept: What is a Symlink?**

A symlink (symbolic link) is a special file that acts as a shortcut to another file:

```
/usr/bin/npm → /some/path/npm → /another/path/npm → actual_npm_binary
```

**Why the problem occurred:**
- When Node.js was installed via NVM, it created symlinks in `/usr/bin/npm`
- These symlinks pointed to other symlinks (instead of directly to the binary)
- This created an indirect reference chain
- The system followed the chain until it hit a maximum limit, then gave up
- Result: "too many levels of symbolic links"

**Diagram:**
```
Direct (Good):
/usr/bin/npm → /home/user/.nvm/versions/node/v20/bin/npm ✓

Indirect (Bad):
/usr/bin/npm → /usr/local/bin/npm → /home/user/.nvm/versions/node/v20/bin/npm → ... → npm ✗
```

---

### 2. Non-Interactive Shell Problem (Challenge 2)

**Core Concept: Interactive vs Non-Interactive Shells**

When you SSH into a server normally:
```bash
ssh user@host
```

Your shell is **interactive** and loads startup files:
- `~/.bashrc` (bash runtime configuration)
- `~/.bash_profile` (bash login configuration)

These files contain NVM initialization code:
```bash
# Inside ~/.bashrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # Load NVM
```

However, GitHub Actions runs SSH commands as **non-interactive** shell sessions:
```yaml
uses: appleboy/ssh-action@v1.0.3
```

**What happens:**
- Non-interactive shell doesn't load `~/.bashrc` or `~/.bash_profile`
- NVM initialization code never runs
- `npm` command is never added to PATH
- Result: "npm: command not found" even though it's installed

**Comparison:**

```
Interactive Shell (Manual SSH):
1. User connects via SSH
2. Shell loads ~/.bashrc
3. NVM code runs: export PATH=$PATH:~/.nvm/versions/node/v20/bin
4. npm is in PATH ✓
5. npm --version works ✓

Non-Interactive Shell (GitHub Actions):
1. Action runs SSH command
2. Shell doesn't load ~/.bashrc
3. NVM code doesn't run
4. npm NOT in PATH ✗
5. npm --version fails ✗
```

---

### 3. Cascading Failure Problem (Challenges 3 & 4)

**Core Concept: Dependent Jobs & Error Propagation**

When one step fails, dependent steps also fail:

```yaml
steps:
  - run: npm install    # Step 1: FAILS
  - run: npm run build  # Step 2: Skipped/Fails
  - run: pm2 reload     # Step 3: Fails because dist/ doesn't exist
```

**Cascade:**
```
npm install FAILS (no npm in PATH)
    ↓
npm run build NEVER RUNS
    ↓
dist/index.js NEVER CREATED
    ↓
pm2 reload FAILS (nothing to reload)
    ↓
pm2 start FAILS (dist/index.js doesn't exist)
```

---

### 4. Environment PATH Concept

**Core Concept: What is PATH?**

PATH is an environment variable that tells the system where to look for executable commands:

```bash
echo $PATH
# Output: /usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin
```

When you type `npm`, the system searches these directories in order:
```
1. /usr/local/bin/npm   ← Found here in our solution!
2. /usr/bin/npm
3. /bin/npm
4. ... (other directories)
```

**Solution: Create symlinks in /usr/local/bin/**

Since `/usr/local/bin` is always in PATH (even for non-interactive shells), creating symlinks there makes npm available everywhere:

```bash
# This works for ANY shell type
sudo ln -sf $(which npm) /usr/local/bin/npm
```

---

## Solutions Implemented

### Solution 1: Use /usr/local/bin Symlinks

**The Fix:**
```bash
# Create symlinks in a directory that's always in PATH
sudo ln -sf $(which node) /usr/local/bin/node
sudo ln -sf $(which npm) /usr/local/bin/npm
sudo ln -sf $(which pm2) /usr/local/bin/pm2
```

**Why it works:**
- `/usr/local/bin` is in PATH for ALL shell types (interactive and non-interactive)
- These are direct symlinks, not chains
- GitHub Actions non-interactive shells can still find npm

**Verification:**
```bash
# Should work in non-interactive shells too
/usr/local/bin/npm --version
```

---

### Solution 2: Install Node.js Properly via NVM

**Before (Missing):**
```
❌ Node.js not installed
❌ npm not available
```

**After (Installation Steps):**
```bash
# Step 1: Install NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# Step 2: Install specific Node.js version
nvm install 20

# Step 3: Create system-wide symlinks
sudo ln -sf $(which node) /usr/local/bin/node
sudo ln -sf $(which npm) /usr/local/bin/npm

# Step 4: Install PM2 globally
npm install -g pm2
sudo ln -sf $(which pm2) /usr/local/bin/pm2

# Step 5: Setup your application
cd /home/ubuntu/zodiack-backend
npm install
npm run build
pm2 start dist/index.js --name "zodiack-backend"
pm2 save
```

**What each step does:**
1. **NVM Installation**: Installs Node Version Manager (allows multiple Node.js versions)
2. **Node.js v20 Installation**: Installs Node.js v20 via NVM
3. **Symlinks Creation**: Makes node/npm/pm2 available in all shells
4. **PM2 Installation**: Installs process manager for production apps
5. **App Setup**: Installs dependencies, builds app, starts with PM2

---

### Solution 3: Simplified GitHub Actions Workflow

**Before (Complex & Failed):**
```yaml
script: |
  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
  /usr/bin/npm install  # ❌ Broken symlink
  /usr/bin/npm run build
```

**After (Simple & Works):**
```yaml
script: |
  cd /home/ubuntu/zodiack-backend
  git pull origin main
  npm install            # ✓ Uses /usr/local/bin/npm
  npm run build
  pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
  pm2 save
```

**Why the simplified version works:**
- No path manipulation needed
- npm is directly available (via /usr/local/bin symlink)
- Straightforward, easy to debug

---

## Complete Setup Flow

### Phase 1: Server Preparation (One-time)

```
┌─────────────────────────────────────────┐
│ AWS EC2 Instance (Ubuntu)               │
│ (Fresh, no Node.js)                     │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ 1. Install NVM (Node Version Manager)   │
│    - Allows managing multiple Node      │
│    - Non-invasive installation          │
│    - Stores in ~/.nvm directory         │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ 2. Install Node.js v20 via NVM          │
│    - Located at ~/.nvm/versions/node/v20│
│    - npm comes included with Node.js    │
│    - Contains build tools needed        │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ 3. Create System-Wide Symlinks          │
│    - /usr/local/bin/node → actual node  │
│    - /usr/local/bin/npm → actual npm    │
│    - Makes available to all shells      │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ 4. Install PM2 (Process Manager)        │
│    - Symlink /usr/local/bin/pm2         │
│    - Manages app lifecycle              │
│    - Handles restarts/crashes           │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ 5. Setup Application                    │
│    - Clone/setup code repository        │
│    - npm install dependencies           │
│    - npm run build (TypeScript → JS)    │
│    - pm2 start application              │
│    - pm2 save state for persistence     │
└─────────────────────────────────────────┘
```

### Phase 2: GitHub Actions Triggered (Every Push to main)

```
┌─────────────────────────────────────────┐
│ Developer pushes code to main branch     │
└─────────────────────┬───────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│ STEP 1: BUILD JOB (GitHub Runner)       │
│                                         │
│ - Checkout code                         │
│ - Setup Node.js v20                     │
│ - Run: npm ci (clean install)           │
│ - Run: npm run build                    │
│ - Set environment variables             │
│   (NODE_ENV, DATABASE_URL, etc.)        │
│                                         │
│ Output: dist/ folder with compiled JS   │
└─────────────────────┬───────────────────┘
                      │
        ✓ Build succeeds?
                      │
          ┌───────────┴───────────┐
          │                       │
         YES                     NO
          │                       │
          ▼                       ▼
    Continue            ❌ Stop (notify dev)
          │
          ▼
┌─────────────────────────────────────────┐
│ STEP 2: DEPLOY JOB (Runs after build)   │
│                                         │
│ Prerequisites:                          │
│ - Build job succeeded                   │
│ - SSH keys configured in GitHub Secrets │
│ - EC2 security group allows SSH (22)    │
│                                         │
│ Actions:                                │
│ 1. SSH into EC2 instance                │
│ 2. Navigate to project folder           │
│ 3. git pull origin main                 │
│ 4. npm install (update deps)            │
│ 5. npm run build (rebuild)              │
│ 6. pm2 reload OR pm2 start              │
│ 7. pm2 save (persist state)             │
│                                         │
│ Output: Application running on EC2      │
└─────────────────────┬───────────────────┘
                      │
          ✓ Deploy succeeds?
                      │
          ┌───────────┴────────────┐
          │                        │
         YES                      NO
          │                        │
          ▼                        ▼
    ✓ Deployment            ❌ SSH failed
      complete               (check logs)
          │
          ▼
┌─────────────────────────────────────────┐
│ Application updated & running on EC2    │
│ Server accessible at configured URL     │
│ PM2 will auto-restart on failures       │
└─────────────────────────────────────────┘
```

---

## Architecture & Component Behavior

### Component 1: GitHub Actions Runner

**What it is:**
- Virtual machine hosted by GitHub
- Runs your CI/CD workflow
- Only exists for the duration of the workflow

**Behavior:**
```yaml
runs-on: ubuntu-latest  # Latest Ubuntu image, fresh each time
```

**Life cycle:**
```
1. Spin up new Ubuntu VM
2. Run workflow steps
3. Upload build artifacts (if needed)
4. Tear down VM
5. Next run gets fresh VM
```

**What it does:**
```
Build Job (On GitHub Runner):
├── Step 1: Checkout code from repository
├── Step 2: Install Node.js v20
├── Step 3: Run npm ci (clean install - removes old node_modules)
├── Step 4: Run npm run build (TypeScript → JavaScript compilation)
│   └── Reads: tsconfig.json, src/*.ts
│   └── Outputs: dist/index.js, dist/**/*.js
└── Step 5: Complete build (artifacts stay on GitHub)
```

---

### Component 2: Build Job

**Purpose:** Compile TypeScript to JavaScript, validate code

**Environment Variables:**
```yaml
env:
  NODE_ENV: ${{ secrets.NODE_ENV }}        # "production"
  DATABASE_URL: ${{ secrets.DATABASE_URL }} # Your database connection string
  JWT_SECRET_KEY: ${{ secrets.JWT_SECRET_KEY }}
  # ... more 15+ secret variables
```

**Why so many env vars:**
- Your application reads these during build time
- They're used to configure the app (database, auth keys, API keys)
- Secrets are never stored in code, only in GitHub Secrets

**Build process:**
```
Input Files:
├── src/
│   ├── index.ts
│   ├── routes/
│   ├── controllers/
│   └── middleware/
├── tsconfig.json
├── package.json
└── .env.example

Processing:
├── Load environment variables
├── TypeScript compiler (tsc)
│   ├── Check types
│   ├── Compile to JavaScript
│   └── Generate sourcemaps
├── Minification (optional)
└── Output files

Output Files:
└── dist/
    ├── index.js
    ├── routes/
    ├── controllers/
    └── middleware/
```

---

### Component 3: Deploy Job

**Purpose:** Push code to EC2 and restart application

**Dependencies:**
```yaml
needs: build  # Only runs if build succeeds
```

**SSH Connection Flow:**
```
GitHub Actions          EC2 Server
    │                      │
    ├──── SSH KEY ────────→ │ (authenticate)
    │                      │
    ├──── SCRIPT ─────────→ │ (execute commands)
    │                      │
    │ ← ──── STDOUT ─────── │ (console output)
    │ ← ──── STDERR ─────── │ (error output)
    │                      │
    └──── CLOSE ─────────→ │ (disconnect)
```

**EC2 Execution Steps:**
```bash
# Step 1: Navigate to project
cd /home/ubuntu/zodiack-backend

# Step 2: Get latest code
git pull origin main
# Downloads: latest .github/workflows/ci.yml, src files, etc.

# Step 3: Install dependencies
npm install
# Reads: package.json
# Installs: node_modules/ (fresh copy)
# Creates: package-lock.json

# Step 4: Build application
npm run build
# Reads: tsconfig.json, src/
# Outputs: dist/

# Step 5: Update PM2 process
pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
# Logic: 
#   - Try to reload existing process
#   - If doesn't exist, start new process
#   - Seamless restart (0 downtime if already running)

# Step 6: Persist PM2 state
pm2 save
# Saves: /home/user/.pm2/dump.pm2
# So app restarts on server reboot
```

---

### Component 4: PM2 (Process Manager)

**What is PM2?**

PM2 is a production process manager for Node.js applications:

```
PM2's Job:
│
├── START: Start your application
├── MONITOR: Watch if app crashes
├── RESTART: Automatically restart if it dies
├── LOGS: Collect application logs
├── CLUSTER: Run on multiple CPU cores (optional)
└── PERSIST: Remember app even after server restart
```

**Why PM2?**

Without PM2:
```bash
# Manual approach (BAD for production):
node dist/index.js

# Problems:
# - App runs in foreground
# - If app crashes, it stays down
# - Can't easily manage (stop/restart)
# - Logs scattered everywhere
# - Can't run multiple instances
```

With PM2:
```bash
# Production approach (GOOD):
pm2 start dist/index.js --name "zodiack-backend"

# Benefits:
# - Runs in background
# - Auto-restarts on crash
# - Easy commands (pm2 stop/restart/logs)
# - Centralized logging
# - Can run multiple instances
```

**PM2 Key Commands:**
```bash
pm2 start dist/index.js --name "app"     # Start app
pm2 stop app                             # Stop app
pm2 restart app                          # Restart app
pm2 reload app                           # Reload (0 downtime)
pm2 logs app                             # View logs
pm2 list                                 # See all apps
pm2 save                                 # Save current state
pm2 startup                              # Start on boot
pm2 delete app                           # Remove app
```

**PM2 State Persistence:**
```
Without pm2 save:
├── Server restarts
└── Application is DOWN ❌

With pm2 save:
├── Server restarts
├── PM2 daemon loads /home/user/.pm2/dump.pm2
├── Application auto-starts
└── Application is UP ✓
```

---

### Component 5: Environment Variables & Secrets

**What are secrets?**

Sensitive data that shouldn't be in code:
```
❌ BAD (in code):
const DB_URL = "mongodb://user:password@localhost";

✓ GOOD (as secret):
const DB_URL = process.env.DATABASE_URL;
// VALUE stored in GitHub Secrets, not in code
```

**How secrets flow:**

```
GitHub Secrets (web UI)
    ↓
.github/workflows/ci.yml
    ↓
env: section (passed to runners)
    ↓
Build job reads process.env.VARIABLE_NAME
    ↓
Application receives configuration
```

**Example:**
```yaml
# In .github/workflows/ci.yml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}

# In your Node.js code:
const dbUrl = process.env.DATABASE_URL;
// Gets value from GitHub Secrets securely
```

---

## Workflow Explanation

Here's the complete workflow file with detailed comments:

```yaml
# GitHub Actions Workflow File
# Defines automated CI/CD pipeline

name: CI/CD Pipeline
# Human-readable name shown in GitHub UI

on:
  push:
    branches: [main]
# Trigger: Whenever code is pushed to main branch

jobs:
  # ===== BUILD JOB =====
  build:
    # Job name (shown in GitHub UI)
    
    runs-on: ubuntu-latest
    # Use latest Ubuntu image for this job
    # (Fresh VM, no dependencies from previous runs)
    
    steps:
      # Each step runs sequentially
      # Job fails if any step fails
    
      - uses: actions/checkout@v4
        # ACTION: Clone repository code into runner
        # Why: Need source code to build
        # Output: Code available in $GITHUB_WORKSPACE
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          # ACTION: Install Node.js v20 on runner
          # Why: Needed to run npm and build app
          # Installs: node, npm, npx commands
          
          cache: npm
          # OPTIMIZATION: Cache npm packages
          # Speeds up: npm install (reuses old node_modules)
      
      - run: npm ci
        # COMMAND: Clean Install dependencies
        # npm ci = npm Clean Install (for CI/CD, more reliable than npm install)
        # What it does:
        #   - Reads package.json
        #   - Downloads exact versions from package-lock.json
        #   - Creates node_modules/ folder
        #   - Installs all dependencies
        # Why npm ci instead of npm install?
        #   - npm ci: Deterministic, reproducible
        #   - npm install: Can update versions (avoid in CI)
      
      - run: npm run build
        # COMMAND: Build/Compile application
        # Runs: "build" script from package.json
        # Typically runs TypeScript compiler:
        #   tsc (TypeScript → JavaScript)
        # Input: src/**/*.ts files
        # Output: dist/ folder with JavaScript
        
        env:
          # Environment variables available during build
          # Build script can read these via process.env
          
          NODE_ENV: ${{ secrets.NODE_ENV }}
          # Your app's environment mode
          # Value: Usually "production"
          # Usage: if (process.env.NODE_ENV === 'production') { ... }
          
          PORT: ${{ secrets.PORT }}
          # Server port to listen on
          # Value: Usually 5000 or 8000
          # Usage: app.listen(process.env.PORT)
          
          LOCAL_URL: ${{ secrets.LOCAL_URL }}
          # URL for local/backend connections
          # Value: "http://localhost:5000"
          
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          # MongoDB or database connection string
          # Value: "mongodb+srv://user:pass@cluster.mongodb.net/db"
          # Usage: mongoose.connect(process.env.DATABASE_URL)
          
          JWT_SECRET_KEY: ${{ secrets.JWT_SECRET_KEY }}
          # Secret key for signing JWT tokens
          # Value: Long random string (keep secure!)
          # Usage: jwt.sign(payload, process.env.JWT_SECRET_KEY)
          
          JWT_REFRESH_KEY: ${{ secrets.JWT_REFRESH_KEY }}
          # Secret key for refresh tokens
          # Value: Different long random string
          # Usage: jwt.sign(payload, process.env.JWT_REFRESH_KEY)
          
          JWT_ACCESS_EXPIRES_IN: ${{ secrets.JWT_ACCESS_EXPIRES_IN }}
          # How long access tokens last
          # Value: "15m" or "900" (seconds)
          # Usage: { expiresIn: process.env.JWT_ACCESS_EXPIRES_IN }
          
          JWT_ACCESS_EXPIRES_IN_FOR_TEACHER: ${{ secrets.JWT_ACCESS_EXPIRES_IN_FOR_TEACHER }}
          # Different expiry time for teacher tokens
          # Value: "24h" or "86400" (seconds)
          
          JWT_REFRESH_EXPIRES_IN_FOR_TEACHER: ${{ secrets.JWT_REFRESH_EXPIRES_IN_FOR_TEACHER }}
          # Refresh token expiry for teachers
          # Value: "7d" or "604800" (seconds)
          
          CLOUDINARY_NAME: ${{ secrets.CLOUDINARY_NAME }}
          # Cloudinary account name (image hosting)
          # Value: "your-cloudinary-name"
          # Usage: cloudinary.v2.config({ cloud_name: ... })
          
          CLOUDINARY_API_KEY: ${{ secrets.CLOUDINARY_API_KEY }}
          # API key for Cloudinary authentication
          # Value: "1234567890..." (from Cloudinary dashboard)
          # Usage: { api_key: process.env.CLOUDINARY_API_KEY }
          
          CLOUDINARY_API_SECRET: ${{ secrets.CLOUDINARY_API_SECRET }}
          # Secret for Cloudinary authentication
          # Value: Long secure string
          # Usage: { api_secret: process.env.CLOUDINARY_API_SECRET }
          
          Nodemailer_GMAIL: ${{ secrets.Nodemailer_GMAIL }}
          # Gmail account for sending emails
          # Value: "your-email@gmail.com"
          # Usage: nodemailer.createTransport({ user: ... })
          
          Nodemailer_GMAIL_PASSWORD: ${{ secrets.Nodemailer_GMAIL_PASSWORD }}
          # Gmail app password for authentication
          # Value: 16-character password from Gmail
          # Usage: nodemailer.createTransport({ pass: ... })
          
          UPLOAD_FOLDER: ${{ secrets.UPLOAD_FOLDER }}
          # Local folder for file uploads
          # Value: "./uploads" or "/var/uploads"
          # Usage: fs.mkdir(process.env.UPLOAD_FOLDER)
          
          FIREBASE_TYPE: ${{ secrets.FIREBASE_TYPE }}
          FIREBASE_PROJECT_ID: ${{ secrets.FIREBASE_PROJECT_ID }}
          FIREBASE_PRIVATE_KEY_ID: ${{ secrets.FIREBASE_PRIVATE_KEY_ID }}
          FIREBASE_PRIVATE_KEY: ${{ secrets.FIREBASE_PRIVATE_KEY }}
          FIREBASE_CLIENT_EMAIL: ${{ secrets.FIREBASE_CLIENT_EMAIL }}
          FIREBASE_CLIENT_ID: ${{ secrets.FIREBASE_CLIENT_ID }}
          FIREBASE_AUTH_URI: ${{ secrets.FIREBASE_AUTH_URI }}
          FIREBASE_TOKEN_URI: ${{ secrets.FIREBASE_TOKEN_URI }}
          FIREBASE_AUTH_PROVIDER_CERT_URL: ${{ secrets.FIREBASE_AUTH_PROVIDER_CERT_URL }}
          FIREBASE_CLIENT_CERT_URL: ${{ secrets.FIREBASE_CLIENT_CERT_URL }}
          FIREBASE_UNIVERSE_DOMAIN: ${{ secrets.FIREBASE_UNIVERSE_DOMAIN }}
          # All Firebase service account credentials
          # What it's for: Firebase authentication, database, storage
          # These are from your Firebase project's service account JSON file
          # Usage: firebase.initializeApp({ ... all these vars ... })

  # ===== DEPLOY JOB =====
  deploy:
    # Second job that runs AFTER build succeeds
    
    needs: build
    # DEPENDENCY: Only run if "build" job succeeded
    # If build fails, deploy is skipped
    
    runs-on: ubuntu-latest
    # Use fresh Ubuntu VM (separate from build job)
    
    steps:
      - name: Deploy to Server via SSH
        # Friendly name for this step
        
        uses: appleboy/ssh-action@v1.0.3
        # ACTION: SSH into remote server and run commands
        # This is a third-party action from the GitHub marketplace
        
        with:
          host: ${{ secrets.EC2_HOST }}
          # Remote server IP or hostname
          # Value: "1.2.3.4" or "your-server.com"
          # This is the EC2 public IP
          
          username: ${{ secrets.EC2_USER }}
          # SSH username to login as
          # Value: "ubuntu" (default EC2 user)
          
          key: ${{ secrets.EC2_SSH_KEY }}
          # SSH private key for authentication
          # Value: Content of your .pem file
          # Generated when you created EC2 instance
          # Keep this SECRET - never commit to repo!
          
          script: |
            # Commands to run ON THE EC2 SERVER
            # NOT on GitHub runner
            # Each line executes sequentially
            # If any command fails, remaining commands skip
            
            # Navigate to project directory
            cd /home/ubuntu/zodiack-backend
            # Why: Need to be in project folder to run git/npm
            
            # Get latest code from GitHub
            git pull origin main
            # What: Downloads latest commits from main branch
            # Updates: All files changed since last deployment
            # Skips: If already up-to-date
            
            # Install/update dependencies
            npm install
            # What: Reads package.json, installs all dependencies
            # Creates: node_modules/ folder
            # Updates: If package.json changed
            
            # Compile TypeScript to JavaScript
            npm run build
            # What: Runs build script from package.json
            # Input: src/ files
            # Output: dist/ folder with compiled JavaScript
            
            # Reload PM2 process (or start if doesn't exist)
            pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
            # What: Updates running app without downtime
            # pm2 reload: Graceful restart (if already running)
            # pm2 start: Start fresh process (if not running)
            # Logic: Try reload first, fall back to start if fails
            # Result: App updated and running
            
            # Save PM2 process list to disk
            pm2 save
            # What: Persists app process list
            # File: /home/ubuntu/.pm2/dump.pm2
            # Why: App will auto-start if server reboots
            # Without this: App stays down after reboot
```

---

## Troubleshooting Guide

### Issue 1: Build Fails with Environment Variable Errors

**Error Message:**
```
Error: process.env.DATABASE_URL is undefined
```

**Root Cause:**
- Environment variable not added to GitHub Secrets
- Typo in secret name
- Incorrect syntax in env section

**Solution:**
```
1. Go to GitHub repo → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: DATABASE_URL
4. Value: your_actual_database_url
5. Click "Add secret"
```

---

### Issue 2: Deploy Fails - SSH Connection Refused

**Error Message:**
```
Connection refused at 1.2.3.4:22
```

**Root Cause:**
- EC2 security group doesn't allow SSH (port 22)
- Wrong EC2 IP address
- EC2 instance is stopped

**Solution:**
```
1. Go to AWS Console → EC2 → Security Groups
2. Find your security group
3. Edit Inbound Rules
4. Add rule:
   - Type: SSH
   - Port: 22
   - Source: 0.0.0.0/0 (or your GitHub IP)
5. Save
```

---

### Issue 3: npm: command not found on EC2

**Error Message:**
```
bash: npm: command not found
```

**Root Cause:**
- Node.js/npm not installed on EC2
- Symlinks not created

**Solution:**
```bash
# SSH into EC2
ssh -i your-key.pem ubuntu@your-ec2-ip

# Install Node.js via NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 20

# Create symlinks
sudo ln -sf $(which npm) /usr/local/bin/npm
sudo ln -sf $(which node) /usr/local/bin/node
sudo ln -sf $(which pm2) /usr/local/bin/pm2

# Verify
npm --version
```

---

### Issue 4: PM2 Process Not Found

**Error Message:**
```
[ERROR] Process or Namespace zodiack-backend not found
```

**Root Cause:**
- First deployment (process doesn't exist yet)
- app previously deleted

**Solution:**
This is actually OK! The workflow has a fallback:
```bash
pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
#                            ^^ Fallback to start if reload fails
```

The `||` (OR) operator runs the second command only if first fails.

---

### Issue 5: Permission Denied for Symlinks

**Error Message:**
```
permission denied
```

**Root Cause:**
- Missing `sudo` for system-wide symlinks

**Solution:**
```bash
# DO use sudo for /usr/local/bin
sudo ln -sf $(which npm) /usr/local/bin/npm

# NOT needed for user directories
ln -sf $(which npm) ~/.local/bin/npm
```

---

### Issue 6: Workflow Won't Trigger

**Error Message:**
- Workflow doesn't appear in Actions tab
- No automatic deployment when pushing to main

**Root Cause:**
- Workflow file syntax error
- File not in `.github/workflows/` directory
- Not pushed to repository

**Solution:**
```
1. Ensure file is: .github/workflows/ci.yml
2. Validate YAML syntax (indentation matters!)
3. Commit and push: git push origin main
4. Check GitHub repo → Actions tab
```

---

## Quick Reference

### File Structure
```
zodiack-backend/
├── .github/
│   └── workflows/
│       └── ci.yml           # GitHub Actions workflow
├── src/
│   ├── index.ts             # Entry point
│   ├── routes/
│   ├── controllers/
│   └── middleware/
├── dist/                    # Generated by build (not committed)
│   ├── index.js
│   └── ...
├── node_modules/            # Generated by npm install (not committed)
├── package.json             # Project dependencies
├── package-lock.json        # Exact versions used
├── tsconfig.json            # TypeScript configuration
└── .gitignore               # Files to ignore in git
```

### Key Commands
```bash
# Local development
npm install              # Install dependencies
npm run build           # Compile TypeScript
npm start               # Run application

# On EC2 server
pm2 list                # See all processes
pm2 logs app-name       # View application logs
pm2 stop app-name       # Stop app
pm2 restart app-name    # Restart app
pm2 delete app-name     # Remove app
```

### GitHub Secrets Needed
```
- EC2_HOST (EC2 public IP)
- EC2_USER (usually "ubuntu")
- EC2_SSH_KEY (private key content)
- DATABASE_URL
- JWT_SECRET_KEY
- JWT_REFRESH_KEY
- NODE_ENV
- PORT
- CLOUDINARY_NAME
- CLOUDINARY_API_KEY
- CLOUDINARY_API_SECRET
- Nodemailer_GMAIL
- Nodemailer_GMAIL_PASSWORD
- FIREBASE_TYPE
- FIREBASE_PROJECT_ID
- (+ other Firebase config)
```

---

## Deployment Checklist

Before each deployment:

- [ ] Code changes committed and pushed to `main` branch
- [ ] All secrets configured in GitHub
- [ ] EC2 instance running
- [ ] Security group allows SSH (port 22)
- [ ] Node.js installed on EC2 (nvm install 20)
- [ ] Symlinks created (/usr/local/bin/npm, etc.)
- [ ] PM2 installed globally
- [ ] Project cloned on EC2
- [ ] Database is running and accessible
- [ ] Environment variables correct

---

## Summary

| Component | Purpose | Status |
|-----------|---------|--------|
| GitHub Actions | CI/CD automation | Running ✓ |
| Build Job | Compile TypeScript | Working ✓ |
| Deploy Job | Update EC2 app | Working ✓ |
| Node.js/npm | Runtime & package manager | Installed ✓ |
| PM2 | Process management | Running ✓ |
| Symlinks | Make npm available to all shells | Created ✓ |
| Secrets | Secure configuration | Configured ✓ |
| EC2 Server | Production environment | Ready ✓ |

Your CI/CD pipeline is fully functional and ready for continuous deployment!