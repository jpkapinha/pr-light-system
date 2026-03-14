# Usage Guide — PR Traffic Light System

This guide walks you through setting up, customising, and troubleshooting the
PR Traffic Light System in your own GitHub repository.

---

## Prerequisites

Before starting, confirm:
- You have **Admin access** to the target repository
- **GitHub Actions** is enabled (enabled by default on all GitHub repositories)

---

## Full Setup Guide

### Step 1 — Create the Labels

Labels are how developers communicate the risk level of a PR.
Without a label, the system will block the PR automatically.

1. Go to your GitHub repository
2. Click the **Issues** tab > click **Labels** > click **New label**
3. Create the following three labels:

| Label name     | Suggested hex color | Purpose                                   |
|----------------|---------------------|-------------------------------------------|
| `red-light`    | `#EE0701`           | Triggers automatic review request         |
| `green-light`  | `#0E8A16`           | Triggers automatic merge when CI passes   |
| `yellow-light` | `#E4E669`           | Visual flag only — manual action required |

> **Warning:** Label names are case-sensitive. Use lowercase exactly as shown.

---

### Step 2 — Enable Auto-Merge

The `green-light` automation uses GitHub's native auto-merge feature.
It must be explicitly enabled per repository.

1. Go to **Settings > General**
2. Scroll down to the **Pull Requests** section
3. Check the box: **Allow auto-merge**

> If you skip this step, `green-light` PRs will not auto-merge and the action
> will fail with a message telling you exactly what to fix.

---

### Step 3 — Enable Branch Protection on `main` ⚠️

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

### Step 4 — Add the Workflow File

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

The system uses **path-aware reviewer routing**. When a `red-light` PR is detected,
the workflow checks which files were changed and routes the review request to the
appropriate team.

To customise, open `.github/workflows/traffic-light.yml` and find the `ROUTING_RULES`
array in the "Request Review (red-light)" step:

```javascript
const ROUTING_RULES = [
  {
    pattern: /\.(sol)$/i,
    reviewers: ['alice', 'bob'],           // Solidity experts
    team_reviewers: ['solidity-security'], // or a GitHub team
    description: 'Solidity / Smart Contracts'
  },
  {
    pattern: /^(\.github\/workflows\/|infra\/|docker|k8s\/|\.env)/i,
    reviewers: ['charlie'],
    team_reviewers: ['devops-team'],
    description: 'Infrastructure / DevOps'
  },
  // ... add more rules as needed
];
```

Each rule has:
- `pattern` — a regex matched against changed file paths
- `reviewers` — GitHub usernames to request reviews from
- `team_reviewers` — GitHub team slugs (team must have at least Read access)
- `description` — a human-readable label for logging

If a PR changes files matching multiple rules, reviewers from **all** matching rules
are requested. If no rules match, the review falls back to the **repository owner**.

> **Note:** GitHub does not allow requesting a review from the PR author themselves.
> The workflow automatically filters out the PR author from the reviewer list.

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

You can configure the workflow to use OpenRouter to analyze the PR diff and automatically
apply the correct label on your behalf. When AI labels a PR, it also posts a **reasoning
comment** explaining why that label was chosen, which files drove the decision, and how
to override it.

