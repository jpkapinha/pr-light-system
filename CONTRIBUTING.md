# Contributing to PR Traffic Light System

Thank you for your interest in improving this project! All contributions are welcome.

---

## Ways to Contribute

- Improve or clarify the documentation
- Fix bugs or edge cases in the workflow logic
- Add support for new automation patterns (e.g., auto-assign reviewers based on changed files)
- Suggest new label types with meaningful automation
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
5. **Open a Pull Request** and add a traffic light label:

| Change type                                | Label to use    |
|--------------------------------------------|-----------------|
| Documentation, typos, formatting           | `green-light`   |
| Logic or workflow changes needing review   | `red-light`     |
| Changes you want a second opinion on       | `yellow-light`  |

---

## Label Reference

| Label          | When to use                                              |
|----------------|----------------------------------------------------------|
| `red-light`    | Logic or workflow changes that need senior review        |
| `green-light`  | Safe, low-risk changes - docs, formatting, minor fixes   |
| `yellow-light` | Changes you are not fully confident about                |
