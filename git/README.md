**1. What is the difference between `git merge` and `git rebase`?**
The key difference between git merge and git rebase lies in how they integrate changes from one branch into another. 
- `git merge` combines two branches by creating a new merge commit that ties together their histories. This preserves the chronological record and showing all diverging paths. It is useful when collaborating in teams where understanding the branch structure is important. 
- `git rebase` rewrites the commit history by taking the changes from one branch and applying them on top of another. It creates a cleaner, linear history without a merge commit. This makes the history easier to read but can be risky on shared branches since it rewrites commits.

---

**2. What is the use of `git stash` command?**
The git stash command is used to temporarily save changes in your working directory that you’re not ready to commit. It allows you to switch branches or perform other tasks without losing your progress.Later, you can restore the stashed changes using `git stash apply` or `git stash pop`.

---

**3. What are different types of merges in git?**
- **Three-way merge:** A three-way merge occurs when both the branches (source and target) have diverged. Git looks at three points: the common ancestor (base), the tip of the current branch (HEAD), and the tip of the branch being merged. It then merges all changes into a new merge commit. This type of merge is used when the branches have independent changes such as when feature branches are merged into main or develop after both have progressed separately. It preserves the full commit history and shows that the branches were merged.
- **Squash merge:** It combines all commits from a feature branch into a single commit before merging them into the target branch. Instead of keeping every individual commit from the feature branch, Git compresses them into one. This helps maintain a linear and readable commit log, especially for small or experimental branches that don’t need detailed commit granularity.
- **Fast-forward merge:** A fast-forward merge happens when the current branch’s HEAD is directly behind the branch being merged. This means no new commits exist on the target branch since the branch was created. In this case, Git simply moves the HEAD pointer forward to the latest commit of the feature branch. This keeps the history linear and clean, merging without creating a new merge commit. 
- **Octopus merge:** It allows you to merge more than two branches at once into a single commit. It’s most commonly used in large-scale integration scenarios where multiple feature branches are combined simultaneously. However, it only works well when there are no conflicting changes among the branches. Otherwise, git will prevent it.
- **Ours / Theirs merge strategy:** The `ours` and `theirs` merge strategies are used to resolve conflicts or enforce precedence during merges. The `ours` strategy keeps the current branch’s changes and discards the other branch’s changes. While the `theirs` strategy does the opposite. These are especially useful in complex merges or rebases where you intentionally want to keep one branch’s content over the other such as when overwriting a deprecated branch.

---

**4. What is the difference between Gitflow and Trunk based development?**
- **Gitflow:**
   - Gitflow is a structured branching strategy designed to manage large and complex software projects with multiple parallel development efforts. It uses several long-lived branches such as `main`, `develop` and additional branches for `features`, `releases` and `hotfixes`. 
   - Developers create `feature` branches from `develop` to work on new features and merge them back after review. 
   - When a release is ready, a `release` branch is created for final testing and bug fixes before merging into both `main` (for production) and `develop` (for continued development). 
   - This model provides clear separation between stages of development and production, making it suitable for teams that follow scheduled releases or work in environments requiring strict version control and testing.
- **Trunk-based development:**
   - Trunk-Based Development is a simpler, more continuous approach to branching where all developers commit their code directly to a main branch called the trunk or main. 
   - Instead of creating long-lived feature branches, developers create short-lived branches that are merged back into main frequently, daily or multiple times a day. 
   - Continuous integration practices ensure that the main branch is always in a deployable state. This encourages smaller, incremental changes, faster feedback cycles and reduced merge conflicts. 

---

**5. What is the difference between `git revert` and `git reset`?**
- **`git revert`:** It is a safe way to undo changes in Git because it doesn’t remove any existing commits. Instead, it creates a new commit that reverses the changes made by a previous one. This maintains a complete history that is ideal for collaborative environments where others may already have pulled the original commits. For example, if you mistakenly introduced a bug in commit `abc123`, you can run `git revert abc123` to generate a new commit that undoes the changes from that commit while preserving the project’s history. This method is recommended when working on shared branches such as `main` or `develop`.
- **`git reset`:** It actually moves the branch pointer backward, effectively rewriting history. It can modify or delete commits, depending on whether you use `--soft`, `--mixed`, or `--hard`. This makes `git reset` potentially dangerous in shared repositories as it changes commit history. This method is recommended in local or private branches where rewriting history won’t affect other collaborators. For example:
    - `git reset --hard HEAD~1`: The `--hard` flag completely removes the latest commit and resets your working directory to the previous state and erases any changes.
    - `git reset --soft HEAD~1`: The `--soft` flag moves the HEAD pointer to a previous commit but keeps all your changes staged.
    - `git reset --mixed HEAD~1`: The `--mixed` flag resets the HEAD to a previous commit and unstages all your changes, but keeps them in your working directory.

---

**6. What is the difference between `git fetch` and `git pull`?**
- **`git fetch`:** It is a safe and read-only command that downloads commits, branches, and tags from a remote repository into your local repository without merging or changing your working directory. It simply updates your remote-tracking branches (like `origin/main`). This allows you to review changes before integrating them. For example, running `git fetch origin` will retrieve the latest changes from the remote repository. But, your `local` main branch will remain unchanged until you explicitly merge or rebase.
- **`git pull`:** It is essentially a shortcut for `git fetch` followed by `git merge` (or `git rebase`). It fetches changes from the remote repository and immediately merges them into your current branch. For example, `git pull origin main` fetches the latest changes from the remote `main` branch and merges them into your local `main`. However, it can sometimes lead to unexpected merge conflicts if you’re not aware of incoming changes. This is why many developers prefer to use `git fetch` first, review the updates and then merge manually.

---

**7. What is a staging area?**
The staging area (index) in Git is an intermediate space where changes are prepared before they are committed to the repository. When you modify files in your working directory, those changes are not automatically included in the next commit. Instead, you must explicitly add them to the staging area using the `git add` command. For example, once you have staged you changes using `git add`, running `git commit` records a snapshot of the staged content in the project’s history.

---

**8. How do you revert a commit?**
- **`git revert`:** `git revert abc123` - This will create a new commit that negates the changes from `abc123` to restore the code to its previous state without losing history.
- **`git reset`:** `git reset --hard HEAD~1` - This will completely remove the last commit and all its changes and revert your branch to the previous commit.
- **`git rebase -i` (Interactive Rebase):** It allows you to edit, combine or remove commits in your branch’s history before pushing them. It’s often used for cleaning up commit history or undoing mistakes in recent commits. For example, `git rebase -i HEAD~3` - This command opens an interactive editor listing your last three commits. You can then choose to `edit`, `drop` or `reword` any of them.
- **`git checkout`:** It can revert a specific file to a previous version from an earlier commit. For example, `git checkout abc123 -- index.html` - It will restore the `index.html` file to how it looked in commit `abc123`, keeping all other files unchanged.

---

