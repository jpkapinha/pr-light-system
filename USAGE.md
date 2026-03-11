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

### Step 3 - Enable Branch Protection on `main` ⚠️

> **This step is required for auto-merge to work.** GitHub only allows
> auto-merge on PRs that target a branch with protection rules enabled.
> Without this, the auto-merge API call will be rejected regardless of
> permissions or workflow configuration.

1. Go to **Settings > Branches**
2. Click **Add branch protection rule**
3. Set **Branch name pattern** to `main` (or your default branch name)
4. ✅ Check **Require status checks to pass before merging**
5. In the search box, find and add `Check Traffic Light Label` as a required status check
6. ✅ Check **Require branches to be up to date before merging**
7. Click **Save changes**

> **Why is this required?** GitHub's auto-merge feature is intentionally
> gated behind branch protection. It is designed to prevent accidental merges
> and only activates when there are enforced checks in place. This is a GitHub
> platform constraint, not a limitation of this workflow.

---

### Step 4 - Add the Workflow File

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

> **Note:** GitHub does not allow requesting a review from the PR author themselves.
> If the repo owner opens a PR, the red-light review request will fail. Add at least
> one collaborator as a reviewer to avoid this.

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

### AI Auto-Labeling

You can configure the workflow to use OpenRouter to analyze the PR diff and automatically apply the correct label on your behalf. If the AI applies a label, the rest of the automation (`green-light` auto-merge, `red-light` review request) continues exactly as if a human had applied it.

**Step 1. Get and Add your API Key**
1. Go to [OpenRouter](https://openrouter.ai/) and create an account.
2. Navigate to your **Keys** settings and generate a new API Key.
3. In your GitHub repository, go to **Settings > Secrets and variables > Actions**
4. Click **New repository secret**
5. Name: `OPENROUTER_API_KEY`
6. Secret: Paste your valid OpenRouter API Key.

**Step 2. (Optional) Provide Custom Rules/Examples**
By default, the AI is prompted with sensible defaults (e.g. docs = green, core logic = red). To teach the AI your specific repository rules, create a file at `.github/ai-rules.csv` containing historical examples:

```csv
PR Title; Files Changed; Correct Label; Reason
Update README; *.md; green-light; Documentation only
Add user table; db/schema.sql; red-light; Database migrations require review
Fix button color; src/Button.css; green-light; Safe UI styling change
```
*Note: The AI reads this file plaintext, so any delimiter (commas, semicolons) is fine.*

> **Bypass the AI:** If a human manually applies a traffic light label *before* the action runs (or immediately upon PR creation), the AI will gracefully skip itself and respect the human's choice.

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
Go to **Settings > General > Pull Requests** and enable "Allow auto-merge".

**Green-light auto-merge fails even with "Allow auto-merge" enabled**
Branch protection rules are missing on `main`. Follow Step 3 above.
Auto-merge requires at least one branch protection rule to be active —
this is enforced by GitHub at the platform level.

**Red-light review request fails**
Check that the reviewer username is correct and that they have at least
Read access to the repository. Also note that GitHub does not allow the
PR author to be their own reviewer — add a collaborator to the `reviewers` list.

**The workflow is not running at all**
Confirm the file is at exactly `.github/workflows/traffic-light.yml`.
The folder must start with a dot (`.github`), not just `github`.

**The label check keeps failing even after adding a label**
Label names are case-sensitive. Confirm you are using exactly:
`red-light`, `green-light`, or `yellow-light` (all lowercase, hyphenated).

**GitHub Actions is not enabled on the repository**
Go to **Settings > Actions > General** and set it to
"Allow all actions and reusable workflows".

---

## Why Use This System?

| Without Traffic Lights                             | With Traffic Lights                                     |
|----------------------------------------------------|---------------------------------------------------------|
| Developers forget to request reviews               | Reviews are requested automatically on every risky PR   |
| Routine PRs wait days for a manual merge           | Low-risk PRs merge themselves once CI is green          |
| No standard for PR risk classification             | Every PR is explicitly categorised before it can merge  |
| Risky changes can slip through unreviewed          | High-risk PRs are always flagged - no exceptions        |
