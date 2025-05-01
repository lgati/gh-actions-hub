## 🛠️ TE Help GitHub Actions Workflow Suite

This workflow suite enables Test Engineering (TE) to be paged via GitHub PR comments (`/te-help`) and automates Jira ticket creation, Slack notifications, and comment updates — all from a single command.

---

### 🚀 How to Use This (Quick Start)

#### ✅ 1. Configure the Trigger Workflow
Copy `trigger-te-help.yml` to the target repository.
This listens to PR comment events and triggers the central workflow.

```yaml
on:
  issue_comment:
    types: [created]

jobs:
  call-te-help-workflow:
    if: startsWith(github.event.comment.body, '/te-help')
    uses: businessinsider/insider-gh-test-eng-workflows/.github/workflows/te-help-workflow.yml@main
    with:
      pr_number: ${{ github.event.issue.number }}
      comment_id: ${{ github.event.comment.id }}
      comment_author: ${{ github.event.comment.user.login }}
      jira_base_url: https://businessinsider.atlassian.net
      jira_user_email: lgati@insider.com
    secrets:
      GITHUBACTIONSTOKEN, JIRA_API_TOKEN, DD_API_KEY, DD_APP_KEY
```

#### ✅ 2. Comment Format

```
/te-help [--mock] [--high] [--force-jira] [notes]
```

**Available Flags:**
- `--mock` → dry-run mode
- `--high` → marks as high priority
- `--force-jira` → triggers Jira even in mock mode

**Examples:**
- `/te-help Just adding notes to the ticket`
- `/te-help --high This is urget`
- `/te-help --high --mock Dry run but urgent`

#### ✅ 3. What It Does
- Parses comment and extracts flags and notes
- Fetches failed PR checks (Argos and Currents apps)
- Conditionally creates a Jira ticket
- Sends a priority-aware Slack notification
- Updates the PR comment
- Adds a `te-help-requested` label

---

### 🔁 **Trigger Workflow**: `Trigger TE Help`

**File**: `.github/workflows/trigger-te-help.yml`

**Trigger**:
- `issue_comment.created`: If comment starts with `/te-help`

**What it does**:
- Conditionally triggers the `TE Help Workflow` using `workflow_call`
- Passes PR context, author, and flag values
- Grants necessary permissions
- Forwards Jira and Datadog secrets

---

### ⚙️ **Main Workflow**: `TE Help Workflow`

**File**: `.github/workflows/te-help-workflow.yml`

**Inputs**:
- `pr_number`, `comment_id`, `comment_author`, `jira_base_url`, `jira_user_email`, `mock_mode`

**Secrets**:
- Jira and Datadog API credentials

**Jobs**:

1. **📝 update-comment-before**  
   Adds 👀 reaction and a message saying the request is being processed

2. **⚙️ fetch-workflow-config**  
   Parses flags like `--mock`, `--high`, `--force-jira`

3. **👩‍💻 get-oncall**  
   Calls `get-te-oncall.yml` to fetch or mock current on-call TE engineer via Datadog API

4. **📄 create-jira-ticket**  
   Calls `create-te-jira.yml` to create a Jira incident ticket and assign it to the on-call engineer

5. **📣 notify-slack**  
   Calls `notify-te-slack.yml` to send a message to Slack using a derived or default channel/user

6. **✏️ update-comment-after**  
   Updates the original comment with Jira ticket info and switches 👀 to 🚀

7. **🏷️ add-label-to-pr**  
   Adds a `te-help-requested` label via `add-label-to-pr.yml`

---

### 📬 Sub-Workflow: `Get On-Call Engineer`

**File**: `.github/workflows/get-te-oncall.yml`

**Purpose**:
- Queries Datadog to get current on-call engineer’s email or returns a mocked response

---

### 🧾 Sub-Workflow: `Create Jira Ticket`

**File**: `.github/workflows/create-te-jira.yml`

**Purpose**:
- Creates a Jira ticket in the `TE` project
- Assigns it to the on-call email
- Outputs ticket ID and URL
- Supports `mock_mode` and `--force-jira`

---

### 🔔 Sub-Workflow: `Notify TE on Slack`

**File**: `.github/workflows/notify-te-slack.yml`

**Purpose**:
- Posts a message to Slack with repo, PR, and Jira info
- Dynamically determines Slack handle from email
- Adjusts styling and color based on `--high` flag

---

### 🏷️ Sub-Workflow: `Add Label to PR`

**File**: `.github/workflows/add-label-to-pr.yml`

**Purpose**:
- Ensures label exists by calling `create-label.yml`
- Adds it to the PR

---

### 🧰 Additional Reusable Workflows

- **`fetch-workflow-config.yml`** – Parses comment flags and notes
- **`fetch-failures.yml`** – Gathers PR failure context for Slack and Jira
- **`update-comment-before.yml`** – Adds 👀 emoji and "processing" message
- **`update-comment-after.yml`** – Adds 🚀 emoji and Jira badge
- **`create-label.yml`** – Ensures label existence with color and description

---

### 📁 File Map

```
.github/workflows/
├── trigger-te-help.yml      # Trigger from target repo
├── te-help-workflow.yml            # Main orchestration logic
├── fetch-workflow-config.yml       # Parses comment flags
├── fetch-failures.yml              # Gets PR check results
├── get-te-oncall.yml               # On-call logic
├── create-te-jira.yml              # Jira issue creator
├── notify-te-slack.yml             # Slack message sender
├── update-comment-before.yml       # Adds initial comment status
├── update-comment-after.yml        # Posts Jira badge and final update
├── add-label-to-pr.yml             # Calls label workflow + applies label
└── create-label.yml                # Creates label if missing
```

---

Maintainer: @lucasgatti  
Built to streamline TE help, visibility, and escalation — with one comment. 🚀