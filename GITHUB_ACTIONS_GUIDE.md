# GitHub Actions CI/CD - Complete Analysis & Learning Guide

## 📋 Table of Contents

1. [Pipeline Analysis](#pipeline-analysis)
2. [Your Updates](#your-updates)
3. [Next Steps](#next-steps)
4. [GitHub Actions 80/20 Tutorial](#github-actions-8020-tutorial)
5. [Practical Examples](#practical-examples)
6. [Best Practices](#best-practices)

---

## Pipeline Analysis

Your current `pipeline.yaml` has a solid three-job structure:

- **explore-github-actions**: Logs environment information
- **verificate**: Validates HTML files  
- **fake_deploy**: Simulates production deployment

### ✅ What's Good

- ✅ Job dependencies properly configured (`needs: job-name`)
- ✅ Branch-specific deployment (only main branch deploys)
- ✅ Proper checkout of repository code
- ✅ Conditional step execution (`if: failure()`)
- ✅ Using established actions from GitHub Marketplace

---

## Your Updates

Great improvements! You've updated the action versions:

### 📦 Version Upgrades Made

| Component              | Before | After      | Status       |
|------------------------|--------|------------|--------------|
| actions/checkout       | v5     | **v6.0.2** | ✅ Updated   |
| actions/upload-artifact| v4     | **v6.0.0** | ✅ Updated   |

**Why this matters:**

- v6+ includes security patches and improvements
- Specific version pinning (v6.0.2) = stable + secure
- You're now using well-maintained, recent versions

---

## Next Steps

Your pipeline is **production-ready**. Here are optional enhancements:

### 1. **Add Permissions Block** (Security - Recommended)

```yaml
permissions:
  contents: read
  actions: read
```

**Why:** Restricts what GitHub Actions can do in your repo (least privilege principle)

### 2. **Add Concurrency** (Performance - Optional)

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false
```

**Why:** Prevents duplicate workflow runs, saves CI time

### 3. **Add Retention to Artifacts** (Housekeeping - Nice to have)

```yaml
- name: Save error logs
  if: failure()
  uses: actions/upload-artifact@v6.0.0
  with:
    name: relatorio-validacao
    path: html5validator.log
    retention-days: 30  # Auto-delete after 30 days
```

**Why:** Avoid storage bloat from old logs

### 4. **Enable Debug if Needed** (Troubleshooting)

In GitHub UI:

- Settings → Secrets and variables → Actions
- Add secret: `ACTIONS_STEP_DEBUG` = `true`

---

## Current Pipeline (With Your Updates)

Here's your pipeline annotated:

```yaml
name: Learning GitHub Actions
run-name: ${{ github.actor }} trying actions

# Optional: Uncomment to add permissions
# permissions:
#   contents: read
#   actions: read

on:
  push:
    branches: ["develop", "main"] # where the pipeline will be triggered

jobs:
  explore-github-actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."

      - name: Check out repository code
        uses: actions/checkout@v6.0.2  # ✅ Updated version
      
      - run: echo "The ${{ github.repository }} repository has been cloned to the runner."
      - run: echo "The workflow is now ready to test your code on the runner."

      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      - run: echo "This job's status is ${{ job.status }}."

  verificate:
    needs: explore-github-actions # only executes if explore-github-actions pass
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v6.0.2  # ✅ Updated version

      - name: HTML5Validator
        uses: Cyb3r-Jak3/html5validator-action@v7.2.0
        with:
          root: './'
          css: false
          format: json

      - name: Save error logs
        if: failure() # Only runs if the previous step fails
        uses: actions/upload-artifact@v6.0.0  # ✅ Updated version
        with:
          name: relatorio-validacao
          path: html5validator.log
          # retention-days: 30  # Optional: auto-delete logs after 30 days

  fake_deploy:
    needs: verificate # only executes if verificate pass
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' # only executes on main branch
    steps:
      - name: Fake Deploy
        run: echo "Deploying to production environment..."
```

---

## GitHub Actions 80/20 Tutorial

The 80/20 rule: **20% of features give you 80% of results**. Here's what you MUST know:

### Core Concepts You Need to Know

#### 1. **Workflow Triggers** (20% effort, 80% value)

Defines WHEN your workflow runs:

```yaml
on:
  push:
    branches: [main, develop]      # Runs on push to these branches
    paths: ['src/**', '*.yml']     # Runs only if these files change
  pull_request:
    branches: [main]                # Runs on PR to main
  schedule:
    - cron: '0 2 * * *'            # Runs at 2 AM UTC daily
  workflow_dispatch:               # Manual trigger via UI
```

**When to use each:**

- `push`: Test on every commit
- `pull_request`: Verify code before merge
- `schedule`: Run nightly tests, backups
- `workflow_dispatch`: Manual CI/CD runs

---

#### 2. **Jobs & Steps** (Essential structure)

```yaml
jobs:
  build:                            # Job name (appears in UI)
    runs-on: ubuntu-latest          # Runner OS
    timeout-minutes: 30             # Maximum duration
    
    steps:
      - name: Step name             # Display name
        run: echo "Hello"           # What to execute
        
      - name: Run script
        run: |                      # Multi-line script
          npm install
          npm run build
          npm test
```

**How it works:**

- Jobs run in PARALLEL by default
- Steps run SEQUENTIALLY within a job
- Use `needs: job-name` to create dependencies

---

#### 3. **Context & Variables** (The glue)

```yaml
# GitHub-provided contexts (free info)
${{ github.event_name }}       # What triggered the workflow (push, pull_request, etc)
${{ github.ref }}              # Branch name (refs/heads/main)
${{ github.sha }}              # Commit hash
${{ github.actor }}            # Who triggered it
${{ github.repository }}       # owner/repo
${{ github.workspace }}        # Working directory
${{ runner.os }}               # Ubuntu, Windows, macOS
${{ job.status }}              # Success, failure, cancelled

# Self-defined variables
env:
  NODE_VERSION: 18
  
steps:
  - run: echo ${{ env.NODE_VERSION }}
```

**Key insight:** Everything GitHub knows about the event is available as context. Use it!

---

#### 4. **Conditional Execution** (Super powerful)

```yaml
# Run only on specific branches
if: github.ref == 'refs/heads/main'

# Run only if previous step succeeded
if: success()

# Run only if previous step failed
if: failure()

# Run regardless of previous failures
if: always()

# Complex conditions
if: github.event_name == 'push' && github.ref == 'refs/heads/main'

# Check if file was changed
if: contains(github.event.head_commit.modified, 'src/app.js')
```

---

#### 5. **Actions (Reusable Tasks)**

```yaml
# Built-in actions (from GitHub)
- uses: actions/checkout@v6          # Clone your repo
- uses: actions/cache@v4             # Cache dependencies
- uses: actions/upload-artifact@v6   # Store build outputs

# Third-party actions (from marketplace)
- uses: Cyb3r-Jak3/html5validator-action@v7.2.0

# Action inputs (passing parameters)
- uses: actions/upload-artifact@v6
  with:
    name: my-artifact
    path: build/
    retention-days: 30
```

**Action structure:**

```bash
action-name@version
├── uses: references the action
├── with: passes inputs/parameters
├── id: names this step for later reference
└── if: conditional execution
```

---

#### 6. **Outputs & Data Passing** (Between steps)

```yaml
steps:
  # Step 1: Create data
  - name: Get version
    id: version              # IMPORTANT: Must have ID
    run: echo "version=1.2.3" >> $GITHUB_OUTPUT
  
  # Step 2: Use data from step 1
  - name: Use version
    run: echo "Deploying v${{ steps.version.outputs.version }}"

  # Use between jobs
  - name: Build artifact
    id: build
    run: |
      mkdir -p release
      echo "artifact_path=release/app.zip" >> $GITHUB_OUTPUT
```

**Output pattern:**

- Step 1 writes to `$GITHUB_OUTPUT`
- Step 2+ reads via `${{ steps.STEP_ID.outputs.VARIABLE_NAME }}`

---

#### 7. **Secrets (Sensitive Data)**

```yaml
# Access in workflow:
run: echo ${{ secrets.DATABASE_PASSWORD }}

# Set in GitHub:
# Settings → Secrets and variables → Actions → New repository secret
# Name: DATABASE_PASSWORD
# Value: your-secret-value
```

**Best practices:**

- NEVER hardcode secrets
- Use least-privilege principle
- Rotate regularly
- Use environment-specific secrets

---

### The 80/20 Takeaway

```yaml
name: 80/20 CI Pipeline
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      # 1. Check out code (essential)
      - uses: actions/checkout@v4
      
      # 2. Setup environment (usually needed)
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      # 3. Install + Build + Test (your logic)
      - run: npm install
      - run: npm run build
      - run: npm test
      
      # 4. Upload results (optional but useful)
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: coverage/
```

**That's it.** This pattern handles 80% of CI/CD needs.

---

## Practical Examples

### Example 1: Node.js Project with Testing

```yaml
name: Node CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16, 18, 20]  # Test on multiple Node versions
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'  # Built-in NPM caching
      
      - name: Install dependencies
        run: npm ci  # ci = clean install (better for CI)
      
      - name: Run linter
        run: npm run lint
        continue-on-error: true  # Don't fail if lint has warnings
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

**What's new:**

- `matrix`: Test multiple configurations
- `cache: 'npm'`: Automatic NPM cache
- `ci` instead of `install`: Faster, deterministic
- `continue-on-error`: Skip non-blocking failures
- Coverage reporting

---

### Example 2: Python Project with Multiple Jobs

```yaml
name: Python CI/CD
on:
  push:
    branches: [main]
  pull_request:

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - run: pip install flake8 black isort
      
      - name: Format check
        run: |
          black --check .
          isort --check-only .
          
      - name: Lint
        run: flake8 . --max-line-length=100
  
  test:
    runs-on: ubuntu-latest
    needs: quality  # Depends on quality job
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - run: pip install -r requirements.txt pytest pytest-cov
      - run: pytest --cov=. --cov-report=xml
      
      - uses: codecov/codecov-action@v3
  
  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'  # Only on main
    
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          # Your deployment script
          echo "Deploying..."
```

**Key patterns:**

- `needs: quality`: Job dependency
- `if: github.ref == 'refs/heads/main'`: Branch-specific
- Separate quality, test, deploy stages
- Secrets passed via `env`

---

### Example 3: Docker Build & Push

```yaml
name: Docker CI
on:
  push:
    branches: [main]
    tags: ['v*']  # Trigger on version tags

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to registry
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myrepo/myapp:latest
            myrepo/myapp:${{ github.sha }}
```

---

### Example 4: Manual Workflow with Inputs

```yaml
name: Manual Deploy
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ${{ github.event.inputs.environment }}
        run: |
          echo "Deploying version ${{ github.event.inputs.version }}"
          echo "to ${{ github.event.inputs.environment }}"
```

**How to use:** GitHub UI → Actions → Manual Deploy → Run workflow

---

## Best Practices

### 1. **Security**

```yaml
permissions:
  contents: read          # Read-only repository
  # Grant only what you need

# Never log secrets
- run: echo ${{ secrets.API_KEY }}  # ❌ WRONG - shows in logs
- run: |                             # ✅ RIGHT - masked automatically
    curl -H "Authorization: Bearer ${{ secrets.API_KEY }}"
```

### 2. **Performance**

```yaml
# Use cache to speed up builds
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}

# Use matrix for parallel testing
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16, 18, 20]
# Runs 9 combinations in parallel

# Fail fast (stop if one fails)
strategy:
  fail-fast: true
```

### 3. **Debugging**

```yaml
# Enable debug logging
- name: Enable debug logging
  if: runner.debug == '1'
  run: set -x  # bash debug mode

# Access logs via GitHub UI:
# Actions tab → Workflow run → Click on job → View logs
```

### 4. **Workflow Hygiene**

```yaml
# Add meaningful names
name: CI - Build, Test, Deploy

# Use run-name for better visibility
run-name: ${{ github.actor }} - ${{ github.event_name }}

# Add concurrency to prevent duplicates
concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref }}
  cancel-in-progress: true
```

### 5. **Status & Notifications**

```yaml
# Notify Slack on failure
- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    payload: |
      {
        "text": "❌ Workflow failed: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
      }
```

---

## Quick Reference Cheat Sheet

```yaml
# Trigger on push
on: push

# Trigger on specific branches
on:
  push:
    branches: [main, dev]

# Job dependency
needs: previous-job

# Run conditionally
if: github.ref == 'refs/heads/main'

# Parallel matrix testing
strategy:
  matrix:
    node: [16, 18, 20]

# Output from step
id: build
run: echo "result=success" >> $GITHUB_OUTPUT

# Use step output
${{ steps.build.outputs.result }}

# Use secrets
${{ secrets.MY_SECRET }}

# Context variables
${{ github.sha }}
${{ github.ref }}
${{ github.actor }}
```

---

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Marketplace](https://github.com/marketplace?type=actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

**Last Updated:** 2026-02-11  
**Your Pipeline Status:** ✅ Production-Ready (v6+ versions installed)  
**Next Step:** Consider adding `permissions` block for enhanced security
