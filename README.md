
<div  align="center">
<h1>
بسم الله الرحمن الرحيم
</h1>
</div>

# GitHub Actions Guide

---

## Table of Contents

1. Introduction to CI/CD and GitHub Actions
2. GitHub Platform Overview
3. Workflows and Actions
4. Workflow Components and Execution
5. Your First Workflow
6. Pricing and Runners
7. GitHub Marketplace
8. YAML Basics for GitHub Actions
9. Types of Workflow Triggers
10. Jobs and Steps in Detail
11. Matrix Strategy
12. Conditionals and Expressions
13. Logging and Annotations
14. Secrets and Secure Data
15. Understanding Variables, Contexts, Secrets, and Runtime Data Flow in GitHub Actions
16. Conclusion 

---

## 1) Introduction to CI/CD and GitHub Actions

### What Problem Are We Actually Solving?

Imagine a team of 10 developers all working on the same codebase. Developer A finishes a feature and pushes it. Developer B pushes another change an hour later. How do you know both changes still work together? How do you know they didn't accidentally break something that was already working? In the early days of software, you found out the hard way — usually during a late-night deployment when something exploded in production.

**CI/CD is the systematic answer to this problem.** It automates the process of verifying code quality and delivering software, so that problems are caught early (when they're cheap to fix) rather than late (when they're expensive and embarrassing).

---

### Continuous Integration (CI)

**What it is conceptually:**

**CI** is a development practice where every developer merges their code changes into a shared branch frequently — ideally multiple times a day. Each merge automatically triggers a process that builds the application and runs all the tests. If something breaks, the team knows immediately and can fix it before anyone else is affected.

The word "integration" here means integrating your individual changes with everyone else's. The word "continuous" means doing this constantly, not once a week in a big stressful merge session.

**Why it matters in microservices:**

In a microservices architecture, dozens of small services talk to each other over APIs. Service A depends on Service B. If someone changes Service B's API without testing whether Service A is still working, you might not find out until both services are deployed together. CI catches this by running automated tests that verify the interaction contracts between services.

**What CI typically does:**

- Installs all dependencies
- Compiles or builds the project
- Runs unit tests (does each function work in isolation?)
- Runs integration tests (do components work together?)
- Runs linting and formatting checks (is the code style consistent?)
- Runs security scanning (are there known vulnerabilities in the dependencies?)

---

### Continuous Delivery vs. Continuous Deployment (CD)

Both terms describe what happens **after** CI passes. They differ in how much human involvement is required before code reaches production.

**Continuous Delivery** means the pipeline always produces a deployable artifact. After CI tests pass, the code is automatically deployed to a staging environment (a production-like environment used for final checks), but a human must manually approve and trigger the actual production deployment.

This is an ideal choice for teams that must meet compliance standards, obtain business approval before releases, or ensure a final human review before going live.

**Continuous Deployment** removes the manual approval gate entirely. Every commit that passes all automated checks is automatically deployed to production. No human clicks a button. This requires very high confidence in your automated test coverage and monitoring.

A simple mental model:

|Term|What it means|
|---|---|
|**Continuous Delivery**|Always _ready_ to release — a human pulls the trigger|
|**Continuous Deployment**|Always _automatically_ released — the pipeline pulls the trigger|

---

### Where GitHub Actions Fits

GitHub Actions is **not** just a CI/CD tool. It is a general-purpose event-driven automation platform. Anything that can be described as `"when X happens, do Y"` is a candidate for a GitHub Actions workflow. Examples beyond CI/CD:

- When a new issue is opened → automatically label it and assign it to the right team
- When a pull request is merged → post a notification to a Slack channel
- Every night at midnight → run a database cleanup job
- When someone stars the repo → update a public metrics dashboard
- When a security vulnerability is detected → open an issue and notify the maintainer

This is why GitHub Actions is worth learning deeply — it becomes the automation backbone of your entire development workflow, not just your test pipeline.

---

## 2) What GitHub Offers Beyond Code Hosting

Understanding the full platform helps you understand how GitHub Actions fits in as one piece of a larger system:

- **Pull Requests** — A structured way to propose, review, discuss, and merge code changes. Pull requests are the primary collaboration primitive in modern software development.
- **Issues** — A lightweight project tracking system for bugs, feature requests, and tasks.
- **Code Review** — Inline comments on diffs, approval workflows, required reviewers.
- **Security Scanning** — Automated tools including Dependabot (dependency vulnerability alerts), CodeQL (static analysis for code vulnerabilities), and secret scanning (detecting accidentally committed passwords and tokens).
- **GitHub Copilot** — An AI coding assistant integrated directly into IDEs and the GitHub web editor.
- **GitHub Actions** — The automation layer that ties everything together.

---

## 3) Workflows and Actions

### What Is a Workflow?

A workflow is a complete, self-contained automation script. Think of it like a recipe: it describes what ingredients are needed (triggers), who does the cooking (runners), and what steps to follow (jobs and steps).

Technically, a workflow is a **YAML file** stored in the `.github/workflows/` directory of your repository. GitHub watches this directory and automatically reads any `.yml` or `.yaml` files it finds there. You can have multiple workflow files in the same repository — one for CI testing, one for deployment, one for issue automation, and so on. Each file is an independent workflow.

Let's see **The key characteristics of a workflow:**

- It is version-controlled alongside your code (change history, rollbacks, code review for workflow changes)
- It is portable — anyone who forks your repository gets your workflows too
- It is triggered by events — it does not run unless something causes it to run
- It can contain one or multiple jobs

---

### What Is an Event (Trigger)?

An event is the thing that wakes up a workflow and tells it to start running. Without an event, a workflow just sits there as a YAML file doing nothing.

Events come in several flavors:

- **Repository events** — actions taken directly in the repository (pushing code, opening a pull request, creating a release, opening an issue, starring the repo)
- **Schedule** — a time-based trigger using cron syntax
- **Manual** — a human or external system deliberately triggers the workflow
- **External API calls** — a system outside GitHub sends an HTTP request to GitHub asking it to trigger a workflow

The event also carries **context data** (called a payload) about what happened. For a push event, the payload contains information about what commits were pushed, who pushed them, and which branch was affected. Your workflow can read this data to make decisions.

---
### What Is an Action?

An action (lowercase) is a reusable, self-contained unit of automation. Think of it as a pre-built function you can call in your workflow without having to write the underlying logic yourself.

For example, checking out your repository code from GitHub onto the runner machine is a common task that every workflow needs to do. Instead of writing 20 lines of shell script to handle authentication, cloning, and edge cases, you simply call the `actions/checkout` action. Someone already wrote and tested that logic. You just use it.

Actions can be:

- **Official actions** — maintained by GitHub itself (e.g., `actions/checkout`, `actions/setup-node`)
- **Community actions** — published by third parties on the GitHub Marketplace
- **Custom actions** — written by you and stored in your own repository

Actions can be written in **JavaScript/TypeScript** (runs directly on the runner) or packaged as **Docker containers** (runs in an isolated environment with full control over the OS and dependencies).

---

### What Is a Runner?

A runner is the machine that actually executes your workflow jobs. When a workflow is triggered, GitHub (or your own infrastructure) provides a fresh, clean virtual machine, runs all the steps in your job, and then <span style ="color:red">discards the machine</span>.

#### Let's See Runner Types :

- 1. **GitHub-hosted runners**: come pre-installed with a huge variety of tools: Node.js, Python, Java, .NET, Docker, Git, and many more. You don't have to configure them. You just say "give me an Ubuntu machine" and GitHub handles the rest.

- 2. **Self-hosted runners**: are machines you register yourself. You control the hardware, the operating system, the installed software, and the network access. This is useful when you need specific hardware (GPU for ML workloads, ARM processors for mobile builds), access to internal resources (databases behind a firewall), or want to reduce costs for high-volume workflows.

---

## 4) Workflow Components and Execution

### The Component Hierarchy

A GitHub Actions workflow has a clear hierarchy:

```
Workflow (the .yml file)
└── Job(s) (run on a runner, parallel by default)
    └── Step(s) (run sequentially within a job)
        └── Run command OR Uses action
```

Every level of this hierarchy has a specific purpose and its own rules.

---

### Understanding the Parallel vs. Sequential Rule

This is one of the most important concepts to internalize early, because getting it wrong leads to confusing failures.

#### Let's Start : 

- **Jobs are parallel by default.** If your workflow has three jobs — `test`, `build`, and `deploy` — all three will start at the same time as soon as the workflow is triggered. This is efficient because independent jobs can run concurrently. However, it means you **cannot** assume that `build` is finished before `deploy` starts unless you explicitly declare that dependency.

