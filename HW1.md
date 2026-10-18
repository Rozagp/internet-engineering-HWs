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
 

## Question 2: Selecting Partial Changes (Hunks)

### 1. Definition of a Hunk in Git Diff Output
In Git's diff output, a **hunk** refers to a discrete, contiguous block of modified lines (added, deleted, or changed) presented alongside a few lines of unmodified surrounding context lines. Git groups nearby edits into individual hunks, with each hunk preceded by a header showing original and new line ranges (e.g., `@@ -12,6 +12,8 @@`). This modular division allows you to view and stage distinct modifications independently, even when they occur within the same file.

---

### 2. Interactively Reviewing and Staging Specific Hunks via CLI
To review changes interactively and pick specific hunks for the next commit, execute:

```bash
git add -p <file_path>
# or
git add --patch <file_path>
```

#### Key Prompts and Options
For every detected hunk, Git prompts with `Stage this hunk [y,n,q,a,d,s,e,?]?`. The key actions include:

* **`y`**: Stage this hunk into the staging area (index).
* **`n`**: Do not stage this hunk (leave it in the working tree).
* **`q`**: Quit the interactive session; any hunks already selected remain staged.
* **`a`**: Stage this hunk and all subsequent hunks in this file.
* **`d`**: Skip this hunk and all subsequent hunks in this file.
* **`s`**: Split the current hunk into smaller distinct sub-hunks (if separated by unchanged lines).
* **`e`**: Manually edit the hunk in your default text editor to stage changes line-by-line.
* **`?`**: Print a quick help menu explaining all available options.

---

### 3. Benefits for Creating Cohesive (Atomic) Commits
* **Logical Separation of Concerns (Atomic Commits):** Prevents mixing unrelated modifications (e.g., refactoring existing code, fixing a distinct bug, and temporary debug logging) in a single commit, enabling you to keep each commit focused on one specific task.
* **Cleaner and Easier Code Reviews:** Reviewers can inspect isolated changes with clear, focused intent instead of parsing a tangled diff.
* **Safer Rollbacks and Cherry-Picking:** When commits are strictly atomic, individual bug fixes or features can be reverted (`git revert`) or backported (`git cherry-pick`) cleanly without causing accidental regressions or dragging along unrelated code.
