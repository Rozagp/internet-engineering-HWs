## Question 1: git merge vs. git rebase

### 1. Functionality of Each Command
* **`git merge`**: Integrates changes from a specified branch into your current branch. When the histories have diverged, it creates a new **merge commit** tied to both parent branches, joining them while preserving their historical commit logs.
* **`git rebase`**: Takes commits from the current branch and reapplies them one by one onto the tip of the target branch. Git rewrites history by generating brand-new commits (with new hashes) on top of the target base branch.

---

### 2. Differences in History Structure & Change Integration
* **`git merge`**:
  * **Structure**: Preserves a **non-linear**, branched history graph.
  * **History Preservation**: Keeps the complete historical record intact, explicitly showing when branches were split off and joined back together.
* **`git rebase`**:
  * **Structure**: Produces a **linear**, single-line sequence of commits.
  * **History Preservation**: Rewrites project history to present the work as though it happened sequentially from the latest base commit, without divergence.

---

### 3. Appropriate Scenarios & Rationale

#### A) Scenario for `git merge`
* **Scenario**: Merging a completed feature or pull request into shared, public branches like `main` or `develop`.
* **Rationale**:
  1. **History integrity**: Preserving existing commit IDs avoids rewriting public history, which would cause severe sync problems for other team members.
  2. **Auditability**: Explicit merge commits make it easy to see when features landed, track release milestones, or revert entire features at once.

#### B) Scenario for `git rebase`
* **Scenario**: Updating a local, unpushed feature branch with recent updates from `main` before opening a pull request.
* **Rationale**:
  1. **Clean log**: Eliminates redundant "Merge branch 'main' into feature" clutter.
  2. **Simplified debugging and review**: A straight linear timeline makes running tools like `git bisect` or reviewing the pull request significantly easier.
