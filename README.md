# 🚦 PR Traffic Light System

Automate your Pull Request workflow using simple labels. This system allows you to automatically request reviews for "Red" (high-risk/important) PRs and auto-merge "Green" (low-risk/routine) tasks once CI passes.

## 🚀 Quick Start (Setup in 2 Minutes)

To implement this into your repository, follow these 3 steps:

### 1. Create the Labels
In your repository, go to **Issues > Labels** and create the following:
*   `red-light`: Red color. (Used for PRs that MUST be reviewed)
*   `green-light`: Green color. (Used for PRs that can be auto-merged)

### 2. Enable Auto-Merge in Settings
GitHub requires you to explicitly allow auto-merging:
1.  Go to **Settings > General**.
2.  Scroll down to the **Pull Requests** section.
3.  Check the box **"Allow auto-merge"**.

### 3. Add the Workflow File
Create a new file in your repo at `.github/workflows/traffic-light.yml` and paste the content from [traffic-light-template.yml](./traffic-light-template.yml).

---

## 🛠 How it Works

### 🔴 Red Light (`red-light`)
When this label is added to a PR:
- The system automatically requests a review from the **Repository Owner**.
- *Customization:* You can edit the `reviewers` array in the YAML file to include specific handles or teams.

### 🟢 Green Light (`green-light`)
When this label is added to a PR:
- The system enables GitHub's native **Auto-Merge**.
- The PR will merge automatically as soon as all required status checks (tests, linting, etc.) pass.

### 🟡 Yellow Light (`yellow-light`) (Optional)
- Use this for PRs that require manual attention but don't follow the automated paths. It serves as a visual indicator for "Proceed with Caution".

---

## 🛡 Security Note
Only users with **Write** access to the repository can apply labels. This ensures that only authorized team members can trigger the auto-merge or review request features.

## 📄 License
This project is licensed under the [MIT License](./LICENSE).
