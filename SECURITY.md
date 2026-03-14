# Security Policy

## AI-Assisted PR Classification — Data Handling

When the optional AI auto-labeling feature is enabled (by providing an `OPENROUTER_API_KEY` secret),
the workflow sends the following data to the configured LLM provider (OpenRouter) on every PR
that does not already have a traffic light label:

- **PR title** and **description** (the body text of the pull request)
- **Full code diff** of the PR (truncated to 100,000 characters if larger)
- **Contents of `.github/ai-rules.csv`** if the file exists in the repository

### What is NOT sent

- Repository secrets, environment variables, or API keys
- Files outside the PR diff (source tree, dependencies, build artifacts)
- GitHub tokens or authentication credentials

### Risk considerations for Web3 projects

If your repository contains **proprietary smart contract logic**, **audit-sensitive code**, or
**code under NDA with a client**, be aware that the PR diff will be transmitted to a third-party
API. Consider the following mitigations:

1. **Opt out per-PR:** Manually apply a traffic light label before the workflow runs.
   The AI step is skipped entirely when a label is already present.
2. **Opt out per-repo:** Simply do not add the `OPENROUTER_API_KEY` secret.
   The workflow functions fully without AI — labels are applied manually.
3. **Review your provider's data policy:** Check [OpenRouter's privacy policy](https://openrouter.ai/privacy)
   and the downstream model provider's terms to understand data retention and training usage.
4. **Self-hosted inference:** For maximum confidentiality, consider replacing the OpenRouter
   call with a self-hosted model endpoint. The workflow prompt is fully customizable.

### Reporting a vulnerability

If you discover a security issue in this workflow (e.g., a way to exfiltrate secrets
or bypass label enforcement), please report it privately:

- **Email:** security@protofire.io
- **GitHub:** Open a [private security advisory](https://docs.github.com/en/code-security/security-advisories)

Do not open a public issue for security vulnerabilities.
