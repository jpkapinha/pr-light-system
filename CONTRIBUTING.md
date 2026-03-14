# Contributing to PR Traffic Light System

Thank you for your interest in improving this project. All contributions are welcome.

---

## Ways to Contribute

- Fix bugs or edge cases in the workflow logic
- Improve or clarify documentation
- Add new automation patterns (e.g. auto-assign reviewers based on changed files)
- Propose updates to the classification policy (`ai-rules.csv`)
- Report issues or unexpected behaviour via GitHub Issues

---

## How to Submit a Change

1. **Fork** this repository
2. **Create a branch**: `git checkout -b feature/your-improvement`
3. **Make your changes** and test them
4. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/) format:

| Prefix     | Use for                                              |
|------------|------------------------------------------------------|
| `feat:`    | New features or automation                           |
| `fix:`     | Bug fixes                                            |
| `docs:`    | Documentation-only changes                           |
| `policy:`  | Changes to `ai-rules.csv`                           |

5. **Open a Pull Request** and apply the correct label:

| Change type                                       | Label            |
|---------------------------------------------------|------------------|
| Documentation, typos, formatting                  | `green-light`    |
| Logic or workflow changes                         | `red-light`      |
| Changes to `ai-rules.csv`                        | `red-light`      |
| Changes to `SECURITY.md`                         | `red-light`      |
| Changes you want a second opinion on              | `yellow-light`   |

---

## Updating the Classification Policy (`ai-rules.csv`)

The file `.github/ai-rules.csv` controls how PRs are automatically classified by the AI. Changes here affect the security posture of every team that uses this system.

**Rules for policy changes:**
- Always submitted as a `red-light` PR
- Must be reviewed by a senior developer or manager before merging
- Must include a justification explaining why the current classification is wrong
- Should reference a real PR example that motivated the change
- Must not conflict with existing rules

When adding a new row, fill in all columns:
- **Role** — which team role this applies to
- **Category** — type of work
- **Task Name** — specific task description (be precise)
- **Traffic Light** — GREEN, YELLOW, or RED
- **Justification** — why this risk level is correct
- **Critical Analysis** — known edge cases or failure modes
- **Nuance & Conditions** — when to upgrade or downgrade the level

---

## Override Mechanism

If you disagree with a label on any PR, post a comment as a team member (not the PR author):

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

The system validates the command, checks you are not the PR author, removes the existing label, applies the new one, and re-runs automation. Self-overrides are always blocked.

---

## Label Reference

| Label          | When to use                                              |
|----------------|----------------------------------------------------------|
| `red-light`    | Logic or workflow changes that need senior review        |
| `green-light`  | Safe, low-risk changes — docs, formatting, minor fixes   |
| `yellow-light` | Changes you are not fully confident about                |
