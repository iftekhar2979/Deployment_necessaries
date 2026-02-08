# 🚀 GitHub Actions: A Complete Guide for Beginners

## What is GitHub Actions? (Simple Explanation)

Imagine you have a homework assignment:

📝 **Without Automation (Normal Way):**
1. You write the code
2. You test it manually
3. You check for errors manually
4. You compile it
5. You upload it to the server
6. You restart the server
7. You hope nothing breaks

😫 That's a lot of manual work!

---

**With Automation (GitHub Actions Way):**
1. You write the code
2. You push to GitHub
3. **AUTOMATIC** ⚡ GitHub does everything else automatically:
   - Tests the code
   - Compiles the code
   - Uploads to server
   - Restarts the app

🎉 You just watch it work!

---

## Real-World Analogy 🏭

Think of GitHub Actions like an **automated factory assembly line**:

```
Your Code Repository (Raw Materials)
           ↓
          🤖 ROBOT 1: Test the code (QA check)
           ↓
          🤖 ROBOT 2: Compile the code (Processing)
           ↓
          🤖 ROBOT 3: Upload to server (Packaging)
           ↓
          🤖 ROBOT 4: Start the app (Delivery)
           ↓
      Your Live Website/App 🌐
```

Without robots, YOU do all that work. With robots (GitHub Actions), the work happens automatically! ⚡

---

## Core Concepts Explained Simply

### 1️⃣ **TRIGGER** - What Starts Everything?

A **trigger** is like a button that starts the whole process.

```yaml
on:
  push:
    branches: [main]
```

**In Simple Words:** "When someone pushes code to the main branch, start the process"

**Real-Life Example:**
```
❌ WITHOUT TRIGGER:
Your code sits there... nothing happens

✓ WITH TRIGGER:
Someone: "I'm pushing code!"
GitHub: "New code detected! Starting automation..." 🚀
```

**Common Triggers:**
```yaml
# Trigger 1: When code is pushed
on:
  push:
    branches: [main]

# Trigger 2: When pull request is created
on:
  pull_request:
    branches: [main]

# Trigger 3: On a schedule (like a scheduled task)
on:
  schedule:
    - cron: '0 9 * * *'  # Every day at 9 AM
```

---

### 2️⃣ **JOB** - What Gets Done?

A **job** is a set of tasks that run together.

```yaml
jobs:
  build:
    # This is ONE job called "build"
    # It contains multiple steps
```

**In Simple Words:** A job is like a checklist that must be completed.

**Example Job: "Make Breakfast"**
```yaml
jobs:
  make_breakfast:
    steps:
      - Take eggs from fridge        # Step 1
      - Heat the pan                 # Step 2
      - Cook the eggs                # Step 3
      - Put on plate                 # Step 4
```

---

### 3️⃣ **STEP** - Individual Actions

A **step** is ONE small action.

```yaml
steps:
  - run: npm install        # Step 1: Install packages
  - run: npm run build      # Step 2: Build the app
```

**In Simple Words:** Steps are like small tasks within a bigger job.

**Example Steps in Your Project:**
```yaml
steps:
  1. Download code from GitHub
  2. Install Node.js
  3. Install packages with npm
  4. Build the TypeScript code
  5. Connect to server with SSH
  6. Upload the app
  7. Restart the app
```

---

### 4️⃣ **RUNNER** - The Computer That Does The Work

A **runner** is a computer (provided by GitHub) that runs your automation.

```yaml
runs-on: ubuntu-latest
```

**In Simple Words:** GitHub gives you a temporary computer to run your automation.

**How it Works:**
```
GitHub has thousands of computers (servers) sitting waiting...

When you push code:
1. GitHub assigns one computer to your job
2. The computer is completely fresh (like new)
3. Runs all your steps
4. Sends you the results
5. Deletes itself (computer is thrown away)

Next time you push, you get a brand new computer!
```

**Why This Matters:**
- Each run gets a fresh start (no leftover files from previous runs)
- Multiple projects can run at same time (different computers)
- Your code runs on a real server (not your laptop)

---

### 5️⃣ **ACTIONS** - Pre-Made Automation Recipes

An **action** is code that someone else wrote to do common tasks.

```yaml
- uses: actions/checkout@v4
```

**In Simple Words:** Actions are like pre-made recipes. Instead of writing everything from scratch, you use recipes others created.

**Real-Life Kitchen Analogy:**
```
❌ WITHOUT ACTION (Do it yourself):
- Buy flour
- Buy eggs
- Buy sugar
- Measure everything
- Mix carefully
- Bake for exact time

✓ WITH ACTION (Use a recipe):
- Buy cake mix box
- Add eggs and oil
- Follow 3 simple steps
- Done! 🎂
```

