# PR Traffic Light System

A plug-and-play GitHub Actions workflow that automates your Pull Request process
using a simple traffic light label system. Copy one file into your repo and you're done.

| Label          | What happens                                            |
|----------------|---------------------------------------------------------|
| `red-light`    | Review is automatically requested before merging        |
| `green-light`  | PR auto-merges once all CI checks pass                  |
| `yellow-light` | PR is flagged for manual attention - no automation      |
| *(no label)*   | PR is BLOCKED until a valid label is added              |

---

## Setup - 3 Steps

### Step 1 - Create the Labels

In your repository go to **Issues > Labels > New label** and create these three:

| Label name     | Suggested color | Purpose                      |
|----------------|-----------------|------------------------------|
| `red-light`    | `#EE0701`       | Needs review before merging  |
| `green-light`  | `#0075CA`       | Safe to auto-merge           |
| `yellow-light` | `#E4E669`       | Caution - merge manually     |

> **Important:** Label names are case-sensitive. Use lowercase exactly as shown above.

---

### Step 2 - Enable Auto-Merge in Repo Settings

> Required for `green-light` to work. If you skip this, green-light PRs will fail
> with a clear error message pointing you back here.

1. Go to your repository **Settings > General**
2. Scroll to the **Pull Requests** section
3. Check **"Allow auto-merge"**

> You must be a repository **Admin** to change this setting.

---

### Step 3 - Copy the Workflow File

1. In your repository, create the path `.github/workflows/` if it does not exist
2. Create a new file at `.github/workflows/traffic-light.yml`
3. Copy the full contents of [`.github/workflows/traffic-light.yml`](./.github/workflows/traffic-light.yml) into it
4. Commit and push to your default branch (`main` or `master`)

> GitHub Actions activates automatically. No further configuration needed.

---

### Step 4 (Optional) - Enable AI Auto-Labeling

Want the system to automatically apply the correct label based on the code diff and your custom rules? 
Just add your OpenRouter API key as a repository secret and provide a `.github/ai-rules.csv`. 

> See [USAGE.md](./USAGE.md#ai-auto-labeling) for the complete 2-step setup guide.

---

## How It Works

### No Label -> PR Blocked
When a PR is opened without a traffic light label, the GitHub Actions check **fails**
and blocks the merge. The developer must add a valid label to proceed.

### `red-light` -> Review Requested
A review is automatically requested from the **repository owner**.

**Want to customise who gets the review?**
Open `.github/workflows/traffic-light.yml` and update the reviewers line:
```yaml
# Specific users:
reviewers: ['alice', 'bob']

# A GitHub team (use the team slug, not the display name):
team_reviewers: ['backend-team']
```

### `green-light` -> Auto-Merge Enabled
GitHub's native auto-merge is activated. The PR merges automatically once
all required status checks pass.

### `yellow-light` -> Manual Merge Required
The PR passes validation but no automation runs. A human must review and
merge it manually.

---

## Repository Structure

```
.github/
  workflows/
    traffic-light.yml   <- The only file you need to copy into your repo
README.md               <- This file
USAGE.md                <- Detailed setup, customisation and troubleshooting
CONTRIBUTING.md         <- How to contribute improvements
LICENSE                 <- MIT License
```

---

## Full Documentation

See [USAGE.md](./USAGE.md) for detailed setup instructions, customisation options,
and a full troubleshooting guide.

## License

[MIT](./LICENSE) - free to use in any repository, commercial or personal.
