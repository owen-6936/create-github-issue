# 🛠️ Create Issue Action

Creates GitHub issues with custom title, body, labels, and assignees — with built-in duplicate detection, fallback handling, and CI-friendly cleanup.

## ✨ Features

- ✅ Prevents duplicate issues by checking existing open titles
- 🏷️ Auto-creates missing labels (including `created-by:ci`)
- 👥 Supports multiple assignees
- 📦 Fallback title/body if inputs are empty
- 🔗 Outputs issue URL and number for downstream use
- 🧹 Compatible with cleanup workflows via label filtering

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

---

## 🧪 Testing

Use this matrix-based test workflow to validate behavior:

```yaml
name: Test Create Issue Action

on:
  push:
    branches:
      - ci/test-issue-action

jobs:
  test:
    runs-on: ubuntu-latest
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

    name: ${{ matrix.case.name }}
    steps:
      - uses: actions/checkout@v4

      - name: Run Create Issue Action
        uses: ./
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          title: ${{ matrix.case.title }}
          body: ${{ matrix.case.body }}
          labels: "created-by:ci,${{ matrix.case.labels }}"
          assignees: ${{ matrix.case.assignees }}
          fallback_title: "Fallback Title Used"
          fallback_body: "Fallback body used for empty input."

      - name: Log outputs
        run: |
          echo "🔗 Issue URL: ${{ steps.create.outputs.issue-url }}"
          echo "🔢 Issue Number: ${{ steps.create.outputs.issue-number }}"
```

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
- Outputs are safely escaped using multiline syntax

---

## 📦 License

MIT — © Owen 6936
