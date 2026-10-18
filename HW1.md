## Question 1

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
 

## Question 2

### 1. Definition of a Hunk in Git Diff Output
In Git's diff output, a **hunk** refers to a discrete, contiguous block of modified lines (added, deleted, or changed) presented alongside a few lines of unmodified surrounding context lines. Git groups nearby edits into individual hunks, with each hunk preceded by a header showing original and new line ranges (e.g., `@@ -12,6 +12,8 @@`). This modular division allows us to view and stage distinct modifications independently, even when they occur within the same file.

---

### 2. Interactively Reviewing and Staging Specific Hunks via CLI
To review changes interactively and pick specific hunks for the next commit, we execute:

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



# Question 3

## 1. How git bisect Works
`git bisect` is a built-in Git debugging tool that runs a **binary search** across the project history to hunt down the commit that broke something.

Instead of testing commits one by one:
- We tell Git one commit where the code is broken (`bad`) and an earlier commit where everything still worked (`good`).
- Git splits that range in half and automatically checks out the commit right in the middle.
- We test whether the bug is present at that middle commit.
- Depending on our answer, Git cuts the remaining search space in half again and repeats the process until it isolates the single commit responsible for the bug.

---

## 2. Step-by-Step Workflow

1. **Start the bisect session:**
   ```bash
   git bisect start
   ```

2. **Tag the broken commit:**
   Usually this is wherever we are right now (`HEAD`):
   ```bash
   git bisect bad
   ```

3. **Tag the last known good commit:**
   Provide a commit hash or tag where the feature was working:
   ```bash
   git bisect good <commit-hash-or-tag>
   ```

4. **Test the midpoint:**
   Git checks out a commit halfway between good and bad. Run the app or tests:
   - If it is broken:
     ```bash
     git bisect bad
     ```
   - If it works fine:
     ```bash
     git bisect good
     ```

5. **Let Git find the problem:**
   Repeat step 4 each time Git serves up a new midpoint. Once finished, Git displays the offending commit:
   ```text
   <commit-hash> is the first bad commit
   ```

6. **Reset the working tree:**
   End the session and return to the branch/commit where you started:
   ```bash
   git bisect reset
   ```

---

## 3. Practical Example
For example user authentication crashes on `main` today:
- We know it is broken right now at `HEAD`.
- We know it worked fine during release `v2.1.0` three weeks ago, but there have been 200 commits since then.

We run:
```bash
git bisect start
git bisect bad HEAD
git bisect good v2.1.0
```
Git checks out the commit halfway through the history. You run your auth test (`npm test auth`). 
- If the test passes, you run `git bisect good`.
- Git halves the remaining candidate commits and checks out the next point.

After about 8 iterations, Git identifies the exact commit:
```text
e4b102c is the first bad commit
Author: Alex <alex@example.com>
Date:   Wed Oct 14 11:20:00 2026
Commit Message: Refactor session validation middleware
```

We can also automate the whole loop if we have an automated test script:
```bash
git bisect run npm test
```

---

## 4. Why This Method Is Effective
- **Saves huge amounts of time:** Instead of sifting through hundreds of commits linearly ($O(N)$), binary search cuts the search down exponentially (log N).
- **No guesswork:** We do not need to guess which PR or file might have caused the issue; Git methodically eliminates irrelevant commits.
- **Immediate diff clarity:** Once identified, running `git show <commit-hash>` displays the exact line changes that introduced the defect, making a fix or revert simple.



# Question 4

## Scenario Overview
* The file `README.md` is tracked and already committed in the past.
* We staged some new edits to `README.md`.
* Afterward, we made further edits to `README.md` in the working directory without running `git add` again.
* We also have an untracked directory named `tmp/` containing temporary files.

---

## 1. Inspecting Changes and Repository Status

### Commands to Run
* **Check overall status:**
  ```bash
  git status
  ```
  This provides a high-level summary showing:
  - Changes staged for commit (the first round of edits to `README.md`).
  - Changes not staged for commit (the second round of edits to `README.md`).
  - Untracked files (the `tmp/` folder).