**Step 1. Get and Add your API Key**
1. Go to [OpenRouter](https://openrouter.ai/) and create an account.
2. Navigate to your **Keys** settings and generate a new API Key.
3. In your GitHub repository, go to **Settings > Secrets and variables > Actions**
4. Click **New repository secret**
5. Name: `OPENROUTER_API_KEY`
6. Secret: Paste your valid OpenRouter API Key.

**Step 2. (Optional) Provide Custom Rules/Examples**
By default, the AI is prompted with sensible defaults (e.g. docs = green, core logic = red).
To teach the AI your specific repository rules, create a file at `.github/ai-rules.csv`.

See [`.github/ai-rules.csv`](./.github/ai-rules.csv) for the full classification policy
used by the Protofire team. This file covers frontend, backend, Solidity, DevOps, and QA
roles with Web3-specific risk assessments.

> **Important:** Changes to `ai-rules.csv` should always be submitted as `red-light` PRs
> and reviewed by senior developers or managers, since this file controls the security
> classification policy for all future PRs.

> **Bypass the AI:** If a human manually applies a traffic light label *before* the action
> runs (or immediately upon PR creation), the AI will gracefully skip itself and respect
> the human's choice.

---

### Override Mechanism

If you disagree with an AI-assigned (or human-assigned) label, any team member
**other than the PR author** can override it by posting a comment:

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

The system will:
1. Validate the command syntax
2. Verify the commenter is not the PR author (to prevent self-approval)
3. Remove the existing traffic light label
4. Apply the new label
5. Post a confirmation comment
6. The automation re-triggers with the updated label

This ensures every classification can be contested while maintaining the
four-eyes principle — no one can upgrade their own PR's risk level.

---

## Classification Policy — `ai-rules.csv`

The [`.github/ai-rules.csv`](./.github/ai-rules.csv) file is the source of truth for PR
classification. It is structured with the following columns:

| Column              | Purpose                                                    |
|---------------------|------------------------------------------------------------|
| Role                | Which team role this rule applies to                       |
| Category            | The type of work (UI, APIs, Smart Contracts, etc.)         |
| Task Name           | Specific task description                                  |
| Traffic Light       | The assigned risk level (GREEN, YELLOW, RED)               |
| Justification       | Why this risk level was chosen                             |
| Critical Analysis   | Edge cases or challenges with this classification          |
| Nuance & Conditions | When the classification should be upgraded or downgraded   |

When updating this file, consider:
- Does the new rule conflict with existing rules?
- Are the nuance conditions clear enough for the AI to interpret?
- Has this change been reviewed by someone who understands the security implications?

---

## Measuring Effectiveness

To evaluate whether the traffic light system is well-calibrated, track:

| Metric                          | How to measure                                           | What it tells you                         |
|---------------------------------|----------------------------------------------------------|-------------------------------------------|
| Override rate                   | Count `/relabel` comments vs. total AI-labeled PRs       | How often the AI gets it wrong            |
| Override direction              | Track old-label → new-label pairs                        | Is the AI too strict or too permissive?   |
| Time-to-merge (green-light)     | Compare merge times before/after adoption                | Is automation actually saving time?       |
| Review turnaround (red-light)   | Time from review request to approval                     | Are the right reviewers responding?       |
| Label distribution              | % of PRs in each category over time                      | Is the team writing more low-risk PRs?    |

Use high override rates for specific categories as a signal to update `ai-rules.csv`.

---

## Label Behaviour Reference

| Label          | Automated action                                                                        |
|----------------|-----------------------------------------------------------------------------------------|
| `red-light`    | Review requested from matched reviewers (or owner). Merge is blocked until approved.    |
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

**The `/relabel` command is not working**
The override mechanism requires the `issue_comment` event trigger. Make sure
your workflow file includes `issue_comment: types: [created]` in the `on:` block.
Also verify the commenter is not the PR author — self-overrides are blocked.

**AI reasoning comment is not appearing**
Check the Actions log for the `ai-label-assignment` job. If the LLM returned
an unparseable response, the label may still be applied but the reasoning
comment may be missing. The workflow handles this gracefully with a fallback.

---

## Security & Data Handling

See [SECURITY.md](./SECURITY.md) for full details on what data is sent to external
APIs when AI auto-labeling is enabled, risk considerations for Web3 projects,
and how to opt out per-PR or per-repo.

---

## Why Use This System?

| Without Traffic Lights                             | With Traffic Lights                                     |
|----------------------------------------------------|---------------------------------------------------------|
| Developers forget to request reviews               | Reviews are requested automatically on every risky PR   |
| Routine PRs wait days for a manual merge           | Low-risk PRs merge themselves once CI is green          |
| No standard for PR risk classification             | Every PR is explicitly categorised before it can merge  |
| Risky changes can slip through unreviewed          | High-risk PRs are always flagged — no exceptions        |
| AI classifications are opaque                      | Every AI decision is explained and contestable          |
| Reviews go to the wrong person                     | Reviews are routed by file type to domain experts       |