- **Steps within a job are always sequential.** Every step in a job runs one after another, in the order you wrote them. If step 3 fails, step 4 never runs (unless you use a special condition to override this behavior).

**The `needs` keyword** is how you break the default parallelism between jobs. If `deploy` declares `needs: build`, then GitHub will wait for `build` to complete successfully before starting `deploy`.

---

### The Code Checkout Rule

He surprised many "Including me (*^_^*)": **when your job starts, the runner's workspace is completely empty.** There are no files from your repository. The runner is a fresh machine that knows nothing about your code.

This is intentional — it ensures a clean, reproducible environment for every run. But it means that if your workflow needs to read, compile, or test any files in your repository, the **first thing you must do is check out the code** using the `actions/checkout` action.

Forgetting this is one of the most common mistakes.

---

### Component Summary Table

|Component|Description|Example|
|---|---|---|
|**Workflow file**|A YAML file describing the full automation pipeline|`.github/workflows/ci.yml`|
|**Trigger (`on`)**|The event that starts the workflow|`on: push` or `on: pull_request`|
|**Job**|A group of steps that run on a single runner|`jobs: build:`|
|**Runner**|The OS/environment where a job executes|Ubuntu, Windows, macOS, or self-hosted|
|**Step**|A single command or action within a job|`- run: npm test` or `- uses: actions/checkout@v5`|

---

## 5) Your First Workflow

### Reading a Workflow File for the First Time

Before we looking at any example, we should understand the top-level structure of every workflow file:

```
name:       → The display name of the workflow (shown in the GitHub UI)
on:         → The trigger(s) that activate this workflow
jobs:       → The actual work to be done, organized into named groups
```

Everything else is a detail that hangs off one of these three top-level keys.

- The `on` section answers: "When should this run?" 
- The `jobs` section answers: "What should it do when it runs?"

---

### The `runs-on` Key

Every job must declare `runs-on`, which tells GitHub which type of runner to provision. The most common values are:

- `ubuntu-latest` — the latest LTS version of Ubuntu Linux
- `windows-latest` — the latest Windows Server
- `macos-latest` — the latest macOS

You can also specify exact versions like `ubuntu-22.04` if you need predictable, stable behavior and don't want the runner OS to change when GitHub updates `latest`.

---

### The `uses` vs. `run` Distinction

Every step does one of two things:

- **`run:`** — executes a raw shell command directly on the runner (VM)
- **`uses:`** — calls a pre-built action (from GitHub, the Marketplace, or your own repo)

You cannot use both `run` and `uses` in the same step. They are mutually exclusive.

---

### First Workflow Example

> **Bug Fixed:** The original used `actions/checkout@v6.0.2`, which does not exist. The current stable major version is `v5` (as of the time of writing, check the official repository for the latest). Pinning to a major version tag like `@v5` is the standard practice.

```yaml
name: First Workflow

on:
  push:
    branches:
      - main

jobs:
  example-job:
    runs-on: ubuntu-latest

    steps:
      # This step has no dependency on repo files, so it can safely run first
      - name: Print a welcome message
        run: echo "Welcome to our GitHub Actions workflow!"

      # BEST PRACTICE: Put checkout as early as possible so all subsequent
      # steps have access to your repository files
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Confirm checkout
        run: echo "Repository cloned successfully!"

      - name: List repository files
        run: ls -la
```

### What Each Part Does

- `on: push: branches: [main]` → This workflow only activates on pushes to the `main` branch. Pushes to feature branches, for example, are ignored.
- `runs-on: ubuntu-latest` → GitHub spins up a fresh Ubuntu VM for this job.
- `uses: actions/checkout@v5` → This action authenticates with GitHub, clones your repository into the runner's working directory, and checks out the commit that triggered the workflow.
- `run: ls -la` → After checkout, the working directory contains your repo files. This command lists them to confirm everything is there.

---

## 6) Pricing and Runners

### The Mental Model for Pricing

GitHub Actions charges you for **compute time**. Specifically, it counts the number of minutes your jobs run on GitHub-hosted runners, multiplied by a factor based on the OS. The factor exists because Windows and macOS VMs cost GitHub more to operate than Linux VMs, and those costs are passed on.

The key insight is that **public repositories are completely free** — no minute limits. This is GitHub's investment in the open-source ecosystem. All the open-source projects you rely on get free CI.

For **private repositories**, GitHub gives you a pool of free minutes each month. The exact amount depends on your plan (Free, Pro, Team, or Enterprise). If you exceed the pool, additional minutes are billed at the rates below.

---

### GitHub-Hosted Runner Costs

These machines come pre-configured with everything you need — no setup required.

|OS|Price per Minute|
|---|---|
|Linux (Ubuntu)|$0.008|
|Windows|$0.016|
|macOS|$0.084|

Notice that Windows costs 2× Linux, and macOS costs more than 10× Linux. This is why most CI work defaults to Ubuntu — it's the fastest and cheapest option for workloads that don't need a specific OS.

**Free tier for private repositories:** 2,000 minutes/month across all workflows and repositories in your account. These minutes are drawn from a shared pool, not per-repository.

---

### Self-Hosted Runners

A self-hosted runner is any machine you register with GitHub to run your workflow jobs. It could be a Raspberry Pi under your desk, a VM in your company's data center, or a powerful cloud instance.

**GitHub does not charge you runner minutes for self-hosted runners.** The compute cost is yours entirely. You pay for the machine and its operation, not GitHub.

**When self-hosted runners make sense:**

- You need hardware GitHub doesn't offer (GPUs for ML training, ARM64 for mobile/embedded builds)
- Your jobs need access to internal resources that can't be reached from GitHub's cloud (internal databases, private APIs behind a firewall)
- Your volume of CI runs is high enough that self-hosted becomes cheaper than GitHub's rates
- You need a custom software environment that's impractical to set up fresh every job

**The tradeoff:** You become responsible for maintaining the machine — OS updates, security patches, installed software, uptime. GitHub-hosted runners are maintained by GitHub; self-hosted runners are maintained by you.

---

## 7) GitHub Marketplace

### What the Marketplace Is

The GitHub Marketplace is a directory of pre-built actions that anyone can use in their workflows. As of the time of writing, there are over 18,000 actions covering virtually every common task in software development.

Before writing any custom workflow logic, it's worth checking the Marketplace first. The action you need almost certainly already exists, is well-tested, and is maintained by someone with deep expertise in that specific domain.

### How Actions Are Structured

Every action has a defined interface consisting of:

- **Inputs** — parameters you pass to configure the action's behavior. These are declared with the `with:` key in your step.
- **Outputs** — values the action makes available to subsequent steps. These are accessed via the `steps.<step-id>.outputs.<output-name>` expression.

This input/output interface is what makes actions composable. One action can produce an output (say, a version number), and the next action can consume that output as an input.

---

### Marketplace Example

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'   # Input: which Node.js version to install
    cache: 'npm'         # Input: automatically cache npm dependencies

# The setup-node action also exposes outputs you can use:
- name: Show the installed Node path
  run: echo "Node is at ${{ steps.setup.outputs.node-path }}"
```

---

### Understanding Action Version Pinning

When you write `uses: actions/checkout@v5`, the `@v5` part is a Git ref — it points to a specific commit, branch, or tag in the action's repository. There are three common strategies:

|Pinning Style|Example|Behavior|
|---|---|---|
|Major version tag|`@v5`|Gets all minor and patch updates automatically. Recommended for most cases.|
|Full SHA|`@a81bbbf`|Completely frozen. Never changes. Most secure but requires manual updates.|
|Branch|`@main`|Always gets the latest code. Risky — the action could change behavior without warning.|

- The standard recommendation is to pin to a major version tag (`@v4`, `@v5`, etc.). This gives you automatic bug fixes and security patches without breaking changes.

---

## 8) YAML Basics for GitHub Actions

### Why YAML?

YAML was designed to be human-readable. It uses indentation and plain text to represent structured data, which makes it easier to write and review than alternatives like XML or JSON. GitHub Actions chose YAML for workflow files because workflows need to be read and edited frequently by humans.

**YAML is easy to learn, but some small rules about indentation, strings, and data types can cause unexpected errors. Understanding these rules early can save you a lot of time when debugging later.**

---

### Core YAML Rules

**Indentation is syntax, not style.** In most programming languages, indentation is optional formatting. In YAML, it defines the structure. A misplaced space can change the entire meaning of a document or cause a parse error.

- Use **2 spaces** per indentation level (this is the community standard for GitHub Actions files)
- Be consistent — mixing 2-space and 4-space indentation within a file will cause errors

**Lists** use a leading hyphen `-` followed by a space:

```yaml
branches:
  - main
  - develop
  - "release/**"
