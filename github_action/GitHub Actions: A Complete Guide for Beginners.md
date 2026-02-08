# ⚡ GitHub Actions Quick Reference Card

Perfect for printing or keeping open while coding!

---

## 📋 Basic Structure

```yaml
name: Workflow Name            # What to call it

on:                             # WHEN does it run?
  push:
    branches: [main]

jobs:                           # WHAT jobs to do?
  job_name:
    runs-on: ubuntu-latest      # WHERE to run?
    steps:                      # What STEPS to take?
      - run: command
```

---

## 🎯 Common Triggers

```yaml
# When code is pushed
on:
  push:
    branches: [main]

# When pull request created
on:
  pull_request:
    branches: [main]

# On schedule (like cron job)
on:
  schedule:
    - cron: '0 9 * * *'    # 9 AM every day

# Manually trigger
on: workflow_dispatch
```

---

## 🤖 Popular Actions

```yaml
# Download code
- uses: actions/checkout@v4

# Setup Node.js
- uses: actions/setup-node@v4
  with:
    node-version: 20

# SSH to server
- uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.HOST }}
    username: ${{ secrets.USER }}
    key: ${{ secrets.KEY }}

# Upload to server
- uses: appleboy/scp-action@v0.1.4
```

---

## 📝 Running Commands

```yaml
# Simple command
- run: npm install

# Multiple commands
- run: |
    npm install
    npm run build
    npm test

# With environment variables
- run: npm run build
  env:
    NODE_ENV: production
    DATABASE_URL: ${{ secrets.DB_URL }}

# With working directory
- run: npm build
  working-directory: ./backend
```

---

## 🔐 Using Secrets

```yaml
# Reference secret
env:
  PASSWORD: ${{ secrets.PASSWORD }}

# In code
const password = process.env.PASSWORD;

# GitHub UI: Settings → Secrets → Add secret
# Name: PASSWORD
# Value: your_actual_password
```

---

## 🔀 Control Flow

```yaml
# Only if previous step succeeded (default)
- run: npm build

# Only if failed
- run: echo "Build failed"
  if: failure()

# Always run
- run: cleanup
  if: always()

# Only on main branch
- run: deploy
  if: github.ref == 'refs/heads/main'
```

---

## 📊 Conditional Steps

```yaml
# Run only on main
- if: github.ref == 'refs/heads/main'
  run: npm run deploy

# Run only if file changed
- if: contains(github.event.pull_request.changed_files, 'package.json')
  run: npm install

# Run only on certain events
- if: github.event_name == 'push'
  run: echo "Code was pushed"
```

---

## 🔗 Job Dependencies

```yaml
jobs:
  build:
    # This job runs first
    runs-on: ubuntu-latest
    steps:
      - run: npm build

  test:
    # This runs first
    needs: build
    steps:
      - run: npm test

  deploy:
    # This runs last (after both build and test)
    needs: [build, test]
    steps:
      - run: deploy script
```

---

## 📈 Matrix (Run Multiple Versions)

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
    # This runs 6 times total (3 node × 2 OS)
```

---

## 📤 Uploading Artifacts

```yaml
# Upload files
- uses: actions/upload-artifact@v3
  with:
    name: build-output
    path: dist/

# Download in another job
- uses: actions/download-artifact@v3
  with:
    name: build-output
```

---

## 🌍 Environment Variables Available

```bash
${{ github.repository }}      # owner/repo
${{ github.ref }}              # refs/heads/main
${{ github.sha }}              # commit SHA
${{ github.event_name }}       # push, pull_request, etc
${{ github.actor }}            # who triggered it
${{ github.workspace }}        # working directory
```

---

## 🛠️ Useful Techniques

### Echo to console
```yaml
- run: echo "Hello World"
```

### List files
```yaml
- run: ls -la
```

### Check Node version
```yaml
- run: node --version && npm --version
```

### Run script from package.json
```yaml
- run: npm run build    # Runs "build" script
```

### Install globally
```yaml
- run: npm install -g pm2
```

---

## 🐛 Debugging

### View full logs
Go to: Actions → Click workflow → See all output

### SSH to runner (for debugging)
```yaml
- uses: mxschmitt/action-tmate@v3
  # Now you can SSH into runner and debug
