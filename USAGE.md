# Usage Guide - PR Traffic Light System

This guide walks you through setting up, customising, and troubleshooting the
PR Traffic Light System in your own GitHub repository.

---

## Prerequisites

Before starting, confirm:
- You have **Admin access** to the target repository
- **GitHub Actions** is enabled (enabled by default on all GitHub repositories)

---

## Full Setup Guide

### Step 1 - Create the Labels

Labels are how developers communicate the risk level of a PR.
Without a label, the system will block the PR automatically.

1. Go to your GitHub repository
2. Click the **Issues** tab > click **Labels** > click **New label**
3. Create the following three labels:

| Label name     | Suggested hex color | Purpose                                   |
|----------------|---------------------|-------------------------------------------|
| `red-light`    | `#EE0701`           | Triggers automatic review request         |
| `green-light`  | `#0075CA`           | Triggers automatic merge when CI passes   |
| `yellow-light` | `#E4E669`           | Visual flag only - manual action required |

> **Warning:** Label names are case-sensitive. Use lowercase exactly as shown.

---

### Step 2 - Enable Auto-Merge

The `green-light` automation uses GitHub's native auto-merge feature.
It must be explicitly enabled per repository.

1. Go to **Settings > General**
2. Scroll down to the **Pull Requests** section
3. Check the box: **Allow auto-merge**

> If you skip this step, `green-light` PRs will not auto-merge and the action
> will fail with a message telling you exactly what to fix.

---

### Step 3 - Add the Workflow File

This is the **only file** you need to copy into your repository.

1. In your repository, create `.github/workflows/` if it does not exist yet
2. Create a new file at `.github/workflows/traffic-light.yml`
3. Copy the full contents of the workflow file into it:

   View the file here: [.github/workflows/traffic-light.yml](./.github/workflows/traffic-light.yml)

4. Commit the file directly to your default branch (`main` or `master`)

> The system is now live. It triggers automatically on every new or updated Pull Request.

---

## Customisation

### Change who receives the review request (red-light)

By default, the review is requested from the repository owner.
To change this, open `.github/workflows/traffic-light.yml` and find the step named
`"Request Review (red-light)"`, then update the `reviewers` line:

```yaml
# Request from one or more specific GitHub users:
reviewers: ['alice', 'bob']

# Request from a GitHub team (use the team slug, not the display name):
team_reviewers: ['backend-team']

# Both users and a team:
reviewers: ['alice']
team_reviewers: ['security-team']
```

> For team reviewers, the team must have at least Read access to the repository.

---

### Change the merge method (green-light)

By default the system uses a standard merge commit.
To change it, find the `mergeMethod` line in the workflow and replace the value:

| Value    | Result                                          |
|----------|-------------------------------------------------|
| `MERGE`  | Standard merge commit (default)                 |
| `SQUASH` | Squash all commits into one before merging      |
| `REBASE` | Rebase commits onto the base branch             |

---

## Label Behaviour Reference

| Label          | Automated action                                                                        |
|----------------|-----------------------------------------------------------------------------------------|
| `red-light`    | Review requested from owner (or custom list). Merge is blocked until approved.          |
| `green-light`  | Auto-merge enabled. PR merges once all required CI checks pass.                         |
| `yellow-light` | No automation. Validation passes, but a human must review and merge manually.           |
| *(no label)*   | GitHub Actions check **FAILS**. PR is fully blocked until a valid label is added.       |

---

## Troubleshooting

**Green-light auto-merge is failing with a settings error**
Go to Settings > General > Pull Requests and enable "Allow auto-merge".

**Red-light review request fails**
Check that the reviewer username is correct and that they have at least
Read access to the repository.

**The workflow is not running at all**
Confirm the file is at exactly `.github/workflows/traffic-light.yml`.
The folder must start with a dot (`.github`), not just `github`.

**The label check keeps failing even after adding a label**
Label names are case-sensitive. Confirm you are using exactly:
`red-light`, `green-light`, or `yellow-light` (all lowercase, hyphenated).

**GitHub Actions is not enabled on the repository**
Go to Settings > Actions > General and set it to
"Allow all actions and reusable workflows".

---

## Why Use This System?

| Without Traffic Lights                             | With Traffic Lights                                     |
|----------------------------------------------------|---------------------------------------------------------|
| Developers forget to request reviews               | Reviews are requested automatically on every risky PR   |
| Routine PRs wait days for a manual merge           | Low-risk PRs merge themselves once CI is green          |
| No standard for PR risk classification             | Every PR is explicitly categorised before it can merge  |
| Risky changes can slip through unreviewed          | High-risk PRs are always flagged - no exceptions        |