```

**Key-value maps** use a colon followed by a space:

```yaml
name: My Workflow
runs-on: ubuntu-latest
```

**Comments** start with `#` and can appear anywhere:

```yaml
# This is a comment
run: npm test  # This is an inline comment
```

---

### YAML Scalar Types

YAML automatically detects the type of unquoted values:

```yaml
name: Ahmed        # String (no quotes needed)
age: 30            # Integer
active: true       # Boolean (true/false, yes/no, on/off all work)
score: 3.14        # Float
date: 2025-01-15   # Date (ISO format)
nothing:           # Null (empty value)
```

This auto-detection is usually convenient, but it can cause problems when YAML guesses wrong — which brings us to the quoting rules.

---

### Multi-Line Strings

YAML provides two operators for multi-line strings, and choosing the right one matters when your `run:` command spans multiple lines.

**Literal block `|` — preserves all newlines:**

```yaml
run: |
  echo "Step 1"
  echo "Step 2"
  npm install
  npm test
```

Each line is treated as a separate command executed in sequence. This is the most common style for multi-line `run` steps.

**Folded block `>` — collapses newlines into spaces:**

```yaml
description: >
  This is a very long description
  that wraps across multiple lines
  but will be treated as one line.
```

This is useful for long strings that you want to break for readability in the YAML file but keep as a single line in the actual value.

---

### YAML Data Types — Full Example

```yaml
name: Example User
age: 30
active: true
employment_date: 2025-02-01T15:20:30Z

# A list of maps
employees:
  - name: Ahmed
    role: Backend Developer
  - name: Sarah
    role: QA Engineer

# A nested map
config:
  timeout: 60
  retries: 3
  regions:
    - us-east-1
    - eu-west-1
```

---

### Single Quotes `' '` vs. Double Quotes `" "`

This is one of the most important distinctions for writing correct YAML, and it comes up constantly in GitHub Actions files.

**The fundamental difference:**

- Single quotes treat everything inside them as **literal text**. What you type is exactly what you get.
- Double quotes **process escape sequences**. Special codes like `\n` are interpreted and transformed.

---

#### Double Quotes `" "`

Use double quotes when you need escape sequences or special character formatting inside the string.

```yaml
message: "Hello\nWorld"
```

What gets stored: the string `Hello`, then a real newline character, then `World`.

When printed:

```
Hello
World
```

**Common escape sequences inside double quotes:**

|Escape|Meaning|
|---|---|
|`\n`|Newline (line break)|
|`\t`|Tab character|
|`\"`|Literal double quote character|
|`\\`|Literal backslash character|

**Windows path example — why escaping matters:**

```yaml
path: "C:\\Users\\Ahmed\\Documents"
```

What gets stored: `C:\Users\Ahmed\Documents`

If you wrote `"C:\Users\Ahmed"` without escaping, YAML would try to interpret `\U` and `\A` as escape sequences, and the result would be wrong or a parse error.

---

#### Single Quotes `' '`

Use single quotes when you want what you type to be exactly what gets stored. No escape sequences are processed. Backslashes are just backslashes.

```yaml
message: 'Hello\nWorld'
```

What gets stored: the literal string `Hello\nWorld`. The `\n` is two characters — a backslash and the letter n — **not** a newline.

**The only special case inside single quotes** is how to include a literal single quote character. You double it:

```yaml
text: 'It''s a great day'
```

What gets stored: `It's a great day`

---

#### Practical Comparison

```yaml
double: "Line1\nLine2"
single: 'Line1\nLine2'
```

|Variable|What's actually stored|When echoed|
|---|---|---|
|`double`|`Line1` + newline + `Line2`|Prints on two lines|
|`single`|`Line1\nLine2`|Prints literally as `Line1\nLine2`|

---

#### The YAML Auto-Conversion Gotcha

YAML's type auto-detection can silently change the type of your values. This is one of the most common sources of subtle bugs in YAML configuration files.

```yaml
enabled: yes      # Parsed as boolean true, NOT the string "yes"
count: 007        # Parsed as integer 7, NOT the string "007"
version: 1.10     # Parsed as float 1.1, NOT the string "1.10"
```

To force any value to be treated as a string, quote it:

```yaml
enabled: "yes"    # String "yes"
count: "007"      # String "007"
version: "1.10"   # String "1.10"
```

This is especially important in GitHub Actions when passing values like environment names, version strings, or flags that look like booleans or numbers.

---

#### Quick Reference: When to Use Which Quotes

|Use `' '` single quotes when...|Use `" "` double quotes when...|
|---|---|
|Writing regular expressions|You need `\n` (real newline)|
|Writing Windows file paths|You need `\t` (real tab)|
|Writing literal text with backslashes|You need Unicode escape sequences|
|You want exactly what you type|You need escape processing|

**Quick comparison table:**

