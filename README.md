# 🚦 PR Traffic Light System

Automate your Pull Request workflow using simple labels. This system allows you to automatically request reviews for "Red" (high-risk/important) PRs and auto-merge "Green" (low-risk/routine) tasks once CI passes.

## 🛡 New: "Safety Catch" Validation
We've added a safety mechanism! If a developer opens a PR **without a traffic light label**, the GitHub Action status check will **FAIL**. This blocks the PR from merging accidentally until they choose a valid traffic light category.

## 🚀 Quick Start (Setup in 2 Minutes)

To implement this into your repository, follow these 3 steps:

### 1. Create the Labels
In your repository, go to **Issues > Labels** and create the following:
*   `red-light`: Red color. (Review required)
*   `green-light`: Green color. (Auto-merge allowed)
*   `yellow-light`: Yellow color. (Manual intervention)

### 2. Enable Auto-Merge in Settings
GitHub requires you to explicitly allow auto-merging:
1.  Go to **Settings > General**.
2.  Scroll down to the **Pull Requests** section.
3.  Check the box **"Allow auto-merge"**.

### 3. Add the Workflow File
Create a new file in your repo at `.github/workflows/traffic-light.yml` and paste the content from [traffic-light-template.yml](./traffic-light-template.yml).

---

## 🛠 How it Works

### 🛑 Validation (No Tag)
If no tag is present, the status check **fails**. This ensures every PR is intentionally categorized.

### 🔴 Red Light (`red-light`)
- The system automatically requests a review from the **Repository Owner**.

### 🟢 Green Light (`green-light`)
- The system enables GitHub's native **Auto-Merge**.
- The PR will merge as soon as all checks pass.

### 🟡 Yellow Light (`yellow-light`)
- Used as a visual indicator for "Proceed with Caution". 

---

## 🛡 Security Note
Only users with **Write** access can apply labels. This ensures only authorized team members can trigger the auto-merge or review request features.

## 📄 License
This project is licensed under the [MIT License](./LICENSE).
