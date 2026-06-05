# Pulling Multiple Git Repositories at Once

## Script

```bash
for d in */; do
    if [ -d "$d/.git" ]; then
        echo "===== $d ====="
        git -C "$d" pull
    fi
done
```

---

# Purpose

This script automatically finds all Git repositories inside the current directory and runs `git pull` on each of them.

Instead of manually entering every repository and executing:

```bash
cd repo-name
git pull
```

the script performs the process for all repositories in one go.

---

# Line-by-Line Explanation

## 1. For Loop

```bash
for d in */; do
```

### What it does

`*/` means:

> All directories inside the current directory.

For example, if the current directory contains:

```text
oms/
poorti/
OrderRouting/
ofbiz-oms-udm/
maarg-util/
```

the loop becomes:

```bash
for d in oms/ poorti/ OrderRouting/ ofbiz-oms-udm/ maarg-util/
```

The variable `d` stores one directory name at a time.

### Iterations

#### First Iteration

```bash
d="oms/"
```

#### Second Iteration

```bash
d="poorti/"
```

#### Third Iteration

```bash
d="OrderRouting/"
```

and so on.

---

## 2. Check Whether Directory is a Git Repository

```bash
if [ -d "$d/.git" ]; then
```

### Breakdown

#### `[ ]`

Shell test command.

Equivalent to:

```bash
test -d "$d/.git"
```

#### `-d`

Checks whether a directory exists.

For example:

```bash
[ -d "oms/.git" ]
```

asks:

> Does the directory `oms/.git` exist?

If yes:

```text
true
```

If no:

```text
false
```

### Why Check `.git`?

A Git repository contains a `.git` directory.

Example:

```text
oms/
├── .git/
├── src/
├── build.gradle
```

Therefore:

```bash
[ -d "oms/.git" ]
```

effectively means:

> Is `oms` a Git repository?

---

## 3. Print Repository Name

```bash
echo "===== $d ====="
```

### Purpose

Displays which repository is currently being processed.

Example output:

```text
===== oms/ =====
```

Without this line, Git output from multiple repositories would be difficult to identify.

---

## 4. Pull Latest Changes

```bash
git -C "$d" pull
```

### What is `-C`?

Normally:

```bash
cd oms
git pull
```

means:

1. Enter the directory
2. Execute `git pull`

Git provides a shortcut:

```bash
git -C oms pull
```

which means:

> Execute the Git command as if you were inside the `oms` directory.

No actual `cd` command is required.

### Example

If:

```bash
d="oms/"
```

then:

```bash
git -C "$d" pull
```

becomes:

```bash
git -C oms/ pull
```

which is equivalent to:

```bash
cd oms
git pull
```

---

## 5. End If Block

```bash
fi
```

Marks the end of the `if` statement.

---

## 6. End Loop

```bash
done
```

Marks the end of the `for` loop.

---

# Example Execution

Suppose the current directory contains:

```text
component/
├── oms/
│   └── .git
├── poorti/
│   └── .git
├── OrderRouting/
│   └── .git
├── README/
└── random-folder/
```

### Iteration 1

```bash
d="oms/"
```

Check:

```bash
[ -d "oms/.git" ]
```

Result:

```text
true
```

Execute:

```bash
git -C oms/ pull
```

---

### Iteration 2

```bash
d="poorti/"
```

Check:

```bash
[ -d "poorti/.git" ]
```

Result:

```text
true
```

Execute:

```bash
git -C poorti/ pull
```

---

### Iteration 3

```bash
d="README/"
```

Check:

```bash
[ -d "README/.git" ]
```

Result:

```text
false
```

Skip.

---

### Iteration 4

```bash
d="random-folder/"
```

Check:

```bash
[ -d "random-folder/.git" ]
```

Result:

```text
false
```

Skip.

---

# Equivalent Expanded Version

The script is effectively doing:

```bash
if [ -d "oms/.git" ]; then
    git -C oms pull
fi

if [ -d "poorti/.git" ]; then
    git -C poorti pull
fi

if [ -d "OrderRouting/.git" ]; then
    git -C OrderRouting pull
fi
```

but automatically for every directory found.

---

# Key Concepts Learned

* `for` loop → Iterates through directories.
* `*/` → Matches all directories in the current location.
* `d` → Loop variable containing the current directory name.
* `[ ]` → Shell test command.
* `-d` → Checks whether a directory exists.
* `.git` → Identifies a Git repository.
* `echo` → Prints repository names for readability.
* `git -C <dir>` → Executes Git commands inside another directory without using `cd`.
* `git pull` → Fetches and merges the latest changes from the remote repository.

---

# Practical Use in Maarg

Since the following are separate repositories:

```text
runtime/component/oms
runtime/component/poorti
runtime/component/OrderRouting
runtime/component/ofbiz-oms-udm
runtime/component/ofbiz-oms-usl
runtime/component/maarg-util
runtime/component/mantle-shopify-connector
runtime/component/shopify-oms-bridge
runtime/component/maarg-docker-config
```

running the script from:

```bash
cd runtime/component
```

will pull the latest changes for all component repositories automatically.