**Common Actions Used:**

```yaml
# Action 1: Download your code
- uses: actions/checkout@v4
  # Gets your code from GitHub so you can use it

# Action 2: Install Node.js
- uses: actions/setup-node@v4
  with:
    node-version: 20
  # Sets up Node.js for you automatically

# Action 3: SSH to a server
- uses: appleboy/ssh-action@v1.0.3
  # Connects to your server automatically
```

---

### 6️⃣ **ENVIRONMENT VARIABLES** - Pass Information Around

**Environment variables** are like sticky notes you pass around with information.

```yaml
env:
  DATABASE_URL: "mongodb://..."
  JWT_SECRET: "super_secret_key"
```

**In Simple Words:** Environment variables are containers that hold important information your code needs.

**Real-Life Example:**
```
Your app needs information to work:

❌ BAD WAY (Hard-code it):
const password = "12345";  // Anyone can see it in code!

✓ GOOD WAY (Use environment variables):
const password = process.env.JWT_SECRET;
// Password is stored securely, not in code
```

**How It Works:**
```
1. Store sensitive info in GitHub Secrets (hidden)
2. Pass them as environment variables
3. Your code reads them from process.env
4. Never exposed in code!

Code reads:     process.env.DATABASE_URL
                        ↑
                     Gets value from → GitHub Secrets
```

**Example Environment Variables Your Project Needs:**

| Variable | What It Is | Example |
|----------|-----------|---------|
| `NODE_ENV` | Which mode the app runs in | `production` |
| `DATABASE_URL` | Where the database lives | `mongodb+srv://user:pass@server` |
| `JWT_SECRET` | Secret key for login tokens | `aB#$xYz9Q@wL` |
| `PORT` | Which port the server listens on | `5000` |
| `CLOUDINARY_KEY` | Image hosting credentials | `1234567890abc` |

---

## Your Workflow File Explained 🔍

Now let's look at your actual workflow file, piece by piece!

### Part 1: Name and Trigger

```yaml
name: CI/CD Pipeline
```
**What it does:** This is just the name of your automation (shown in GitHub UI).

```yaml
on:
  push:
    branches: [main]
```
**What it does:** "Start when code is pushed to the main branch"

**Real Example:**
```
Developer pushes code to main branch
        ↓
GitHub detects: "New code on main!"
        ↓
GitHub Actions trigger starts! 🚀
        ↓
Build job begins automatically
```

---

### Part 2: Build Job (Compile Your Code)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

**What it does:**
- Creates a job named `build`
- Uses latest Ubuntu computer
- This computer only exists during the job

```yaml
    steps:
      - uses: actions/checkout@v4
```

**What it does:** Downloads your code from GitHub

```
You (on laptop):              GitHub Server:
┌──────────────────┐          ┌─────────────────┐
│                  │          │ Your Repository │
│                  │  ← ← ← ← │ Download code   │
│ GitHub Runner    │          └─────────────────┘
│ (temporary PC)   │
└──────────────────┘
```

---

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
```

**What it does:**
1. **Install Node.js v20** - Needed to run JavaScript/TypeScript
2. **Cache npm packages** - Speed up future runs (reuse old packages)

**Think of it like:**
```
Installing a software:
1. Download Node.js installer
2. Run the installer
3. Verify installation
4. Say "OK, ready to go!"
```

---

```yaml
      - run: npm ci
```

**What it does:** Install all the packages (libraries) your project needs

**What is npm?**
```
npm = Node Package Manager

Your app needs libraries:
├── Express (for web server)
├── MongoDB (for database)
├── JWT (for login)
└── ... 30+ more libraries

