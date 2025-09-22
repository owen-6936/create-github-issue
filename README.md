# 🛠️ Create Issue Action

[![CI Matrix](https://img.shields.io/github/actions/workflow/status/owen-6936/create-github-issue/ci/test-action.yml?label=CI%20Matrix&logo=github)](https://github.com/owen-6936/create-github-issue/actions/workflows/ci/test-action.yml)
[![Version](https://img.shields.io/badge/version-v1.1.0-blue?logo=semver)](https://github.com/owen-6936/create-github-issue/releases)
[![License](https://img.shields.io/github/license/owen-6936/create-github-issue?color=brightgreen)](LICENSE)

Creates GitHub issues with custom title, body, labels, and assignees — with built-in duplicate detection, fallback handling, and CI-friendly cleanup.

---

## ✨ Features

- ✅ Prevents duplicate issues by checking existing open titles
- 🏷️ Auto-creates missing labels (including `created-by:ci`)
- 👥 Supports multiple assignees
- 📦 Fallback title/body if inputs are empty
- 🔗 Outputs issue URL and number for downstream use — even on duplicates
- 🧹 Compatible with cleanup workflows via label filtering
- 🧠 Modular output verification via `verify-issue` step

---

## 🚀 Usage

```yaml
- name: Create GitHub Issue
  uses: ./ # or path to your action
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    title: "Bug: Unexpected behavior in preview"
    body: |
      Steps to reproduce:
      1. Open preview
      2. Click "Generate"
      3. Observe crash

      Expected: graceful fallback
    labels: "bug,created-by:ci"
    assignees: "owen-6936"
    fallback_title: "Automated Issue"
    fallback_body: "No issue body was provided by the workflow."
```

---

## 🔐 Required Permissions

To run this action successfully, your workflow must include:

```yaml
permissions:
  issues: write
  contents: write
```

### Why

- `issues: write` — Required to create, update, and close issues.
- `contents: write` — Needed for label creation and repository metadata access via `gh`.

> These permissions must be set at the **job** or **workflow** level. Without them, the GitHub CLI will fail to authenticate or perform actions.

---

## 🧾 Inputs

| Name             | Required | Description                                      | Default                                  |
|------------------|----------|--------------------------------------------------|------------------------------------------|
| `github-token`   | ✅       | GitHub token for CLI authentication              | —                                        |
| `title`          | ❌       | Title of the issue                               | `""`                                     |
| `body`           | ❌       | Body content of the issue                        | `""`                                     |
| `labels`         | ❌       | Comma-separated list of labels                   | `""`                                     |
| `assignees`      | ❌       | Comma-separated list of usernames                | `""`                                     |
| `fallback_title` | ❌       | Used if `title` is empty                         | `"Automated Issue"`                      |
| `fallback_body`  | ❌       | Used if `body` is empty                          | `"No issue body was provided by the workflow."` |

---

## 📤 Outputs

| Name           | Description                          |
|----------------|--------------------------------------|
| `issue-url`    | URL of the created or existing issue |
| `issue-number` | Issue number                         |

> Outputs are guaranteed via the `verify-issue` step, even when a duplicate is detected.

---

## 🧪 Matrix Testing

Use this matrix-based test workflow to validate behavior:

```yaml
strategy:
  matrix:
    case:
      - name: "Fresh Issue"
        title: "Test: Fresh Issue"
        body: "This is a new issue for testing."
        labels: "automation,test"
        assignees: "${{ github.actor }}"

      - name: "Duplicate Detection"
        title: "Test: Fresh Issue"
        body: "Should trigger duplicate detection."
        labels: "duplicate-check"
        assignees: ""

      - name: "Fallback Handling"
        title: ""
        body: ""
        labels: ""
        assignees: ""

      - name: "Label Bootstrap"
        title: "Test: Label Creation"
        body: "This issue should trigger fresh label creation."
        labels: "bootstrap,ci-labels,new-start"
        assignees: ""

      - name: "Custom Labels & Assignees"
        title: "Test: Custom Assignment"
        body: "Verifying label creation and assignee propagation."
        labels: "needs-review,agentic-flow"
        assignees: "${{ github.actor }}"
```

Each case validates a unique behavior: creation, duplication, fallback, label bootstrapping, and metadata propagation.

---

## 🧹 Cleanup Workflow

Automatically close CI-created issues after test runs:

```yaml
- name: Close CI-created test issues
  shell: bash
  run: |
    echo "🧹 Cleaning up issues labeled 'created-by:ci'..."
    issues=$(gh issue list --repo "${GITHUB_REPOSITORY}" --label "created-by:ci" --state open --json number | jq -r '.[].number')
    for issue in $issues; do
      echo "🛑 Closing issue #$issue"
      gh issue close "$issue" --repo "${GITHUB_REPOSITORY}" --comment "Closed automatically after CI test run."
    done
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 🧠 Notes

- Requires `gh` CLI and `jq` installed (auto-installed in action)
- Labels are auto-created if missing
- Duplicate detection is based on exact title match
- Outputs are propagated via a dedicated `verify-issue` step
- CI matrix validates all core behaviors
- Action is modular, expressive, and reviewer-friendly

---

## 📦 License

MIT — © Owen 6936
