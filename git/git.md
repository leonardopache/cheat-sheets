# Git Cheatsheet


Command  | Description 
------------- | -------------
|`$> git config --global credential.helper manager-core\` | Set git window credential Manager, in first push ask for token|
|`$> git branch -v -a \|\| git fetch` <br> `$> git fetch --all --prune`| it shows all branches available for checkout --all update local with remote data --prune clean deleted branch|
|`$> git checkout brancheName`|Switch with branches|
|`$> git checkout -b branch-name origin/branch-name`|Checkout remote branch|
|`$> git branch`|Show branches local|
|`$> git branch --delete <branchname>`|Delete branch local|
|`$> git for-each-ref --format='%(if)%(authorname)%(then)%(authorname)%(end): %(refname)' --sort=authorname`|List branches remote and local with author|
|`$> git log -p`|Show history |
|`$> git push \|\| git push origin/master`|Push (send to repository)|
|`$> git pull` <br>`$> git pull --rebase`|Pull (update)|
|`$> git commit -m "commity msg"`|Commit with message|
|`$> git reset HEAD <directoryName>`|Reset with the HEAD version|
|`$> git checkout -- <archivo> `|Reset file|
|`$> git log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit`|Such a beautiful graph historic|
|`$> git diff --name-only branch1 branch2`|Just file names, without print the content diff|
|`$> git merge --abort` \|\| `git reset --hard`|...|
|`$> git checkout HEAD -- my-file`|Reset to head only one file |
|`$> git config --get remote.origin.url` <br> `$> git config --global -e`|Show complete URL of repository <br> Open editor with config global information|
|`$> git fetch --all`<br>`$> git pull --no-commit`<br>`$> git reset --hard '@{u}'`|Update branch cleaning local changes and not merging|
|`$> git diff --cached --diff-filter=ACMR --name-only -- "*.java"`|Show list of files changed |
|`$> git log branc1..branch2 --pretty=format:"%h%x09%an%x09%ad%x09%s"`|Show difference between two branches  in line with only important information|
|....|Git branch link|
|Git merge feature in master<br> `$> git checkout master` <br> `$> git merge feature` <br> If conflict <br> #### To select the changes done in master <br> `$> git checkout --ours {file_name or .(all)}` <br> #### To select the changes done in feature <br> `$> git checkout --theirs {file_name or .(all)}` <br> then <br> `$> git add {file_name or .(all)}` <br> `$> git merge --continue` |Merge easy solve conflict command line|
|Git Rebase master in feature <br>`$> git checkout feature` <br> `$> git rebase master` <br> if conflict <br> To select the changes done in master <br> `$> git checkout --ours {file_name or .(all)}` <br> To select the changes done in feature <br> `$> git checkout --theirs {file_name or .(all)}` <br> then <br> `$> git add {file_name or .(all)}` <br> `$> git rebase --continue` |Rebase easy solve conflict command line| 
|`$> git config --global alias.st "status"`|Create alias|
|`$> git stash save -u name_relevante_for_identification`|Create the stash with an identification name. (-u or --include-untracked)  |
|`$> git stash list`| To see the stash list|
|`$> git stash pop`  \|\|  `$>git stash pop stash@{n} `|Apply the changes stashed and remove from stash list. `stash@{n}` choose N when more the one stash in the list|
|`$> git stash apply`  \|\|  `$>git stash apply stash@{n} `| Apply the changes and keep it in the stash list. `stash@{n}` choose N when more the one stash in the list|
|`$> git stash show`  \|\|  `$>git stash show -p `| See differences and `-p or --patch`  |
|`$> git reset HEAD -- <file or directory>`<br> `$> git restore --staged <file or directory>`|If you need to remove a single file from the staging area. Keep the changes|
|`$> git init`<br>`$> git add . `<br>`$> git commit -m ""`<br>`$> git remote add origin <URL.git>`<br>`$> git branch --set-upstream-to=origin/<branch> <branch>`<br>`$> git pull --rebase` <br> `$> git push`|Push code to new repository|
|`$> git fetch -p (prune)`|Remove ref for local branch not in remote |
|`$> git fetch -p && for branch in $(git for-each-ref --format '%(refname) %(upstream:track)' refs/heads \| awk '$2 == "[gone]" {sub("refs/heads/", "", $1); print $1}'); do echo $branch; done`|List all branches not ref. with a remote. <br> To delete the listed, change the echo for a git branch -D command ( git branch -D $branch)|
|`$> git rebase -i HEAD~X`|It Rebase squash <br> In the branch from HEAD until X commits you want to squash in only one. <br> X is the number of commits to squash.<br> In the editor msg change the lines with "pick" to "squash" all lines marked as squash will be combine|
|`$> git merge --squash [branch/name]`|It Merge squash <br> After the merge create a commit and it will the only one.|
|`$> git checkout feature/branch` <br>`$> git rebase --onto develop feature/branch`|Rebasing the checkout branch (feature) OnTopOf some branch (develop)|
