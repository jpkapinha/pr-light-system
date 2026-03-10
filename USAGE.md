# 🚦 Usage Guide

The Traffic Light System is designed to be a one-copy implementation for any GitHub repository.

## Prerequisites
*   You must be an **Admin** of the target repository to enable **Auto-Merge**.
*   The repository must have **GitHub Actions** enabled.

---

## 🔧 Step-By-Step Setup

### Step 1: Create Labels
1. Go to your GitHub repository.
2. Click on **Issues** -> **Labels**.
3. Create a new label called `red-light` (red color).
4. Create a new label called `green-light` (green color).

### Step 2: Configure Repo Settings
1. Go to **Settings** -> **General**.
2. Scroll to the **Pull Requests** section.
3. Check the box **"Allow auto-merge"**.

### Step 3: Deploy the Workflow
1. Navigate to the root of your repository.
2. Create the following directory path: `.github/workflows/`
3. Download the [traffic-light-template.yml](./traffic-light-template.yml) file.
4. Upload it to your created directory as `traffic-light.yml`.

---

## 🎨 Customizing Reviewers
By default, the `red-light` label requests a review from the individual who owns the repository. 
To add specific reviewers:
1. Open `.github/workflows/traffic-light.yml`.
2. Locate the `reviewers: [owner]` line.
3. Change it to: `reviewers: ['username1', 'username2']` or use teams: `team_reviewers: ['security-team']`.

---

## 🧠 Why use this?
*   **Speed:** Low-risk PRs (docs, formatting, tests) don't need back-and-forth approval.
*   **Security:** High-risk PRs (critical logic, auth) are automatically flagged for senior review.
*   **Reduced Friction:** Developers don't need to manually check for approval to merge a routine task.