| Feature                     | `" "`                   | `' '`                  |
| --------------------------- | ----------------------- | ---------------------- |
| Escape sequences processed  | Yes                     | No                     |
| `\n` becomes a real newline | Yes                     | No                     |
| Easier for Windows paths    | (must escape every `\`) | (backslash is literal) |
| Easier for regex            | (must double every `\`) | (backslash is literal) |
| Literal text, no surprises  | Less safe               |  Safer                 |

---

### Regex in YAML — Senior Best Practice

Regular expressions use backslashes heavily. In double quotes, every backslash must be escaped by doubling it. This makes regex patterns nearly unreadable in double quotes.

**Recommended — single quotes:**

```yaml
pattern: '^\d{3}-\d{2}-\d{4}$'
```

**Avoid — double quotes:**

```yaml
pattern: "^\\d{3}-\\d{2}-\\d{4}$"
```

Both produce the exact same regex. The single-quoted version is far easier to read and far less likely to contain a typo.

**Email regex comparison:**

| Style          | YAML Value               |
| -------------- | ------------------------ |
| Single quotes  | `'^\w+\@\w+\.\w+$'`      |
| Double quotes  | `"^\\w+\\@\\w+\\.\\w+$"` |

**Senior rule of thumb:** Prefer `' '` for literal strings and regex. Use `" "` only when you specifically need `\n`, `\t`, or another escape sequence.

This guidance applies across all YAML-based tooling: GitHub Actions, Docker Compose, Kubernetes manifests, Helm charts, Ansible playbooks.

---

## 9) Types of Workflow Triggers

### The Three Trigger Categories

Every GitHub Actions workflow starts with a trigger. There are three broad categories:

**1. Event-driven (webhook) triggers** — The workflow reacts to something that happens in your repository or organization. These are the most common triggers. The event carries a payload (data about what happened), and your workflow can read that data to make decisions.

**2. Schedule triggers** — The workflow runs at a predetermined time, regardless of any repository activity. Useful for routine maintenance, reports, cleanup jobs, and other time-based tasks.

**3. Manual triggers** — A human (or an external system) deliberately starts the workflow. Useful for deployments, one-off tasks, and processes that shouldn't run automatically.

---

### Trigger Overview Table

|Trigger Type|Description|Example|
|---|---|---|
|**Webhook Events**|Runs when a repository event occurs|`on: push`, `on: pull_request`, `on: issues`|
|**Scheduled**|Runs at specific times using cron syntax|`on: schedule: - cron: '5 2 * * *'`|
|**Manual**|User-initiated via UI, CLI, or API|`workflow_dispatch`, `repository_dispatch`|

---

### Webhook Event Triggers

Webhook triggers work because GitHub maintains a list of events that can occur in a repository. When an event occurs, GitHub sends an HTTP webhook (a POST request carrying the event payload) to all systems listening — including GitHub Actions.

The `on:` key in your workflow file declares which events should activate it. You can listen to a single event or many:

```yaml
on: push                          # Single event
on: [push, pull_request]          # Multiple events
```

**Filtering within events:**

Most events support filters so you don't trigger the workflow for every single occurrence. Common filters:

- `branches:` — only trigger on specific branches
- `branches-ignore:` — trigger on all branches except the listed ones
- `paths:` — only trigger when certain files change
- `paths-ignore:` — trigger on all file changes except the listed ones
- `tags:` — only trigger when specific tags are pushed
- `types:` — only trigger on specific sub-types of an event

---

### Webhook Triggers — Full Example

This workflow triggers on pushes or pull requests to `main` or any `release/**` branch, but **only when files inside `src/` change**. This path filter is valuable — it prevents the CI pipeline from running when you update documentation, fix a typo in a README, or change a GitHub Actions workflow file itself.

```yaml
name: CI on Push and Pull Request

on:
  push:
    branches:
      - main
      - "release/**"
    paths:
      - "src/**"

  pull_request:
    branches:
      - main
      - "release/**"
    paths:
      - "src/**"

jobs:
  test-job:
    runs-on: ubuntu-latest

    steps:
      - name: Git Repository Checkout
        uses: actions/checkout@v5

      - name: Run Tests
        run: npm test
```

---

### Working JavaScript Test Example

This is a minimal but complete Node.js project that demonstrates a real webhook trigger in action.

**`src/app.js`**

```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

module.exports = greet;

if (require.main === module) {
  console.log(greet("World"));
}
```

**`src/test.sh`**

```bash
#!/bin/bash
set -e  # Exit immediately if any command fails

EXPECTED="Hello, Test!"
OUTPUT=$(node -e "console.log(require('./src/app')('Test'))")

if [ "$OUTPUT" = "$EXPECTED" ]; then
  echo "Test passed!"
  exit 0
else
  echo "Test failed! Expected '$EXPECTED' but got '$OUTPUT'"
  exit 1
fi
```

**`package.json`**

>  The Node.js package manifest must always be named `package.json`. Without the `.json` extension, `npm install`, `npm test`, and all npm commands will fail to find this file.

```json
{
  "name": "simple-node-app",
  "version": "1.0.0",
  "description": "A simple Node.js app for GitHub Actions PR testing",
  "main": "src/app.js",
  "scripts": {
    "test": "bash src/test.sh"
  },
  "dependencies": {}
}
```

**The full workflow for this project:**

```yaml
name: PR Test Workflow

on:
  pull_request:
    branches:
      - main
    paths:
      - "src/**"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Run tests
        run: npm test
```

---
### Scheduled Triggers — Cron Syntax

#### What Is Cron?

Cron is a time-based job scheduler.. It uses a compact five-field expression to describe when a job should run. GitHub Actions adopted the same syntax for its `schedule` trigger.

All scheduled workflow times are in **UTC** — not your local timezone. This is a common source of confusion. If you want a job to run at 9 AM Eastern Time (UTC-5), you schedule it for `0 14 * * *` (14:00 UTC = 9:00 AM ET).

#### Cron Expression Structure

```
* * * * *
│ │ │ │ └── Day of the week (0–7, where both 0 and 7 = Sunday)
│ │ │ └──── Month (1–12)
│ │ └────── Day of the month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

An asterisk `*` means "every possible value" for that field. So `* * * * *` means "every minute of every hour of every day."

**Special syntax:**

|Syntax|Meaning|Example|
|---|---|---|
|`*`|Every value|`* * * * *` = every minute|
|`*/n`|Every n-th value|`*/15 * * * *` = every 15 minutes|
|`n,m`|Specific values|`0 9,17 * * *` = at 9:00 AM and 5:00 PM|
|`n-m`|A range|`0 9-17 * * *` = every hour from 9 to 17|

**Common examples:**

|Cron Expression|When it runs|
|---|---|
|`0 0 * * *`|Every day at midnight UTC|
|`0 12 * * 1`|Every Monday at noon UTC|
|`5 2 * * 6`|Every Saturday at 02:05 UTC|
|`*/15 * * * *`|Every 15 minutes|
|`0 9 * * 1-5`|Every weekday at 9:00 AM UTC|
|`0 0 1 * *`|First day of every month at midnight|

**Example scheduled workflow:**

```yaml
name: Scheduled Workflow

on:
  schedule:
    - cron: "0 0 * * *"   # Every day at midnight UTC
    - cron: "0 12 * * 1"  # Every Monday at noon UTC

jobs:
  scheduled-job:
    runs-on: ubuntu-latest
    steps:
      - name: Print schedule message
        run: echo "This workflow runs on a schedule!"
```

> **Important:** GitHub may delay scheduled workflows by up to a few minutes during periods of high load. For genuinely time-critical tasks, consider adding redundancy or using a dedicated scheduling service.

---
### Manual Triggers

#### Why Manual Triggers Exist

Not every automation should happen automatically. There are scenarios where you want a human or an external system to explicitly decide when to run a workflow:

- Deploying to production — you want a human to make the conscious decision to push to prod
- Running a database migration — a destructive operation that shouldn't trigger accidentally
- Running a maintenance job that only makes sense periodically
- Responding to an external system event (a monitoring alert, a customer support ticket, a third-party build system completing)

GitHub provides two mechanisms for this: `workflow_dispatch` for human-initiated runs and `repository_dispatch` for external-system-initiated runs.

---

#### `workflow_dispatch` vs. `repository_dispatch`

|Feature|`workflow_dispatch`|`repository_dispatch`|
|---|---|---|
|**Purpose**|Human manually triggers the workflow|External system triggers the workflow via API|
|**Trigger Source**|GitHub UI, GitHub CLI (`gh`), or API|HTTP POST to the GitHub API|
|**Custom Event Types**|No (just "run this workflow")|Yes — you define event type strings|
|**Inputs**|Typed, named inputs with validation|Free-form JSON payload (`client_payload`)|
|**Common Use Cases**|Deployments, maintenance, ad-hoc runs|Monitoring tools, third-party CI, incident response|

---

#### `workflow_dispatch` — Human-Initiated Trigger

`workflow_dispatch` adds a "Run workflow" button to the Actions tab of your repository. When a user clicks it, they can optionally fill in input parameters you've defined before the workflow starts.

**Theory on inputs:**

Inputs let the person running the workflow provide configuration at runtime. Instead of having separate workflows for deploying to staging vs. production, you can have one workflow that asks "where do you want to deploy?" when someone triggers it. This reduces duplication and centralizes your deployment logic.

Each input has a type that GitHub validates before the workflow starts:

|Type|Description|When to use|
|---|---|---|
|`string`|Plain text|Names, versions, custom flags|
|`boolean`|True/false checkbox in the UI|Feature flags, on/off switches|
|`choice`|Dropdown menu with predefined options|Environments, regions, actions|
|`number`|Numeric value|Replica counts, timeouts, limits|
|`environment`|Maps to a GitHub Environment (with protection rules and secrets)|Deployment targets with approvals|

**Example:**

```yaml
name: Manual Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Target environment"
        required: true
        default: staging
        type: choice
        options:
          - production
          - staging
          - development

      version:
        description: "Version to deploy (e.g. 1.4.2)"
        required: true
        type: string

      dry_run:
        description: "Run in dry-run mode (no actual deployment)"
        required: false
        default: "false"
        type: boolean

jobs:
  manual-job:
    runs-on: ubuntu-latest
    steps:
      - name: Show selected environment
        run: echo "Deploying version ${{ inputs.version }} to ${{ inputs.environment }}"

      - name: Check dry run mode
        if: ${{ inputs.dry_run == true }}
        run: echo "DRY RUN MODE - no actual deployment will occur"
```

**Input configuration properties:**

|Property|Description|
|---|---|
|`description`|Help text displayed to the user in the GitHub UI|
|`required`|Whether the user must provide a value before running|
|`default`|Pre-filled value if the user doesn't change it|
|`type`|The data type and UI control (`string`, `boolean`, `choice`, `number`, `environment`)|
|`options`|Available options in the dropdown (only valid with `type: choice`)|

**Accessing input values inside the workflow:**

```yaml
# Primary syntax (recommended, shorter)
run: echo "Deploying to ${{ inputs.environment }}"

# Alternative syntax (equivalent, uses the full event context)
run: echo "Deploying to ${{ github.event.inputs.environment }}"
```

**Running `workflow_dispatch` from the CLI:**

```bash
gh workflow run deploy.yml --field environment=production --field version=1.4.2
```

---

#### `repository_dispatch` — External-System Trigger

`repository_dispatch` is designed for situations where something outside GitHub needs to trigger a workflow. The external system makes an authenticated HTTP POST request to the GitHub API, and GitHub starts the corresponding workflow.

**Theory on how it works:**

The external system sends a JSON payload containing an `event_type` string (which you define) and an optional `client_payload` object (arbitrary JSON data you want to pass to the workflow). Your workflow listens for specific event types and can read the payload data.

This creates a clean integration contract: any external system that can make HTTP requests can trigger a GitHub Actions workflow, with no access to your repository code required.

```yaml
name: Handle Repository Dispatch

on:
  repository_dispatch:
    types: [incident_report]  # Only activates for this event type

jobs:
  handle-incident-report:
    runs-on: ubuntu-latest
    steps:
      - name: Print event type
        run: echo "Triggered by event: ${{ github.event.action }}"

      - name: Read custom payload
        run: echo "Severity is ${{ github.event.client_payload.severity }}"
```

**Triggering from an external system via curl:**

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/dispatches \
  -d '{"event_type": "incident_report", "client_payload": {"severity": "high", "service": "payment-api"}}'
```

> **Requirements:** The request must be authenticated with a **Personal Access Token (PAT)** or GitHub App token that has `repo` scope. The `event_type` value must match at least one of the strings in the `types:` list of the target workflow.

---

## 10) Jobs and Steps in Detail

### Understanding Job Dependency Graphs

When a workflow has multiple jobs, GitHub Actions builds an internal dependency graph based on the `needs:` declarations and uses it to determine the execution order.

A job with no `needs:` declaration is a root node — it starts immediately when the workflow is triggered. A job with `needs: [job_a, job_b]` is a leaf node relative to those two jobs — it will not start until both `job_a` and `job_b` have completed successfully.

This allows you to express complex pipelines like:

- Run tests on multiple platforms in parallel → only deploy if all pass
- Build a binary → run unit tests and integration tests simultaneously → publish if both pass

**Job ordering example:**

```yaml
on:
  workflow_dispatch:

jobs:
  job_1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running ${{ github.job }}"

  job_2:
    runs-on: ubuntu-latest
    needs: job_1          # Waits for job_1
    steps:
      - run: echo "Running ${{ github.job }}"

  job_3:
    runs-on: ubuntu-latest
    needs: job_1          # Also waits for job_1 — runs in parallel with job_2
    steps:
      - run: echo "Running ${{ github.job }}"

  job_4:
    runs-on: ubuntu-latest
    needs: [job_2, job_3] # Waits for BOTH job_2 and job_3
    steps:
      - run: echo "Running ${{ github.job }}"
```

Execution flow: `job_1` → `job_2` and `job_3` (parallel) → `job_4`. This diamond shape is a very common CI/CD pattern.

---

### Specifying a Shell

**Why this matters:**

The default shell for `run:` steps varies by runner OS:

- Linux and macOS: `bash`
- Windows: `pwsh` (PowerShell Core)

If you're running Windows jobs and write bash-style commands, they will fail. Conversely, PowerShell syntax won't work on Linux runners. Being explicit about the shell removes this ambiguity, especially in cross-platform workflows.

The `{0}` placeholder in custom shell values is replaced at runtime with the path to a temporary script file containing your `run:` command.

```yaml
on:
  workflow_dispatch:

jobs:
  shell-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run default bash
        run: echo "This runs in bash by default"

      - name: Multi-line script with explicit shell and working directory
        run: |
          npm install
          npm run build
        working-directory: /tmp
        shell: bash

      - name: Run a Perl one-liner
        run: print "Hello from Perl\n"
        shell: perl {0}
```

**Available shells:**

|Shell|`shell:` value|
|---|---|
|Bash|`bash`|
|PowerShell Core|`pwsh`|
|Python|`python`|
|POSIX sh|`sh`|
|Windows Command Prompt|`cmd` (Windows only)|
|Custom (any interpreter)|`perl {0}`, `ruby {0}`, `python {0}`, etc.|

---

### Running Steps Inside Docker Containers

**Why run inside a container?**

The runner gives you a VM with pre-installed software. But sometimes you need a very specific environment — a specific version of an interpreter, a specific system library, or a completely isolated execution context. Docker containers let you define that environment precisely and use it in your workflow steps.

**Two levels of container use:**

1. **Step-level** (`uses: docker://image`): Only that single step runs in the container. The runner VM handles all other steps.
2. **Job-level** (`container:` key on the job): The entire job — all its steps — runs inside the container. The runner VM is just the host.

**Important constraint:** Step-level container execution only works with **public** images. The runner can't authenticate with a private registry for step-level execution. For private images, you must use job-level container configuration with explicit credentials.

**Step-level container (public images only):**

```yaml
jobs:
  hybrid-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run on the Ubuntu VM
        run: echo "This step runs on the VM directly"

      - name: Run inside an Alpine container
        uses: docker://alpine:3.14
        with:
          entrypoint: /bin/sh
          args: -c "echo 'This runs inside the Alpine container'"
```

**Job-level container (supports private images):**

```yaml
jobs:
  dockerized_job:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/my-org/my-private-image:latest
      credentials:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    steps:
      - run: node --version
      - run: echo "Every step here runs inside the private container"
```

**Line-by-line explanation of the private container job:**

- `runs-on: ubuntu-latest` — GitHub provisions an Ubuntu VM as the **host machine** for this job.
- `container:` — Tells GitHub Actions to launch a Docker container on top of that host and run all job steps inside the container, not on the host.
- `image:` — The Docker image to pull and run. This one is hosted on GitHub Container Registry (`ghcr.io`).
- `credentials:` — The username and password to authenticate with the container registry. Required for private images.
- `username/password` — These come from **GitHub Secrets** (encrypted, not visible in logs). Never hard-code credentials.
- `steps:` — Every step in this job executes inside the container. The steps cannot tell the difference between running in a container and running on the VM directly.

---

## 11) Matrix Strategy

### The Problem Matrix Solves

Imagine you need to test your application on Ubuntu, Windows, and macOS, and across Node.js versions 16, 18, and 20. Without matrix strategy, you'd write 9 nearly-identical jobs — `test-ubuntu-16`, `test-ubuntu-18`, `test-ubuntu-20`, `test-windows-16`, and so on. That's 9 times the YAML to maintain. If you add a new Node.js version, you'd have to add 3 more jobs.

Matrix strategy solves this by letting you define the variable dimensions (OS and Node version) and having GitHub automatically generate all the combinations and run them as parallel jobs.

---

### How Matrix Works Internally

When GitHub reads a workflow with a `strategy.matrix` definition, it expands the matrix into a set of jobs before execution begins. Each combination of matrix values becomes an independent job. These jobs can run in parallel (subject to the runner availability limits on your account).

The matrix variables you define become available inside the job via `${{ matrix.variable_name }}`. They work exactly like regular variables at that point.

---

### Why Matrix Is One of the Most Valuable Features

**Testing cross-platform compatibility:** If your application claims to work on Linux, Windows, and macOS, you need to actually test it on all three. A bug that only appears on Windows (like a case-sensitive file path) will slip through if you only run CI on Linux.

**Testing across language versions:** A Python 3.10 feature might not exist in Python 3.8. Testing both ensures your application works for users on older runtime versions.

**Time efficiency:** Matrix jobs run in parallel. 9 jobs that each take 5 minutes = 5 minutes total (not 45 minutes), assuming enough runners are available.

---

### Matrix Examples

**Single dimension — multiple OS:**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v5
      - run: echo "Running on ${{ matrix.os }}"
```

Creates 3 parallel jobs.

**Single dimension — multiple language versions:**

```yaml
strategy:
  matrix:
    dotnet-version: ['8.0', '9.0']

steps:
  - uses: actions/setup-dotnet@v4
    with:
      dotnet-version: ${{ matrix.dotnet-version }}
```

**Multiple dimensions — all combinations generated automatically:**

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    dotnet-version: ['8.0', '9.0']
```

GitHub generates:

|OS|.NET Version|
|---|---|
|ubuntu-latest|8.0|
|ubuntu-latest|9.0|
|windows-latest|8.0|
|windows-latest|9.0|

4 jobs from 2×2 matrix values.

---

### Full Matrix Example with Options

```yaml
on:
  workflow_dispatch:

jobs:
  matrix-test:
    strategy:
      fail-fast: false        # Let all combinations run even if one fails
      max-parallel: 3         # Cap at 3 simultaneous jobs
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node_version: [14.x, 16.x, 18.x]   # 9 total combinations

    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node_version }}

      - name: Print matrix combination
        run: echo "Testing Node ${{ matrix.node_version }} on ${{ matrix.os }}"
```

**Key matrix options:**

|Option|Default|Description|
|---|---|---|
|`fail-fast`|`true`|When `true`, GitHub cancels all remaining matrix jobs as soon as one fails. Set to `false` to let all jobs run and see the full picture of failures.|
|`max-parallel`|unlimited|Maximum number of matrix jobs to run simultaneously. Useful for rate-limited resources (a staging database, an API with request limits).|

> **Important:** `os` and `node_version` are names you choose yourself. They are not GitHub Actions keywords. You can name matrix variables anything descriptive.

---

## 12) Conditionals and Expressions

### Why Conditions Are Essential

A single workflow file often needs to behave differently based on context. The deploy step should only run on the main branch. The release notes step should only run when a version tag is pushed. A notification step should only run if the previous step failed. Heavy tests should be skipped for work-in-progress commits.

Without conditional logic, you'd need a separate workflow file for each scenario. With `if:` and expressions, one workflow file can handle many scenarios intelligently.

---

### Expression Syntax

Expressions in GitHub Actions are written inside `${{ }}`. Inside these delimiters, you have access to:

- **Context objects** — structured data about the workflow run, the event, the repository, and more
- **Functions** — built-in functions for string manipulation, type checks, and status checks
- **Operators** — comparison and logical operators

In `if:` conditions specifically, the `${{ }}` wrapper is optional (GitHub evaluates the entire `if:` value as an expression). But including it is clearer and more explicit.

---

### Common Operators

| Operator           | Syntax         | Example                                                              |
| ------------------ | -------------- | -------------------------------------------------------------------- |
| Equality           | `==`           | `github.event_name == 'push'`                                        |
| Inequality         | `!=`           | `github.ref != 'refs/heads/main'`                                    |
| Logical AND        | `&&`           | `github.event_name == 'push' && github.ref == 'refs/heads/main'`     |
| Logical OR         | `\|`           | `github.event_name == 'push' \| github.event_name == 'pull_request'` |
| Negation           | `!`            | `!contains(github.event.head_commit.message, 'WIP')`                 |
| String contains    | `contains()`   | `contains(github.ref, 'release/')`                                   |
| String starts with | `startsWith()` | `startsWith(github.ref, 'refs/tags/')`                               |
| String ends with   | `endsWith()`   | `endsWith(github.ref, '-beta')`                                      |

---

### Important Context Objects

**`github.ref`** — The full Git reference that triggered the workflow. For branch pushes, this is `refs/heads/<branch-name>`. For tag pushes, this is `refs/tags/<tag-name>`. Use this when you want to control execution based on which branch or tag was pushed.

**`github.event_name`** — The name of the event that triggered the workflow (`push`, `pull_request`, `workflow_dispatch`, etc.). Use this when one workflow handles multiple event types and you need to behave differently for each.

**`github.event`** — The complete payload of the triggering event as a JSON object. The contents depend on the event type. For a push event, `github.event.head_commit.message` gives you the commit message. For a pull_request event, `github.event.pull_request.labels` gives you the PR's labels.

**`github.job`** — The ID of the currently running job.

**`inputs`** — The values of `workflow_dispatch` inputs provided by the user.

**`matrix`** — The current combination of matrix values in a matrix job.

**`secrets`** — Encrypted values from repository, organization, or environment secrets.

**`env`** — Environment variables set in the workflow.

---

### Using `github.ref`

```yaml
name: CI/CD Pipeline

on: [push]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v5

      - name: Build Application
        run: echo "Building for all branches..."

      # Only runs when code is pushed to the main branch specifically
      - name: Deploy to Production
        if: ${{ github.ref == 'refs/heads/main' }}
        run: echo "Deploying to production!"

      # Only runs when a tag starting with 'v1' is pushed (e.g., v1.0.0, v1.2.3)
      - name: Publish Release Notes
        if: ${{ startsWith(github.ref, 'refs/tags/v1') }}
        run: echo "Publishing release for tag: ${{ github.ref }}"
```

---

### Using `github.event`

```yaml
name: Code Quality Checks

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v5

      # If the commit message contains "WIP" (Work in Progress), skip heavy tests
      - name: Run Unit Tests
        if: ${{ !contains(github.event.head_commit.message, 'WIP') }}
        run: echo "Running full test suite..."

      # Only triggers when this is a pull_request event AND the PR has a 'security' label
      - name: Notify Security Team
        if: ${{ github.event_name == 'pull_request' && contains(github.event.pull_request.labels.*.name, 'security') }}
        run: echo "Sending alert: security-labeled PR was updated!"
```

> **Note:** `github.event.head_commit.message` is only available on `push` events. On `pull_request` events, that field doesn't exist. When your workflow handles multiple event types, be careful about which event context fields you reference.

---

### Status Check Functions

These functions check the outcome of **all previous steps** (or a specific previous step, using `steps.<id>.outcome`). They are used to run cleanup steps, failure notifications, or conditional follow-up actions.

| Function      | When it resolves to `true`                                                                                                                                     |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `success()`   | All previous steps in this job completed successfully. **This is the implicit default** — steps without an `if:` only run if everything before them succeeded. |
| `failure()`   | At least one previous step failed.                                                                                                                             |
| `always()`    | Always runs, regardless of any previous outcomes. Used for cleanup and notification steps.                                                                     |
| `cancelled()` | The workflow run was cancelled by a user.                                                                                                                      |

```yaml
steps:
  - name: Run Tests
    run: npm test

  # This step runs even if the test step failed — always send a report
  - name: Upload Test Report
    if: always()
    run: echo "Uploading test results..."

  # This step only runs if tests failed — sends an alert to the team
  - name: Notify Team of Failure
    if: failure()
    run: echo "Tests failed! Sending Slack notification..."

  # This step only runs if everything succeeded
  - name: Deploy to Staging
    if: success()
    run: echo "All tests passed. Deploying to staging..."
```

---

## 13) Logging and Annotations

### What Are Workflow Commands?

GitHub Actions runners watch for **special strings** printed to standard output during a step. When the runner detects one of these strings, it interprets it as a command rather than regular log output. These are called **workflow commands**.

Workflow commands let you communicate information from inside a running step back to the Actions framework — without needing a special library or SDK. Any step in any language can use them by simply printing the right string to stdout.

---

### Why This Matters

Standard log output just appears as a line of text in the step log. Workflow commands, on the other hand, create **UI annotations** — visible badges and highlighted messages in the workflow run summary page in the GitHub UI. This makes important warnings and errors much more visible to the person reviewing the workflow run.

Critically, `::warning::` and `::error::` annotations do **not** fail the step. The step continues running. The annotations are surfaced in the UI, but execution proceeds. This lets you report problems for informational purposes without interrupting the workflow.

---

### Log Commands

```bash
echo "::warning::This function is deprecated and will be removed in v3.0"
echo "::error::config.yaml not found. Cannot proceed."
echo "::debug::Variable foo has value: bar"    # Only visible when debug logging is enabled in the repo settings
```

- `::warning::` — Creates a yellow warning annotation in the GitHub UI
- `::error::` — Creates a red error annotation in the GitHub UI
- `::debug::` — Writes a debug message that is hidden by default (enable it in the repository Actions settings for troubleshooting)

You can also attach file location metadata to annotations:

```bash
echo "::error file=src/app.js,line=42,col=10::Null pointer exception"
```

This creates a red annotation that links directly to line 42, column 10 of `src/app.js` in the GitHub UI code viewer.

---

### Setting Environment Variables Between Steps

Each step in a job runs in a separate process. Normal environment variables set in one step's shell are not visible to subsequent steps. To pass values between steps, you write to a special file that GitHub Actions reads between steps:

```bash
echo "MY_VARIABLE=hello" >> $GITHUB_ENV
```

After this step completes, the next step in the same job can read `$MY_VARIABLE` normally.

This is the correct, official way to share state between steps within a job.

---

### Masking Secrets from Logs

When you reference a secret with `${{ secrets.MY_SECRET }}`, GitHub automatically masks the value in log output, replacing it with `***`. But what about values you construct at runtime (not from `secrets` directly)?

For those, you use the mask command:

```bash
echo "::add-mask::$COMPUTED_VALUE"
```

After this line executes, any occurrence of `$COMPUTED_VALUE` in the step logs from that point forward will be replaced with `***`.

**Important:** The mask only applies from the line where you call `::add-mask::` onwards. Any output that already happened before the mask command will not be redacted.

---

### Full Logging Example

```yaml
on:
  workflow_dispatch:

jobs:
  logging-demo:
    runs-on: ubuntu-latest
    steps:
      - name: Check for Deprecated Code
        run: |
          deprecated="true"
          if [[ "$deprecated" == "true" ]]; then
            echo "::warning::A deprecated function was found. Please migrate before v3.0."
          fi

      - name: Set API Key in Environment
        run: echo "API_KEY=12345" >> $GITHUB_ENV

      - name: Use the API Key
        run: echo "The API key is $API_KEY"

      - name: Check Config File Exists
        run: |
          if [ -f "config.yaml" ]; then
            echo "::debug::config.yaml found."
          else
            echo "::error::config.yaml does not exist. Workflow cannot continue."
          fi

      - name: Mask the API Key Before Printing
        run: |
          echo "::add-mask::$API_KEY"
          echo "Using API key: $API_KEY"    # Prints: Using API key: ***
```

>`if [[ "$deprecated" == "true" ]]`. The original condition tests whether the literal string `deprecated` is non-empty — which it always is, making the condition always `true` regardless of what value you assigned to the variable. The corrected version tests the actual value of the `$deprecated` variable.

---

## 14) Secrets and Secure Data

### What Secrets Are?

 A secret is a secure and encrypted value stored in GitHub. It is commonly used to store sensitive information such as passwords, API keys, and access tokens. Workflows can access secrets during execution, but their actual values are hidden to protect security.

### Why They Exist

Without secrets, developers would be tempted to paste API keys, passwords, and tokens directly into workflow YAML files. These files are in the repository. Even in private repositories, this is a significant security risk — anyone with repository access can read the file.

Secrets provide a secure, centralized store for sensitive values. The workflow file only references the secret by name (`${{ secrets.MY_SECRET }}`), not by value. Even someone who can read the workflow file cannot see the actual secret value.

### What to Store as Secrets

- API keys and access tokens
- Container registry credentials (Docker Hub, GHCR)
- Cloud provider credentials (AWS, Azure, GCP)
- Database passwords
- SSH private keys
- Signing certificates

### Using Secrets

```yaml
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

steps:
  - name: Deploy to AWS
    run: aws s3 sync ./dist s3://my-bucket

  # Pass a secret directly to a third-party action
  - name: Send Slack notification
    uses: slackapi/slack-github-action@v1
    with:
      slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Best Practices for Secrets

- Never print a secret value with `echo` (GitHub will mask it, but it's still bad practice)
- Rotate secrets regularly
- Use environment-level secrets for deployment secrets that should only be accessible for specific environments (staging, production)
- Use the minimum-privilege principle — create tokens with only the permissions they need

---

# 15) Understanding Variables, Contexts, Secrets, and Runtime Data Flow in GitHub Actions

One of the most confusing parts of GitHub Actions for beginners is understanding how data moves through a workflow.

At first glance, many things seem to do the same job:

- `env`
    
- `vars`
    
- `secrets`
    
- `${{ }}`
    
- `$GITHUB_ENV`
    
- `$GITHUB_OUTPUT`
    
- `$GITHUB_*`
    

As a result, developers often ask questions such as:

- Why can I use `$NAME` in one place and `${{ env.NAME }}` in another?
    
- Why doesn't `export` persist between steps?
    
- What is `$GITHUB_ENV` actually doing?
    
- When should I use a Secret instead of an Environment Variable?
    
- Why does GitHub provide both `github.repository` and `GITHUB_REPOSITORY`?
    

To answer these questions, we first need to understand how GitHub Actions executes a workflow.

---
#### How GitHub Actions Actually Works

Most people imagine GitHub Actions as a simple script runner.

In reality, there are multiple stages involved before your commands execute.

When a workflow starts, GitHub does not immediately run your shell commands.

Instead, it follows a process similar to this:

```text
Workflow File
      ↓
GitHub Parses YAML
      ↓
GitHub Evaluates Expressions
      ↓
Runner Starts
      ↓
Shell Executes Commands
      ↓
GitHub Processes Runtime Files
```

Understanding this execution flow explains nearly every variable-related behavior in GitHub Actions.

---

#### The Three Execution Layers

A useful mental model is to think of GitHub Actions as operating in three distinct layers.

```text
┌─────────────────────────────┐
│ Layer 1: GitHub Engine      │
│ Expression Evaluation       │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Layer 2: Runner Environment │
│ Environment Variables       │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Layer 3: Communication      │
│ GITHUB_ENV / OUTPUT Files   │
└─────────────────────────────┘
```

Let's examine each layer.

---
#### Layer 1: GitHub Expression Evaluation

Before a runner is started, GitHub evaluates expressions.

Expressions use this syntax:

```yaml
${{ ... }}
```

Examples:

```yaml
${{ github.repository }}
${{ github.actor }}
${{ github.sha }}
${{ env.APP_NAME }}
${{ vars.API_URL }}
${{ secrets.API_KEY }}
```

These expressions are resolved by GitHub itself.

For example:

```yaml
run: echo "${{ github.repository }}"
```

may become:

```yaml
run: echo "octocat/demo-repository"
```

before the runner even starts.

This means the shell never sees:

```yaml
${{ github.repository }}
```

It only sees:

```yaml
octocat/demo-repository
```

---
#### Layer 2: Runner Environment Variables

After GitHub finishes evaluating expressions, a runner machine starts.

The runner receives environment variables such as:

```bash
GITHUB_REPOSITORY
GITHUB_SHA
GITHUB_ACTOR
```

and any custom environment variables you define.

Example:

```yaml
env:
  APP_NAME: MyApp
```

Inside a shell:

```yaml
run: echo $APP_NAME
```

The value is resolved by Bash (or PowerShell), not by GitHub.

This is why:

```yaml
$APP_NAME
```

and

```yaml
${{ env.APP_NAME }}
```

may produce the same result while being processed by completely different systems.

---
#### Layer 3: Runtime Communication

Each step runs independently.

Because of this, GitHub provides special files that allow steps to communicate.

Examples:

```text
$GITHUB_ENV
$GITHUB_OUTPUT
```

These files become extremely important when passing data between steps.

We will discuss them later.

---
#### What Is an Environment Variable?

An environment variable is a temporary value available during workflow execution.

Example:

```yaml
env:
  APP_NAME: MyApp
```

Usage:

```yaml
run: echo $APP_NAME
```

Output:

```text
MyApp
```

Environment variables exist only while the workflow is running.

Once the workflow finishes, they disappear.

---

#### Environment Variable Scopes

GitHub Actions allows environment variables to be defined at different scopes.

Think of scopes as visibility boundaries.

---

#### Workflow-Level Environment Variables

Available everywhere in the workflow.

```yaml
name: Demo

on:
  workflow_dispatch

env:
  APP_NAME: MyApp

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo $APP_NAME

  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo $APP_NAME
```

Both jobs can access the variable.

---
#### Job-Level Environment Variables

Available only inside a specific job.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    env:
      APP_NAME: MyApp

    steps:
      - run: echo $APP_NAME
```

Other jobs cannot access it.

---
#### Step-Level Environment Variables

Available only inside one step.

```yaml
steps:
  - name: Example
    env:
      APP_NAME: MyApp

    run: echo $APP_NAME
```

The next step will not see this variable.

---
#### Environment Variable Precedence

What happens if the same variable exists in multiple scopes?

GitHub uses the most specific scope.

Priority order:

```text
Step
 ↓
Job
 ↓
Workflow
```

Example:

```yaml
env:
  NAME: Workflow

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      NAME: Job

    steps:
      - env:
          NAME: Step

        run: echo $NAME
```

Output:

```text
Step
```

The closest definition wins.

---
#### Accessing Environment Variables

There are two common ways to access environment variables.

---
#### Method 1: Shell Syntax

Used inside `run`.

```yaml
run: echo $APP_NAME
```

or

```yaml
run: echo ${APP_NAME}
```

The shell resolves the value during execution.

---
#### Method 2: Expression Syntax

```yaml
${{ env.APP_NAME }}
```

Example:

```yaml
run: echo "${{ env.APP_NAME }}"
```

GitHub resolves this value before execution begins.

---
#### Understanding GitHub Contexts

Contexts are structured objects exposed by GitHub.

Examples:

```yaml
${{ github.repository }}
${{ github.actor }}
${{ github.sha }}
${{ env.APP_NAME }}
${{ vars.API_URL }}
${{ secrets.API_KEY }}
```

Common contexts:

|Context|Purpose|
|---|---|
|github|Workflow metadata|
|env|Environment variables|
|vars|Repository variables|
|secrets|Encrypted values|
|runner|Runner information|
|job|Job information|
|steps|Step outputs|

Think of contexts as data sources available to GitHub during expression evaluation.

---
#### Built-In GitHub Variables

GitHub automatically provides useful environment variables.

Examples:

```bash
GITHUB_REPOSITORY
GITHUB_SHA
GITHUB_REF
GITHUB_ACTOR
GITHUB_RUN_ID
GITHUB_WORKSPACE
```

Usage:

```yaml
run: echo $GITHUB_REPOSITORY
```

Example output:

```text
octocat/demo-repository
```

---
#### Context vs Environment Variable

GitHub often exposes the same information in two forms.

|Context|Runtime Variable|
|---|---|
|`${{ github.repository }}`|`$GITHUB_REPOSITORY`|
|`${{ github.sha }}`|`$GITHUB_SHA`|
|`${{ github.actor }}`|`$GITHUB_ACTOR`|

The difference is when they are evaluated.

### Context

```yaml
${{ github.repository }}
```

Evaluated by GitHub before execution.

### Runtime Variable

```bash
$GITHUB_REPOSITORY
```

Evaluated by the shell during execution.

---
#### Repository Variables (vars)

Repository Variables are persistent configuration values stored by GitHub.

Location:

```text
Repository Settings
    ↓
Secrets and Variables
    ↓
Actions
    ↓
Variables
```

Usage:

```yaml
${{ vars.API_URL }}
```

Characteristics:

- Persistent
    
- Shared across workflows
    
- Not encrypted
    
- Good for configuration values
    

Example:

```yaml
run: echo "${{ vars.API_URL }}"
```

Use Repository Variables for values that rarely change and are not sensitive.

---

#### Secrets

Secrets are encrypted values stored by GitHub.

Location:

```text
Repository Settings
    ↓
Secrets and Variables
    ↓
Actions
    ↓
Secrets
```

Usage:

```yaml
${{ secrets.API_KEY }}
```

Example:

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}