* **View unstaged changes (working tree vs. staging area):**
  ```bash
  git diff
  # Or specifically for the file:
  git diff README.md
  ```
  This shows the modifications we made to `README.md` *after* running the previous `git add`.

* **View staged changes (staging area vs. last commit / HEAD):**
  ```bash
  git diff --staged
  # Or:
  git diff --cached
  ```
  This shows the modifications in `README.md` that are sitting in the index and actually ready to be committed.

### Difference Between the Two Sets of Changes
* **Staged changes (`git diff --staged`):** A snapshot of modifications explicitly added to the Git index. Git will include exactly these lines in the next commit.
* **Unstaged changes (`git diff`):** Edits currently present only in our working tree. Git tracks the file, but these specific line changes have not yet been marked for inclusion in the upcoming commit.

---

## 2. Committing Without the `-a` Flag

### What Gets Committed and What Remains?
* **What gets committed:** Only the changes to `README.md` that were staged *before* the subsequent edits.
* **What stays behind:** 
  - The later unstaged edits to `README.md` remain untouched in our working directory.
  - The entire `tmp/` directory remains untracked and uncommitted in our working directory.

### Why This Happens
Running `git commit -m "message"` (without `-a`) records a snapshot strictly from the **staging area (index)**, not the current state of our working directory. Because Git captures file state at the exact moment you run `git add`, any modifications made afterward do not automatically update the staging area unless staged again.

---

## 3. Preventing `tmp/` from Being Added to Future Commits

### Configuration and Target File
Add an entry for the folder inside the repository's `.gitignore` file located at the root of the project:

```gitignore
tmp/
```

Once saved in `.gitignore`, Git will ignore the `tmp/` directory and all files inside it, keeping them out of `git status` output and preventing commands like `git add .` from staging them.

### Is This Enough for Already-Tracked Files?
* No.`.gitignore` only tells Git to ignore **untracked** files. If a file has already been committed and is actively tracked by Git, adding its path to `.gitignore` does nothing—Git will continue tracking its changes.
* **How to fix if already tracked:** We must explicitly remove it from Git's index while keeping the local file on disk:
  ```bash
  git rm --cached <file-path>
  # For an entire directory:
  git rm -r --cached tmp/
  ```
  After untracking it, we need to commit that removal. From that point on, `.gitignore` will properly ignore future changes to it.




# Question 5

## Scenario Overview
* Us and a teammate branched off from the same base commit.
* Our teammate worked on `main`, while you worked on `feature-description`.
* Both of us edited the exact same line in `README.md` with different text and committed the changes.
* Your working tree is clean. When merging `feature-description` into `main`, Git reports a merge conflict on that line.

---

## 1. Why Did This Conflict Occur?

### Root Cause
Git can merge diverged branches automatically when changes occur in different files or even in different parts of the same file. However, a conflict occurred here because both branches modified the exact same line relative to their common ancestor commit.

Git cannot guess developer intent or know which version is the correct one. Rather than arbitrarily picking one person's work and discarding the other, Git pauses the merge and asks a human to resolve the discrepancy.

### Does Every Two-Person Edit on the Same File Cause a Conflict?
* No.
* If two collaborators modify different lines or different sections of the same file, Git's three-way merge algorithm merges them automatically without human intervention. A conflict is triggered only when changes overlap or affect the immediate surrounding context lines.

---

## 2. Identifying Conflicted Files and Understanding Conflict Markers

### Identifying the Conflicted Files
To see which files have conflicts, we can run:
```bash
git status
```
Git will list conflicted files under the heading:
```text
Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   README.md
```

### Understanding Conflict Markers
Opening `README.md` reveals Git's built-in conflict markers around the affected lines:

```text
<<<<<<< HEAD
This is the updated line written on the main branch.
=======
This is the updated line written on the feature-description branch.
>>>>>>> feature-description
```

