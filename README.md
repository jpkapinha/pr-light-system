# PR Traffic Light System

A plug-and-play GitHub Actions workflow that automates your Pull Request process
using a simple traffic light label system. Copy one file into your repo and you're done.

| Label          | What happens                                            |
|----------------|---------------------------------------------------------|
| `red-light`    | Review is automatically requested before merging        |
| `green-light`  | PR auto-merges once all CI checks pass                  |
| `yellow-light` | PR is flagged for manual attention — no automation      |
| *(no label)*   | PR is BLOCKED until a valid label is added              |

---

## Setup — 3 Steps

### Step 1 — Create the Labels

In your repository go to **Issues > Labels > New label** and create these three:

| Label name     | Suggested color | Purpose                      |
|----------------|-----------------|------------------------------|
| `red-light`    | `#EE0701`       | Needs review before merging  |
| `green-light`  | `#0E8A16`       | Safe to auto-merge           |
| `yellow-light` | `#E4E669`       | Caution — merge manually     |

> **Important:** Label names are case-sensitive. Use lowercase exactly as shown above.

---

### Step 2 — Enable Auto-Merge in Repo Settings

> Required for `green-light` to work. If you skip this, green-light PRs will fail
> with a clear error message pointing you back here.

1. Go to your repository **Settings > General**
2. Scroll to the **Pull Requests** section
3. Check **"Allow auto-merge"**

> You must be a repository **Admin** to change this setting.

---

### Step 3 — Copy the Workflow File

1. In your repository, create the path `.github/workflows/` if it does not exist
2. Create a new file at `.github/workflows/traffic-light.yml`
3. Copy the full contents of [`.github/workflows/traffic-light.yml`](./.github/workflows/traffic-light.yml) into it
4. Commit and push to your default branch (`main` or `master`)

> GitHub Actions activates automatically. No further configuration needed.

---

### Step 4 (Optional) — Enable AI Auto-Labeling

Want the system to automatically apply the correct label based on the code diff and your custom rules?
Just add your OpenRouter API key as a repository secret and provide a `.github/ai-rules.csv`.

> See [USAGE.md](./USAGE.md#ai-auto-labeling) for the complete 2-step setup guide.

---

## How It Works

### No Label → PR Blocked
When a PR is opened without a traffic light label, the GitHub Actions check **fails**
and blocks the merge. The developer must add a valid label to proceed.

### `red-light` → Path-Aware Review Routing
Reviews are automatically requested based on which files changed. The workflow matches
changed file paths against a configurable routing map:

- `.sol` files → Solidity / smart contract reviewers
- `.github/workflows/`, `infra/`, `docker`, `k8s/` → DevOps reviewers
- `src/api/`, `server/`, `backend/` → Backend reviewers
- `src/`, `components/`, `pages/` → Frontend reviewers

If no routing rules match, the review falls back to the **repository owner**.

**Customize the routing map** by editing the `ROUTING_RULES` array in the workflow file.
See [USAGE.md](./USAGE.md#change-who-receives-the-review-request-red-light) for details.

### `green-light` → Auto-Merge Enabled
GitHub's native auto-merge is activated. The PR merges automatically once
all required status checks pass.

### `yellow-light` → Manual Merge Required
The PR passes validation but no automation runs. A human must review and
merge it manually.

---

## AI Reasoning & Transparency

When AI auto-labeling is enabled, the system posts a comment on every classified PR explaining:
- **Which label** was assigned and **why**
- **Which files** drove the decision
- How to **override** the label if you disagree

This makes every classification decision visible, auditable, and educational for the team.

---

## Override Mechanism

If the AI (or a colleague) assigns a label you disagree with, any team member
other than the PR author can override it with a comment:

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

The system removes the old label, applies the new one, and re-runs the automation.
This requires a **different person** than the PR author to prevent self-approval.

---

## Classification Policy — `ai-rules.csv`

The file [`.github/ai-rules.csv`](./.github/ai-rules.csv) is the **source of truth** for how
PRs are classified in this repository. It defines risk levels by role, category, and task type
with explicit justifications and edge-case handling.

This file is a first-class policy artifact:
- **Changes to `ai-rules.csv` should always be `red-light` PRs** reviewed by senior developers or managers
- The same classification logic should be reflected in your team's [web3-skills](https://github.com/protofire/web3-skills) repository
- The AI reads this file on every PR to make its labeling decisions

See the file directly: [`.github/ai-rules.csv`](./.github/ai-rules.csv)

---

## Measuring Effectiveness

Track these metrics to evaluate whether the system is calibrated to your team:

| Metric                          | What it tells you                                        |
|---------------------------------|----------------------------------------------------------|
| Override rate                   | How often AI labels are changed via `/relabel`           |
| Override direction              | Is the AI too strict (red→yellow) or too permissive?     |
| Time-to-merge for green-light   | Are low-risk PRs actually merging faster?                |
| Red-light review turnaround     | Are the right reviewers responding quickly?              |

When the override rate is high for a specific category, update `ai-rules.csv` to improve accuracy.

---

## Repository Structure

```
.github/
  ai-rules.csv             ← Classification policy (first-class artifact)
  workflows/
    traffic-light.yml       ← The only file you need to copy into your repo
README.md                   ← This file
USAGE.md                    ← Detailed setup, customisation and troubleshooting
CONTRIBUTING.md             ← How to contribute improvements
SECURITY.md                 ← Data handling policy for AI features
LICENSE                     ← MIT License
```

---

## Security & Data Handling

When AI auto-labeling is enabled, the PR diff is sent to an external LLM provider.
See [SECURITY.md](./SECURITY.md) for full details on what data is transmitted,
risk considerations for Web3 projects, and how to opt out.

---

## Full Documentation

See [USAGE.md](./USAGE.md) for detailed setup instructions, customisation options,
and a full troubleshooting guide.

## License

[MIT](./LICENSE) — free to use in any repository, commercial or personal.