steps:
  - run: echo $API_KEY
```

Characteristics:

- Encrypted
    
- Hidden from logs
    
- Persistent
    
- Secure

Secrets should be used for:

- API Keys
    
- Access Tokens
    
- Passwords
    
- Connection Strings
    
- Private Credentials
    

---

#### Why Export Does Not Work Across Steps

Many developers expect this workflow to work:

```yaml
steps:
  - run: |
      export VERSION=1.0

  - run: |
      echo $VERSION
```

Output:

```text
(empty)
```

Why?

Because every step runs in a separate shell process.

```text
Step 1 → Shell A

Step 2 → Shell B
```

Variables created inside Shell A disappear when Shell A exits.

Shell B starts fresh.

---
#### What Is $GITHUB_ENV?

A common misconception is that `$GITHUB_ENV` is an environment variable.

It is not.

`$GITHUB_ENV` is a file path created by GitHub.

Example:

```bash
echo "VERSION=1.0" >> $GITHUB_ENV
```

Internally:

```text
Step Executes
      ↓
Write Data To File
      ↓
Step Finishes
      ↓
GitHub Reads File
      ↓
Creates Environment Variables
      ↓
Next Steps Receive Values
```

This mechanism allows data to survive beyond the current step.

---

### Using $GITHUB_ENV

Example:

```yaml
steps:
  - run: |
      echo "VERSION=1.0" >> $GITHUB_ENV

  - run: |
      echo $VERSION