```

### Print variables
```yaml
- run: |
    echo "Branch: ${{ github.ref }}"
    echo "Actor: ${{ github.actor }}"
    echo "Workspace: ${{ github.workspace }}"
```

---

## 📞 Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `command not found` | Tool not installed | Add setup action or `npm install -g` |
| `permission denied` | Missing sudo | Add `sudo` before command |
| `secret not found` | Typo in secret name | Check GitHub Secrets spelling |
| `connection refused` | Can't reach server | Check security group, server IP |
| `file not found` | Wrong path | Use `ls` to verify path |

---

## ✅ Pre-Deployment Checklist

### GitHub Setup
```
☐ Secrets configured (EC2_HOST, EC2_USER, EC2_SSH_KEY, etc)
☐ Workflow file in .github/workflows/
☐ YAML syntax correct (indentation!)
```

### Server Setup
```
☐ EC2 instance running
☐ Security group allows SSH (port 22)
☐ Node.js installed (v20+)
☐ npm available in PATH (/usr/local/bin/npm)
☐ PM2 installed globally
```

### Code Setup
```
☐ package.json exists
☐ build script works: npm run build
☐ dist/ folder created after build
☐ dist/index.js exists
```

---

## 🎓 Learning Order (Beginner → Advanced)

### Week 1: Basics
- [ ] Understand triggers
- [ ] Learn jobs and steps
- [ ] Try first workflow

### Week 2: Intermediate
- [ ] Use secrets properly
- [ ] Add environment variables
- [ ] Deploy to simple server

### Week 3: Advanced
- [ ] Matrix builds
- [ ] Conditional steps
- [ ] Artifacts and caching

---

## 📚 File Structure

```
your-repo/
├── .github/
│   └── workflows/
│       └── ci.yml                 ← Workflow file here!
├── src/
│   ├── index.ts
│   └── ...
├── package.json
├── tsconfig.json
└── .gitignore
```

---

## 🚀 Workflow Template (Copy & Paste Ready)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - run: npm ci
      - run: npm run build
        env:
          NODE_ENV: production
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /path/to/app
            git pull origin main
            npm install
            npm run build
            pm2 reload app || pm2 start dist/index.js --name app
            pm2 save
```

---

## 🎯 Essential Secrets to Set Up

```
EC2_HOST              Your EC2 IP address
EC2_USER              ubuntu (for EC2)
EC2_SSH_KEY           Private key content
DATABASE_URL          MongoDB connection string
JWT_SECRET_KEY        Long random string for JWT
NODE_ENV              production
PORT                  5000 or your port
API_KEY               Your API key
SLACK_WEBHOOK         For notifications
```

---

## 💡 Pro Tips

1. **Cache dependencies** for speed:
   ```yaml
   - uses: actions/setup-node@v4
     with:
       cache: 'npm'  # Saves time!
   ```

2. **Use concurrency** for multiple runs:
   ```yaml
   concurrency:
     group: ${{ github.ref }}
     cancel-in-progress: true
   ```

3. **Add status badges** to README:
   ```markdown
   ![Tests](https://github.com/user/repo/actions/workflows/ci.yml/badge.svg)
   ```

4. **Schedule periodic runs**:
   ```yaml
   on:
     schedule:
       - cron: '0 0 * * 0'  # Weekly
   ```

5. **Notify on failure** (send email/Slack):
   ```yaml
   - if: failure()
     run: send_notification.sh
   ```

---

## 🌐 Resources

- **Official Docs**: https://docs.github.com/en/actions
- **GitHub Marketplace**: https://github.com/marketplace?type=actions
- **Workflow Syntax**: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions

---

## 📱 Print This!

Perfect for:
- 👨‍💻 Desk reference
- 📚 Study material
- 🎓 Classroom handout
- 🔍 Quick lookup while coding

---

**Remember:** GitHub Actions = Automation = Less Work, More Reliability! 🚀