npm downloads all of them automatically!
```

**`npm ci` vs `npm install`:**
- `npm install` - Can update versions (unsafe for CI/CD)
- `npm ci` - Uses exact versions from lock file (safe)

---

```yaml
      - run: npm run build
        env:
          NODE_ENV: ${{ secrets.NODE_ENV }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          # ... more variables
```

**What it does:**
1. Run the build script (TypeScript → JavaScript)
2. Pass environment variables to the build

**Build Process:**
```
TypeScript Code (src/index.ts):
┌──────────────────────┐
│ function greet() {   │
│   console.log("Hi"); │
│ }                    │
└──────┬───────────────┘
       │ TypeScript compiler
       ↓
JavaScript Code (dist/index.js):
┌──────────────────────┐
│ function greet(){    │
│   console.log("Hi"); │
│ }                    │
└──────────────────────┘

Why convert?
- Browsers/servers run JavaScript
- TypeScript helps catch errors
- Best of both worlds!
```

---

### Part 3: Deploy Job (Upload to Server)

```yaml
  deploy:
    needs: build
```

**What it does:** "Only run this job AFTER build job succeeds"

**If Build Fails:**
```
Build Job
│
├─ FAILS ❌
│
└─> Deploy Job: SKIPPED ⏭️
    (doesn't even try to run)
```

**If Build Succeeds:**
```
Build Job
│
├─ SUCCESS ✅
│
└─> Deploy Job: RUNS 🚀
    (can now deploy safely)
```

---

```yaml
    steps:
      - name: Deploy to Server via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
```

**What it does:** Connects to your EC2 server using SSH

**SSH Explained:**
```
GitHub Actions          EC2 Server
┌──────────────┐       ┌──────────┐
│              │       │          │
│ "Hi! I'm    │──────→│ "Who     │
│ GitHub      │       │ are you?"│
│ Actions,    │       │          │
│ here's my   │       │          │
│ SSH key"    │←──────│ "Welcome!│
│              │       │          │
└──────────────┘       └──────────┘

SSH = Secure Shell (secure connection)
Uses a special key instead of password
```

---

```yaml
          script: |
            cd /home/ubuntu/zodiack-backend
            git pull origin main
            npm install
            npm run build
            pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
            pm2 save
```

**What each line does:**

```bash
# Navigate to your project folder
cd /home/ubuntu/zodiack-backend
(Like saying: "Open the folder where my app lives")

# Get latest code
git pull origin main
(Downloads the new code you just pushed)

# Install packages
npm install
(Gets all the libraries needed)

# Compile TypeScript to JavaScript
npm run build
(Makes the dist/ folder with compiled code)

# Restart the app
pm2 reload zodiack-backend || pm2 start dist/index.js --name "zodiack-backend"
(Reload if app running, otherwise start new)

# Save the app state
pm2 save
(Remember this app, restart it if server reboots)
```

---

## GitHub Secrets - Keep Passwords Safe 🔐

### What Are Secrets?

**Secrets** are private information stored safely by GitHub.

**Examples of Secrets:**
- Database passwords
- API keys
- SSH private key
- JWT tokens

**Why Not Put Passwords in Code?**

```
❌ BAD (NEVER do this):

// This is in your GitHub repository (PUBLIC!)
const password = "MyDatabasePassword123";

Anyone can:
- See your password in GitHub
- Use your database
- Steal your data
- Break your app
```

**✓ GOOD WAY (Use Secrets):**

```
Step 1: Store in GitHub Secrets (private, hidden)
GitHub UI: "DATABASE_PASSWORD" = "MyDatabasePassword123"

Step 2: Reference in workflow
env:
  DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}

Step 3: Use in code
const password = process.env.DATABASE_PASSWORD;
// No one can see it in code!
```

### How to Set Up Secrets

```
1. Go to GitHub repo → Settings
2. Find "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Name: DATABASE_URL
   Value: mongodb://user:pass@localhost
5. Click "Add secret"

Done! Now use it in workflow:
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## Real-World Example: Your Project Flow 🔄

### What Happens When You Push Code

```
1. You: "I finished coding!"
   └─ git push origin main ↑

2. GitHub: "New code detected!"
   └─ GitHub Actions trigger ⚡

3. GitHub Runner: "I'm a fresh Ubuntu computer"
   └─ runs-on: ubuntu-latest 💻

4. Step 1: Download Code
   └─ actions/checkout@v4 ⬇️

5. Step 2: Install Node.js
   └─ actions/setup-node@v4 📦

6. Step 3: Install Packages
   └─ npm ci ⚙️

7. Step 4: Build TypeScript
   └─ npm run build 🔨

8. Check: Did build succeed?
   └─ If YES → Go to step 9
   └─ If NO → Stop here! 🛑

9. SSH to EC2 Server
   └─ appleboy/ssh-action 🌐

10. Commands on EC2:
    ├─ git pull (get new code)
    ├─ npm install (install packages)
    ├─ npm run build (compile code)
    ├─ pm2 reload (restart app)
    └─ pm2 save (remember state)

11. Result: Your app is updated! 🎉
    └─ Users see the new version
```

---

## Component Behavior Explained 🔧

### 1️⃣ GitHub Runner Behavior

```
Each time workflow runs:

START:
┌─────────────────────────────┐
│ Fresh Ubuntu Computer (NEW) │
│ - Nothing from previous run │
│ - No extra files            │
│ - Clean slate!              │
└────────────┬────────────────┘
             │ Your steps run
             ↓
┌─────────────────────────────┐
│ Step 1: Download code       │
│ Step 2: Install Node        │
│ Step 3: Build               │
│ Step 4: Deploy              │
└────────────┬────────────────┘
             │ All done!
             ↓
END:
┌─────────────────────────────┐
│ Computer DELETED ❌         │
│ (Disappears, never used again)
└─────────────────────────────┘

Next run? You get a brand new computer!
```

---

### 2️⃣ Build Job Behavior

```
┌─ Build Job Starts
│
├─ Step 1: Checkout ✓
├─ Step 2: Setup Node ✓
├─ Step 3: Install packages ✓
├─ Step 4: Build code...
│
├─ Build succeeds? ✅ YES
│  └─ Continues to next step
│
└─ Build fails? ❌ NO
   └─ STOPS HERE!
      Deploy job never runs
      You get error message
```

---

### 3️⃣ SSH Connection Behavior

```
GitHub Actions              Your EC2 Server
┌──────────────┐           ┌──────────────┐
│ Has:         │           │ Has:         │
│ - Your code  │           │ - Your app   │
│ - Built app  │           │ - Old version│
│              │           │              │
└────────┬─────┘           └────────┬─────┘
         │ SSH key                  │
         │ matches?                 │
         └──────────┬───────────────┘
                    ↓
            ✅ Connection opens!
            │
            ├─ Execute: git pull
            ├─ Execute: npm install
            ├─ Execute: npm build
            ├─ Execute: pm2 reload
            ├─ Execute: pm2 save
            │
            └─ Connection closes!
               App updated! 🎉
```

---

### 4️⃣ PM2 Behavior

```
PM2's Job: Keep your app running 24/7

Scenario 1: App is already running
┌───────────────────────┐
│ Old App Version       │
│ (Running)             │
│ Users connected ✓     │
└──────────┬────────────┘
           │ pm2 reload
           ↓
┌───────────────────────┐
│ New App Version       │
│ (Takes over)          │
│ Users stay connected! │
│ Zero downtime! ⚡     │
└───────────────────────┘

Scenario 2: App is not running (first deploy)
┌───────────────────────┐
│ No app running ✗      │
└──────────┬────────────┘
           │ pm2 start
           ↓
┌───────────────────────┐
│ New App Version       │
│ (Just started)        │
│ Ready for users! ✓    │
└───────────────────────┘

Scenario 3: App crashes
┌───────────────────────┐
│ App crashes ❌        │
│ PM2 detects it        │
└──────────┬────────────┘
           │ Auto-restart
           ↓
┌───────────────────────┐
│ App restarted ✓       │
│ Back online!          │
│ No manual work!       │
└───────────────────────┘
```

---

## Common Mistakes & How to Fix Them 🚨

### Mistake 1: Forgetting Secrets

```yaml
# ❌ WRONG
env:
  DATABASE_URL: mongodb://user:pass@localhost
  # Password is exposed!

# ✅ CORRECT
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  # Password is hidden in GitHub Secrets
```

**Fix:**
1. Go to GitHub repo → Settings → Secrets
2. Add the missing secrets
3. Reference them in workflow

---

### Mistake 2: Hard-Coding Server IP

```yaml
# ❌ WRONG
with:
  host: 1.2.3.4
  # Your server IP is public!

# ✅ CORRECT
with:
  host: ${{ secrets.EC2_HOST }}
  # IP is hidden
```

---

### Mistake 3: Not Waiting for Build to Finish

```yaml
# ❌ WRONG
jobs:
  build:
    # ... build steps
  deploy:
    # No "needs" - runs at same time as build!
    # App might not be built yet!

# ✅ CORRECT
jobs:
  build:
    # ... build steps
  deploy:
    needs: build  # Wait for build to finish
    # App is definitely ready now!
```

---

### Mistake 4: Using Old Node Version

```yaml
# ❌ WRONG
- uses: actions/setup-node@v4
  with:
    node-version: 14  # Old, missing modern features

# ✅ CORRECT
- uses: actions/setup-node@v4
  with:
    node-version: 20  # Modern, secure
```

---

## Quick Checklist Before First Deployment ✅

```
Before you push code:

□ GitHub Secrets set up:
  □ EC2_HOST (server IP)
  □ EC2_USER (usually "ubuntu")
  □ EC2_SSH_KEY (private key)
  □ DATABASE_URL
  □ JWT_SECRET_KEY
  □ All other environment variables

□ Server ready:
  □ EC2 instance running
  □ SSH port (22) allowed in security group
  □ Node.js installed (v20)
  □ npm symlinks created

□ Workflow file correct:
  □ File at .github/workflows/ci.yml
  □ YAML syntax correct (indentation matters!)
  □ All secrets referenced correctly

□ Application ready:
  □ package.json exists
  □ "build" script works locally
  □ dist/ folder generated by build
  □ dist/index.js is entry point
```

---

## How to Debug If Something Breaks 🔍

### Step 1: Check GitHub UI

```
Go to: Your repo → Actions tab
       ↓
Look for red ❌ or green ✅

If red ❌:
  - Click the failed workflow
  - Scroll down to see error messages
  - Read what went wrong
```

### Step 2: Common Errors

```
Error: "npm: command not found"
└─ Fix: Install Node.js on EC2 (use NVM)
   Solution provided in earlier sections

Error: "Connection refused"
└─ Fix: Check EC2 is running, security group allows SSH

Error: "process.env.DATABASE_URL is undefined"
└─ Fix: Add DATABASE_URL to GitHub Secrets

Error: "dist/index.js not found"
└─ Fix: npm run build failed, check build logs
```

### Step 3: SSH to Server & Check

```bash
# SSH to your EC2
ssh -i your-key.pem ubuntu@your-ec2-ip

# Check if app is running
pm2 list

# View app logs
pm2 logs app-name

# Check Node.js installed
npm --version
node --version

# Try building manually
cd /home/ubuntu/zodiack-backend
npm install
npm run build
```

---

## Summary: The Complete Flow 📊

```
YOU (Developer)
    ↓
    Write code locally
    Test locally
    Push to GitHub
    ↓
GITHUB (Cloud)
    ↓
    Detects: Code on main branch!
    Triggers: Build job
    ↓
GITHUB RUNNER (Temporary Computer)
    ↓
    Download code
    Install Node.js
    Install packages
    Build TypeScript → JavaScript
    ↓
    Build succeeds? ✅
    ↓
    Triggers: Deploy job
    ↓
YOUR EC2 SERVER (Production)
    ↓
    SSH connection opens
    Download new code
    Install packages
    Build code
    Reload/Start app with PM2
    Connection closes
    ↓
USERS 🌍
    ↓
    See your new features!
    App works 24/7 with PM2
```

---

## Key Takeaways 🎯

| Concept | What It Does | Why It Matters |
|---------|-------------|----------------|
| **Trigger** | Starts automation | Automatic = less work |
| **Jobs** | Groups of tasks | Organize workflow |
| **Steps** | Individual actions | Break down into pieces |
| **Runner** | Computer that works | Separate from your laptop |
| **Actions** | Pre-made recipes | Don't reinvent the wheel |
| **Secrets** | Hide passwords | Keep data safe |
| **Environment Variables** | Pass information | Configure app dynamically |
| **PM2** | Keeps app running | 24/7 uptime + auto-restart |

---

## Next Steps 🚀

1. **Read your workflow file** with this guide
2. **Set up all secrets** on GitHub
3. **Test the workflow** by pushing code
4. **Watch it run** in Actions tab
5. **Check your server** - app should be updated
6. **Celebrate!** 🎉 You have CI/CD!

---

## Questions? 🤔

Common questions from beginners:

**Q: What if the workflow breaks my app?**
A: Worst case, you stop PM2 and restart the old version. Deploy → rollback takes seconds.

**Q: Can I run workflow without pushing code?**
A: Yes! Go to Actions → Select workflow → "Run workflow" → Choose branch

**Q: How often should I deploy?**
A: Every time you push! Each push = potential deployment. Automation ensures quality.

**Q: Is this secure?**
A: Yes! If you use Secrets correctly (which we do). Passwords never in code.

**Q: Can I see the logs?**
A: Yes! Go to Actions → Click workflow → See all output from each step

**Q: What if SSH key expires?**
A: Generate a new key, update the secret. Take 5 minutes.

---

## Final Words

GitHub Actions seems complex at first, but it's just:

1. **When** → Code pushed to main
2. **What** → Run these steps
3. **How** → Using environment variables
4. **Where** → On GitHub runner, then SSH to server
5. **Why** → Automate boring work, deploy safely

You've got this! 💪

Start small, understand each step, then build bigger workflows. Every expert was once a beginner who didn't give up.

Happy deploying! 🚀