```

Output:

```text
1.0
```

This works because GitHub injects the variable into future steps.

---

#### env vs $GITHUB_ENV

These concepts are related but not identical.

|Feature|env|$GITHUB_ENV|
|---|---|---|
|Static|Yes|No|
|Dynamic|No|Yes|
|Available Immediately|Yes|No|
|Available In Future Steps|Yes|Yes|

### Use `env` when:

The value is already known.

```yaml
env:
  APP_NAME: MyApp
```

#### Use `$GITHUB_ENV` when:

The value is generated during execution.

```bash
echo "BUILD_ID=$(date +%s)" >> $GITHUB_ENV
```

---

#### What Is $GITHUB_OUTPUT?

`$GITHUB_OUTPUT` is used to expose structured outputs from a step.

Example:

```yaml
- id: build
  run: |
    echo "version=1.0.0" >> $GITHUB_OUTPUT
```

Reading the output:

```yaml
${{ steps.build.outputs.version }}
```

Complete example:

```yaml
steps:
  - id: build
    run: |
      echo "version=1.0.0" >> $GITHUB_OUTPUT

  - run: |
      echo "${{ steps.build.outputs.version }}"
```

Output:

```text
1.0.0
```

---

#### When Should You Use $GITHUB_OUTPUT?

Use `$GITHUB_OUTPUT` when:

- One step produces a result
    
- Another step consumes that result
    
- The value represents an output rather than an environment variable
    

Think of it as a function return value.

```text
Function
   ↓
