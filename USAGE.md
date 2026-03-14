# Usage Guide — PR Traffic Light System

Complete reference for setup, customisation, and troubleshooting.

---

## Prerequisites

- **Admin access** to the target repository
- **GitHub Actions** enabled (on by default for all GitHub repos)

---

## Full Setup Guide

### Step 1 — Create the Labels

Without labels the system will block every PR. Create these three labels first.

1. Go to **Issues → Labels → New label**
2. Create each one:

| Label name     | Hex color   | Purpose                                        |
|----------------|-------------|------------------------------------------------|
| `red-light`    | `#EE0701`   | Triggers automatic review request              |
| `green-light`  | `#0E8A16`   | Triggers automatic merge when CI passes        |
| `yellow-light` | `#E4E669`   | Visual flag only — no automation, manual merge |

> **Label names are case-sensitive.** Use lowercase with hyphens exactly as shown.

---

### Step 2 — Enable Auto-Merge

The `green-light` path uses GitHub's native auto-merge. It must be enabled per repo.

1. **Settings → General → Pull Requests**
2. Check **"Allow auto-merge"**

> If you skip this, `green-light` PRs will fail with a clear message telling you exactly what to fix.

---

### Step 3 — Enable Branch Protection on `main`

> **Required for auto-merge.** GitHub only allows auto-merge on branches that have protection rules. This is a GitHub platform constraint.

1. **Settings → Branches → Add branch protection rule**
2. Branch name pattern: `main` (or your default branch)
3. ✅ **Require status checks to pass before merging**
4. Search for and add `Check Traffic Light Label` as a required check
5. ✅ **Require branches to be up to date before merging**
6. Save changes

---

### Step 4 — Add the Workflow File

This is the **only file** you need to copy into your repository.

1. Create `.github/workflows/` if it doesn't exist
2. Create `.github/workflows/traffic-light.yml`
3. Copy the full contents from [`.github/workflows/traffic-light.yml`](./.github/workflows/traffic-light.yml)
4. Commit directly to your default branch

The system is now live and triggers on every new or updated PR.

---

### Step 5 (Optional) — Enable AI Auto-Labeling

The AI reads the PR diff and automatically applies the correct label. It also posts a comment explaining its reasoning and which files drove the decision.

**Step 5a — Add your API key**

