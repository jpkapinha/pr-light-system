# Contributing to PR Traffic Light System

Thank you for your interest in improving this project! All contributions are welcome.

---

## Ways to Contribute

- Improve or clarify the documentation
- Fix bugs or edge cases in the workflow logic
- Add support for new automation patterns (e.g., auto-assign reviewers based on changed files)
- Suggest new label types with meaningful automation
- Propose updates to the classification policy (`ai-rules.csv`)
- Report issues or unexpected behaviour via GitHub Issues

---

## How to Submit a Change

1. **Fork** this repository
2. **Create a branch**: `git checkout -b feature/your-improvement`
3. **Make your changes** and test them
4. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/) format:
   - `feat:` for new features or automation
   - `fix:` for bug fixes
   - `docs:` for documentation-only changes
   - `policy:` for changes to `ai-rules.csv`
5. **Open a Pull Request** and add a traffic light label:

| Change type                                          | Label to use    |
|------------------------------------------------------|-----------------|
| Documentation, typos, formatting                     | `green-light`   |
| Logic or workflow changes needing review              | `red-light`     |
| Changes to `ai-rules.csv` (classification policy)    | `red-light`     |
| Changes to `SECURITY.md`                             | `red-light`     |
| Changes you want a second opinion on                 | `yellow-light`  |

---

## Updating the Classification Policy

The file `.github/ai-rules.csv` controls how PRs are automatically classified.
Changes to this file have security implications and **must always be submitted
as `red-light` PRs** reviewed by senior developers or managers.

When proposing a change to `ai-rules.csv`:
- Explain why the current classification is incorrect or insufficient
- Provide a real example PR that motivated the change
- Consider whether the change affects multiple roles or categories
- Check that new rules don't conflict with existing ones

---

## Override Mechanism

If you disagree with a label on a PR, you can override it by commenting:

```
/relabel green-light
/relabel yellow-light
/relabel red-light
```

This must come from a team member **other than the PR author**.

---

## Label Reference

| Label          | When to use                                              |
|----------------|----------------------------------------------------------|
| `red-light`    | Logic or workflow changes that need senior review         |
| `green-light`  | Safe, low-risk changes — docs, formatting, minor fixes   |
| `yellow-light` | Changes you are not fully confident about                |
