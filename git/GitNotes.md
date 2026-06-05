# Git Notes – OFBiz Learnings Repository

## 1. Check Repository Status

### Basic Status

```bash
git status
```

Shows:

* Modified files
* Staged files
* Untracked files

### Show All Untracked Files

```bash
git status -uall
```

Useful when Git only shows a folder name instead of all untracked files.

Example:

```text
On branch master

Untracked files:
    Code BreakDowns/Identifier Of MixedCart Order Decesion Tree.txt
    git/Pulling Multiple Git Repositories.md
```

---

## 2. Add Files to Staging

### Add Everything

```bash
git add .
```

### Add Specific Folder

```bash
git add "Assignments/Module5/productinformationmgr/"
```

### Stage All Changes Including Deletions

```bash
git add -A
```

---

## 3. Review Staged Changes

### View Staged Changes

```bash
git diff --cached
```

Shows exactly what will be included in the next commit.

### View Summary of Staged Changes

```bash
git diff --cached --summary
```

Example:

```text
rename GitNotes.md => git/GitNotes.md (100%)
create mode 100644 git/Pulling Multiple Git Repositories.md
```

---

## 4. Unstage Changes

### Unstage a Specific File

```bash
git restore --staged <file-name>
```

Example:

```bash
git restore --staged "Code BreakDowns/Identifier Of MixedCart Order Decesion Tree.txt"
```

### Unstage Everything

```bash
git restore --staged .
```

Useful when you want to reorganize commits.

---

## 5. Commit Changes

```bash
git commit -m "Your commit message"
```

Example:

```bash
git commit -m "Updated Module5 ProductInformationMGR"
```

---

## 6. Push Changes

### Normal Push

```bash
git push origin main
```

### Force Push

```bash
git push --force origin main
```

Use force push carefully.

---

## 7. Undo / Remove Commits

### Remove Latest Commit Locally

```bash
git reset --hard HEAD~1
```

### Safer Alternative

```bash
git revert <commit-id>
git push origin main
```

Creates a new commit that reverses the changes.

---

## 8. Copy Updated Project Files into Repository

### Copy Normal Files

```bash
cp -r /home/rachitaarsh/OFBiz/plugins/productinformationmgr/* \
~/OFBiz-Learnings/Assignments/Module5/productinformationmgr/
```

### Copy Hidden Files Too

```bash
cp -r /home/rachitaarsh/OFBiz/plugins/productinformationmgr/. \
~/OFBiz-Learnings/Assignments/Module5/productinformationmgr/
```

---

## 9. Remove Wrong Folder from Git

```bash
git rm -r Assignments/Modue5
git commit -m "Remove incorrect Modue5 folder"
git push --force origin main
```

---

## 10. Restore Deleted Files

### Restore Specific File/Folder

```bash
git restore Assignments/Modue5
```

### Restore Everything

```bash
git restore .
```

---

## 11. View Commit History

### Full History

```bash
git log --oneline
```

### Last 5 Commits

```bash
git log --oneline -5
```

---

## 12. Compare Changes

```bash
git diff
```

Shows unstaged changes.

---

## 13. View Folder Structure Recursively

```bash
ls -R git
```

Example:

```text
git:
GitNotes.md
Pulling Multiple Git Repositories.md
```

Nested Example:

```text
git:
GitNotes.md
Advanced/

git/Advanced:
Rebase.md
CherryPick.md
```

Useful for verifying moved files and directory structure.

---

## 14. Check If a File Is Ignored

```bash
git check-ignore -v <file-name>
```

Example:

```bash
git check-ignore -v logs/debug.log
```

Output:

```text
.gitignore:12:logs/ logs/debug.log
```

Shows which `.gitignore` rule is causing a file to be ignored.

---

## 15. Save GitHub Token

### Permanent

```bash
git config --global credential.helper store
```

### Temporary (10 Hours)

```bash
git config --global credential.helper 'cache --timeout=36000'
```

---

## 16. Ignore Personal Files

Add file to `.gitignore`:

```bash
echo "filename" >> .gitignore
```

Then:

```bash
git add .gitignore
git commit -m "Ignore personal files"
```

---

## 17. Troubleshooting

### Check Connectivity

```bash
ping github.com
nslookup github.com
sudo systemctl restart NetworkManager
```

---

## 18. Pull Latest Changes from HotWax Codebase

If Git reports local changes preventing a pull:

```bash
git stash
git pull origin main
git stash pop
```

Alternative:

Navigate into each repository/component and pull individually.

```bash
cd component-name
git pull
```

---

## 19. Pull Multiple Git Repositories at Once

Run from a directory containing multiple Git repositories.

```bash
for d in */; do
    if [ -d "$d/.git" ]; then
        echo "===== $d ====="
        git -C "$d" pull
    fi
done
```

### How It Works

* `*/` → Iterates through all directories.
* `[ -d "$d/.git" ]` → Checks if directory is a Git repository.
* `echo "===== $d ====="` → Prints repository name.
* `git -C "$d" pull` → Executes pull inside repository.

Example Output:

```text
===== runtime/ =====
Already up to date.

===== mantle-shopify-connector/ =====
Updating 4e3ab12..9d7c5fe
Fast-forward
```

---

## 20. Typical Daily Workflow

```bash
git status
git add .
git commit -m "Meaningful message"
git push origin main
```

If something wrong gets pushed:

```bash
git reset --hard HEAD~1
git push --force origin main
```