#### Meaning of Each Marker:
* **`<<<<<<< HEAD`**: Marks the beginning of the conflicting block and shows the version present on your current checkout target branch (`main`).
* **`=======`**: The separator line dividing the two competing versions.
* **`>>>>>>> feature-description`**: Marks the end of the conflicting section and indicates the incoming branch name or commit hash where the alternate change came from.

---

## 3. Steps to Keep Both Changes and Complete the Merge

To combine both sets of information without losing either teammate's contribution:

### Step 1: Open the file and manually edit the content
We can open `README.md` in text editor. Delete the marker lines (`<<<<<<<`, `=======`, and `>>>>>>>`) entirely and edit the text so that both ideas/sentences are combined logically:

```text
This is the updated line written on the main branch.
This is the updated line written on the feature-description branch.
```

### Step 2: Ensure no conflict markers remain
Check that all markers were removed cleanly by searching for residual syntax:
```bash
grep -rn "<<<<<<<" .
# or
git diff
```
If nothing turns up, all conflict markers have been eliminated.

### Step 3: Stage the resolved file
The cleaned file can be staged to tell Git that this conflict has been settled:
```bash
git add README.md
```

### Step 4: Finalize the merge commit
Complete the merge operation:
```bash
git commit
```
Git opens the default editor with a pre-filled merge commit message (e.g., `Merge branch 'feature-description'`). To complete the merge, we have to save and close the editor.

---

## 4. How to Abort an In-Progress Merge

If after inspecting the conflict we decide that now is not the right time to merge, we can cancel the operation and restore the repo to its pre-merge state by running:

```bash
git merge --abort
```

### this command:
* Cleans up all unresolved conflict markers from your working tree.
* Resets the staging area and working directory back to the state of `HEAD` before `git merge` was invoked.


# Question 6

## Scenario 1:

### Goal
We made changes to a tracked file called `notes.txt`. We no longer want these changes. They have not been staged, and the version currently in the staging area matches the last commit. We want to discard only this file's working tree edits while leaving all other files intact.

### Command
```bash
git restore notes.txt
```

### Impact
* **Working Tree:** The edits made to `notes.txt` are erased. The file is reverted back to match the staged snapshot (and latest commit). Other files are completely untouched.
* **Staging Area:** Unaffected (remains identical to the last commit).
* **Commit History:** Unaffected (no new commits are made, and no existing commits change).

---

## Scenario 2

### Goal
We have accidentally run `git add notes.txt`, placing unwanted staged changes into the index. We have not committed yet. We want to remove `notes.txt` from the staging area so it won't be included in the next commit, but keep all the local edits safely in thee working tree.

### Command
```bash
git restore --staged notes.txt
```

### Impact
* **Working Tree:** Unaffected. All your edits inside `notes.txt` remain intact on disk in your working tree.
* **Staging Area:** The staged changes for `notes.txt` are removed from the index. The index now reflects the state from the previous commit.
* **Commit History:** Unaffected (no commits are modified or created).

---

## Scenario 3

### Goal
A standard commit (not a merge commit) was pushed to the shared `main` branch, and teammates have already pulled it. The working tree is clean. We want to undo the effect of that commit without deleting it or rewriting history.

### Recommended Command
```bash
git revert <commit-hash>
git push origin main
```

### Impact
* **Working Tree & Staging Area:** Git computes the exact inverse of the changes introduced in `<commit-hash>` and applies them to your working tree and staging area.
* **Commit History:** A brand-new commit is created that reverses the bad commit. The existing history is strictly preserved and moves forward linearly without any deletion of previous commits.

### Why `git reset` and `git push --force` are wrong
1. `git reset --hard` deletes the commit locally. Pushing that with `--force` rewrites the remote branch's history. Because teammates have already pulled the bad commit, their local branches will diverge from the remote branch. When they next pull or push, Git will attempt to reconcile the divergent branches, typically causing merge conflicts or reintroducing the bad commit via an accidental merge.
2. In a shared production branch like `main`, preserving an unbroken, transparent audit log is critical. `git revert` is safe and collaborative because it records the reversal as an explicit, traceable event that team members can pull cleanly with a standard `git pull`.
