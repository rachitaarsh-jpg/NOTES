# Git Notes -- OFBiz Learnings Repository

## 1. Check current repository status

``` bash
git status
```

## 2. Add files to staging

Add everything:

``` bash
git add .
```

Add specific folder:

``` bash
git add "Assignments/Module5/productinformationmgr/"
```

Stage all changes including deletions:

``` bash
git add -A
```

## 3. Commit changes

``` bash
git commit -m "Your commit message"
```

Example:

``` bash
git commit -m "Updated Module5 ProductInformationMGR"
```

## 4. Push changes

Normal push:

``` bash
git push origin main
```

Force push:

``` bash
git push --force origin main
```

## Undo / Remove pushed commits

Remove latest commit locally:

``` bash
git reset --hard HEAD~1
```

Alternative safer method:

``` bash
git revert <commit-id>
git push origin main
```

## Copy updated project files into repo

``` bash
cp -r /home/rachitaarsh/OFBiz/plugins/productinformationmgr/* ~/OFBiz-Learnings/Assignments/Module5/productinformationmgr/
```

Copy hidden files too:

``` bash
cp -r /home/rachitaarsh/OFBiz/plugins/productinformationmgr/. ~/OFBiz-Learnings/Assignments/Module5/productinformationmgr/
```

## Remove wrong folder from Git

``` bash
git rm -r Assignments/Modue5
git commit -m "Remove incorrect Modue5 folder"
git push --force origin main
```

## Restore deleted files

``` bash
git restore Assignments/Modue5
```

Restore all:

``` bash
git restore .
```

## View commit history

``` bash
git log --oneline
git log --oneline -5
```

## Compare changes

``` bash
git diff
```

## Save GitHub token

Permanent:

``` bash
git config --global credential.helper store
```

Temporary (10h):

``` bash
git config --global credential.helper 'cache --timeout=36000'
```

## Troubleshooting

Check connectivity:

``` bash
ping github.com
nslookup github.com
sudo systemctl restart NetworkManager
```

## Typical workflow

``` bash
git status
git add .
git commit -m "Meaningful message"
git push origin main
```

If you accidentally pushed something:

``` bash
git reset --hard HEAD~1
git push --force origin main
```



##To stop Git from showing untracked files repeatedly, add them to .gitignore:

``` echo "filename" >> .gitignore
Then:
git add .gitignore && git commit -m "Ignore personal files"
```


##Command To pull from code base HOTWAX of it shows commit error 
    git stash
    git pull origin main
    git stash pop
```else just go to each component and pull one by one