1. Get an API key at [openrouter.ai](https://openrouter.ai/)
2. **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `OPENROUTER_API_KEY`
   - Value: your key

**Step 5b — (Optional) Add custom classification rules**

By default the AI uses sensible defaults. To teach it your team's specific rules, create `.github/ai-rules.csv`. This repo includes a full example covering Frontend, Backend, Solidity, DevOps, and QA roles.

See [`.github/ai-rules.csv`](./.github/ai-rules.csv) for the reference policy.

> **Bypassing AI per-PR:** If a developer manually adds a label before the workflow runs, the AI step skips itself and respects that choice.

> **Turning AI off repo-wide:** Don't add `OPENROUTER_API_KEY`. Everything works — labels are just applied manually.

---

## Customisation

### Change who gets the review request

When a `red-light` PR is opened, the workflow checks which files changed and routes the review to the right team using the `ROUTING_RULES` array in the workflow file.

Open `.github/workflows/traffic-light.yml` and find the `ROUTING_RULES` block:

```javascript
const ROUTING_RULES = [
  {
    pattern: /\.(sol)$/i,
    reviewers: ['alice', 'bob'],           // GitHub usernames
    team_reviewers: ['solidity-security'], // GitHub team slug
    description: 'Solidity / Smart Contracts'
  },
  {
    pattern: /^(\.github\/workflows\/|infra\/|docker|k8s\/|\.env)/i,
    reviewers: ['charlie'],
    team_reviewers: ['devops-team'],
    description: 'Infrastructure / DevOps'
  },
  {
    pattern: /^(src\/api\/|server\/|backend\/)/i,
    reviewers: ['dave'],
    team_reviewers: ['backend-team'],
    description: 'Backend / API'
  },
  {
    pattern: /^(src\/|components\/|pages\/|app\/)/i,
    reviewers: ['eve'],
    team_reviewers: ['frontend-team'],
    description: 'Frontend'
  },
];
```

**How it works:**
- Each rule has a `pattern` (regex matched against changed file paths), `reviewers` (GitHub usernames), `team_reviewers` (GitHub team slugs), and `description` (for logs)
- A PR can match **multiple rules** — reviewers from all matches are combined
- If no rules match, or all reviewer lists are empty, it falls back to the **repository owner**
- The PR author is automatically filtered out (GitHub rejects self-review requests)
- Team reviewers require the team to have at least **Read** access to the repository

---

### Change the merge method

By default, `green-light` uses a standard merge commit. To change it, find `mergeMethod` in the workflow and replace the value:

| Value    | Result                                     |
|----------|--------------------------------------------|
| `MERGE`  | Standard merge commit (default)            |
| `SQUASH` | Squash all commits into one before merging |
| `REBASE` | Rebase commits onto the base branch        |

---

### Update the classification policy (`ai-rules.csv`)

The file `.github/ai-rules.csv` controls AI labeling decisions. Each row maps a role + task type to a risk level.

| Column              | Purpose                                                   |
|---------------------|-----------------------------------------------------------|
| Role                | Team role (Frontend Engineers, Solidity Developers, etc.) |
| Category            | Type of work (UI Components, Core Business Logic, etc.)   |
| Task Name           | Specific task description                                 |
| Traffic Light       | Risk level: GREEN, YELLOW, or RED                         |
| Justification       | Why this risk level was chosen                            |
| Critical Analysis   | Known edge cases or challenges                            |
| Nuance & Conditions | When to upgrade or downgrade the classification           |

> **Changes to `ai-rules.csv` must always be submitted as `red-light` PRs** and reviewed by senior developers. This file is a security policy artifact.

When updating a rule:
- Explain why the current classification is wrong
- Include a real PR example that motivated the change
- Check for conflicts with existing rules
- Verify the nuance conditions are specific enough for the AI to interpret

---

## Override Mechanism

Any team member **other than the PR author** can override any label by posting a PR comment:

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

**What happens when you run this:**
1. The system validates the command format
2. It verifies you are not the PR author (self-overrides are blocked)
3. It removes the existing traffic light label
4. It applies the new label
5. It posts a confirmation comment
6. The automation re-triggers with the new label

**Why the author cannot self-override:** This enforces the four-eyes principle. No one can upgrade their own PR's risk classification.

**Example:**

> Developer A opens a PR. AI labels it `yellow-light`.
> Developer A thinks it should be `green-light` but cannot override their own PR.
> Developer B reviews the diff, agrees, and posts `/relabel green-light`.
> The system relabels the PR and auto-merge is activated.

---

## Label Behaviour Reference

| Label          | What happens                                                                            |
|----------------|-----------------------------------------------------------------------------------------|
| `red-light`    | Review requested from matched team (or repo owner). PR cannot merge until approved.     |
| `green-light`  | Auto-merge enabled. PR merges once all required CI checks pass.                         |
| `yellow-light` | Validation passes. No automation. A human must review and merge manually.               |
| *(no label)*   | `Check Traffic Light Label` check **fails**. PR is fully blocked.                       |

---

## Troubleshooting

**The workflow is not running at all**
Confirm the file is at exactly `.github/workflows/traffic-light.yml`.
The folder must start with a dot (`.github`), not just `github`.
Go to **Settings → Actions → General** and confirm actions are set to "Allow all actions and reusable workflows".

**The label check fails even after adding a label**
Label names are case-sensitive. Must be exactly `red-light`, `green-light`, or `yellow-light` — all lowercase, hyphenated.

**`green-light` auto-merge fails with a settings error**
Go to **Settings → General → Pull Requests** and enable "Allow auto-merge".

**`green-light` auto-merge fails even with "Allow auto-merge" enabled**
Branch protection is missing on `main`. Follow Step 3 above.
Auto-merge requires at least one active branch protection rule — this is enforced by GitHub.

**`red-light` review request fails**
- Confirm the reviewer's GitHub username is correct
- Confirm they have at least Read access to the repository
- GitHub rejects self-review requests — if the PR author is the only reviewer in the list, the fallback warning comment will appear instead

**The `/relabel` command is not working**
- Confirm your workflow file includes `issue_comment: types: [created]` in the `on:` block
- The commenter must not be the PR author — self-overrides are blocked by design
- The command must be on its own line with no extra text: `/relabel green-light`

**AI is not labeling PRs**
- Check that `OPENROUTER_API_KEY` is set under **Settings → Secrets and variables → Actions**
- Check the Actions log for the `ai-label-assignment` job for API error messages
- If a label was already applied before the workflow ran, the AI skips intentionally

**AI comment is missing but label was applied**
The AI returned a response the parser couldn't fully read. The label was applied using the fallback text-only parser. The classification is still valid. Check the Actions log for details.

**AI labels are consistently wrong for a specific task type**
Update `.github/ai-rules.csv` with a clearer rule for that category and submit it as a `red-light` PR.

---

## Security & Data Handling

See [SECURITY.md](./SECURITY.md) for full details on what data is transmitted when AI auto-labeling is enabled, risk considerations for proprietary or NDA-covered code, and how to opt out.