Returns Result
   ↓
Caller Uses Result
```

This is the same idea.

---

#### Runtime Data Lifecycle

Understanding data lifetime is important.

```text
Workflow Starts
       ↓
GitHub Evaluates Expressions
       ↓
Runner Starts
       ↓
Environment Variables Created
       ↓
Steps Execute
       ↓
GITHUB_ENV Updates Variables
       ↓
GITHUB_OUTPUT Creates Outputs
       ↓
Workflow Ends
       ↓
Runtime Data Destroyed
```

---

#### Temporary vs Persistent Data

##### Temporary Data

Destroyed after workflow execution:

- Workflow env
    
- Job env
    
- Step env
    
- GITHUB_ENV values
    
- GITHUB_OUTPUT values
    

---

#### Persistent Data

Stored by GitHub:

- Repository Variables
    
- Organization Variables
    
- Secrets
    
- Environment Secrets
    



---

## 16) Conclusion 

- Always use `actions/checkout` as the first step whenever your job needs repository files.

**Trigger design:**

- Use `push` and `pull_request` for CI validation.

- Use `paths:` filters to avoid running CI on documentation or configuration changes that don't affect the application.

- Use `workflow_dispatch` for anything that should only run when a human decides it should.

- Use `repository_dispatch` for integrations with external systems.

- Use `schedule` for routine, time-based automation.

  

**YAML writing:**

- Use single quotes for literal strings, regex patterns, and file paths with backslashes.

- Use double quotes only when you need `\n`, `\t`, or other escape sequences.

- Quote any value that could be auto-converted by the YAML parser (`yes`, `no`, `true`, `false`, version numbers like `1.10`).

  

**Security:**

- Store all sensitive values in GitHub Secrets.

- Reference secrets with `${{ secrets.NAME }}` — never hard-code them in workflow files.

- Use `::add-mask::` for sensitive values computed at runtime.

- Audit the permissions of third-party actions before using them.

  

**Job structure:**

- Use `needs:` to enforce job ordering when dependencies exist.

- Use `if:` conditions to skip unnecessary steps based on branch, event type, or commit message.

- Use `fail-fast: false` in matrix jobs when you want full visibility into all failures.

- Use `always()` for cleanup steps that must run even when the job fails.

  

**Action versioning:**

- Pin to a major version tag (`@v4`, `@v5`) for the right balance of stability and automatic security updates.

- For highly sensitive production workflows, consider pinning to a full commit SHA for maximum predictability.

  

---

GitHub Actions is a flexible, deeply integrated automation platform that grows with your team. Starting with a simple CI pipeline and expanding into deployment automation, issue management, scheduled maintenance, and external integrations is a natural progression.


The most important concepts to internalize:


1. The **hierarchy** — workflow → job → step — and which level controls what.

2. The **parallel/sequential rule** — jobs parallel by default, steps always sequential.

3. The **code checkout requirement** — the workspace is empty until you check out.

4. **Triggers** — matching the right trigger type to the right use case.

5. **Conditionals** — using `if:`, `github.ref`, `github.event`, and status functions to make workflows smart.

6. **Secrets** — never putting sensitive data in workflow files.

  

Once these fundamentals are solid, the rest is practice, experimentation, and reading the documentation for specific use cases as they arise.


----
- ## وفي النهاية يا صديقي، اتمنى اكون قدرت اوصل المعلومة بشكل بسيط وواضح، واكون فعلا افدتك بجد واستفدت معايا خطوة بخطوة في فهم الموضوع. ولو في أي جزء لسه مش واضح او حاسس إنه محتاج شرح اكتر، متتكسفش خالص انك تتواصل معايا في اي وقت، وانا موجود وهكمل معاك لحد ما الصورة تبقى كاملة بإذن الله. ولو انت استفدت حتى بنسبة بسيطة، فده بالنسبة ليا اهم من اي حاجة، ومتشكر جدا على وقتك وتركيزك، وياريت تدعيلي دعوة جميلة زيك كدا ولك بالمثل بإذن الرحمن وربنا يوفقك دايما في اللي جاي 

##


- ## واتفكر دايما ان ربنا كريم اوي وعطاياها لاتحصي.. ماعليك غير انك تسعي وتكمل سعي وتحمد ربنا علي نعمه وتتوكل عليه ولازم تتيقن انك بتتوكل علي ربنا وبس كدا ياصديقي ربنا يوفقك ويعينك يارب والي لقاء اخر ان شاء الله 


###  Connect with me :- 

 <a href="https://www.linkedin.com/in/ahmed-zaher-62a652255/" target="_blank">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/linkedin/linkedin-original.svg" width="20"/>
   Ahmed Zaher
</a>

 <a href="https://x.com/AhmedZaher882" target="_blank">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/twitter/twitter-original.svg" width="20"/>
   Ahmed Zaher
</a>

<a href="https://wa.me/201158905589" target="_blank">
  <img src="https://img.icons8.com/color/48/whatsapp--v1.png" width="25" height="25" />
  +201158905589
</a>

 <a href="https://www.facebook.com/ahmed.t.zaher.2025" target="_blank">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/facebook/facebook-original.svg" width="20"/>
	   Ahmed T. Zaher
</a>

 <a href="https://github.com/ahmed-tarek-2004" target="_blank">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/github/github-original.svg" width="20"/> Ahmed Zaher
</a>

