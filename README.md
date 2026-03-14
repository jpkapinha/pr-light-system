# PR Traffic Light System

A plug-and-play GitHub Actions workflow that automates your Pull Request process using a simple traffic light label system. Drop one file into any repo and it works immediately.

| Label          | What happens                                                   |
|----------------|----------------------------------------------------------------|
| `red-light`    | Review is automatically requested from the right team          |
| `green-light`  | PR auto-merges once all CI checks pass                         |
| `yellow-light` | PR is flagged for manual attention — no automation             |
| *(no label)*   | PR is **blocked** — merge is impossible until a label is added |

---

## Quick Start — 4 Steps

### Step 1 — Create the Labels

In your repository: **Issues → Labels → New label**. Create these three:

| Label name     | Color     | Hex       |
|----------------|-----------|-----------|
| `red-light`    | Red       | `#EE0701` |
| `green-light`  | Green     | `#0E8A16` |
| `yellow-light` | Yellow    | `#E4E669` |

> Label names are **case-sensitive**. Use lowercase exactly as shown.

---

### Step 2 — Enable Auto-Merge

Required for `green-light` to work.

1. **Settings → General → Pull Requests**
2. Check **"Allow auto-merge"**

---

### Step 3 — Enable Branch Protection

Required for auto-merge to activate (GitHub enforces this at platform level).

1. **Settings → Branches → Add branch protection rule**
2. Branch name pattern: `main`
3. ✅ Check **"Require status checks to pass before merging"**
4. Add `Check Traffic Light Label` as a required status check
5. ✅ Check **"Require branches to be up to date before merging"**
6. Save

---

### Step 4 — Copy the Workflow File

1. Create `.github/workflows/` in your repo if it doesn't exist
2. Copy [`.github/workflows/traffic-light.yml`](./.github/workflows/traffic-light.yml) into it
3. Commit to `main`

**Done.** The system is live. Every new PR is now automatically processed.

---

### Step 5 (Optional) — Enable AI Auto-Labeling

Let the AI read the diff and apply the correct label automatically — and explain its reasoning in a comment.

1. Get an API key from [OpenRouter](https://openrouter.ai/)
2. **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `OPENROUTER_API_KEY`
   - Value: your API key
3. (Optional) Add `.github/ai-rules.csv` to teach the AI your team's specific rules

See [USAGE.md](./USAGE.md#ai-auto-labeling) for full details.

---

## How It Works

### No label → PR blocked
The `Check Traffic Light Label` status check fails. The PR cannot be merged until a valid label is added.

### `red-light` → Path-aware review routing
The workflow checks which files changed and routes the review request to the right team:

| Files changed                                    | Routed to                  |
|--------------------------------------------------|----------------------------|
| `*.sol`                                          | Solidity reviewers         |
| `.github/workflows/`, `infra/`, `docker`, `k8s/`| DevOps reviewers           |
| `src/api/`, `server/`, `backend/`                | Backend reviewers          |
| `src/`, `components/`, `pages/`, `app/`          | Frontend reviewers         |
| *(no match)*                                     | Repository owner (fallback)|

Customize the routing map by editing `ROUTING_RULES` in the workflow file.
See [USAGE.md](./USAGE.md#change-who-gets-the-review-request) for instructions.

### `green-light` → Auto-merge
GitHub's native auto-merge is enabled on the PR. It merges automatically once all required status checks pass.

### `yellow-light` → Manual merge required
Validation passes, no automation runs. A human must review and merge it manually.

---

## AI Reasoning & Transparency

When AI auto-labeling is enabled, the AI posts a comment on every PR it classifies:

```
🟡 AI Classification: `yellow-light`

Reasoning: This PR modifies a wagmi connector configuration which directly
affects RPC endpoints. Wrong configuration can expose users to MITM attacks.

Files of concern:
- src/connectors/walletConnect.ts

ℹ️ About this classification
This label was automatically assigned by AI based on the PR diff and
the repository .github/ai-rules.csv policy.
- Agree? No action needed.
- Disagree? A team member can override with: /relabel green-light
```

Every decision is visible, auditable, and overridable.

---

## Override Mechanism

If any label (AI-assigned or human-assigned) is wrong, any team member **other than the PR author** can override it by posting a comment:

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

The system removes the old label, applies the new one, and re-runs automation. The PR author cannot override their own label — a second person is always required.

---

## Classification Policy — `ai-rules.csv`

[`.github/ai-rules.csv`](./.github/ai-rules.csv) is the source of truth for how PRs are classified. It covers every role (Frontend, Backend, Solidity, DevOps, QA) with risk levels, justifications, and edge-case conditions.

- The AI reads this file on every PR to make labeling decisions
- **Changes to this file must always be `red-light` PRs** reviewed by senior developers

---

## Measuring Effectiveness

| Metric                        | What it tells you                              |
|-------------------------------|------------------------------------------------|
| Override rate                 | How often AI labels are wrong                  |
| Override direction            | Is the AI too strict or too permissive?        |
| Time-to-merge (green-light)   | Are low-risk PRs actually merging faster?      |
| Review turnaround (red-light) | Are the right reviewers responding quickly?    |

High override rate on a specific category = update `ai-rules.csv`.

---

## Repository Structure

```
.github/
  ai-rules.csv                ← Classification policy (edit to teach the AI your rules)
  workflows/
    traffic-light.yml         ← The only file you need to copy into your repo
README.md                     ← This file
USAGE.md                      ← Full setup, customisation, and troubleshooting guide
CONTRIBUTING.md               ← How to contribute improvements to this project
SECURITY.md                   ← Data handling policy for the AI feature
LICENSE                       ← MIT License
```

---

## Security

When AI auto-labeling is enabled, the PR diff is sent to an external LLM provider. See [SECURITY.md](./SECURITY.md) for what is and isn't transmitted, and how to opt out per-PR or per-repo.

---

## License

[MIT](./LICENSE) — free to use in any repository, commercial or personal.
