This is a minimalist doc with commands and summary info. For details, see https://gist.github.com/adam-m-jcbs/1ce3f4d3d3c2c7e0d73a1dad66ffc7c7

### Subtrees in this repo
`Projects/XRB/Sensitivity/datarepo` <--> git@github.com:adam-m-jcbs/xrb-sens-datashare.git

### Add remote
```
$ git remote add subrepo-remote-label  <remote url or equivalent>
```

### Setup branch for subtree management while configuring remote
```
$ git checkout -b subrepo_branch subrepo-remote-label/master
```

### Pull upstream changes
```
$ git fetch subrepo-remote-label 
$ git merge -X subtree=Path/to/subrepo_root --squash --allow-unrelated -Xtheirs subrepo-remote-label/master
$ git commit -m'Update Path/to/subrepo_root with latest changes from subrepo-remote-label'
```

### Push changes to subtree (backport)  
Note: still note sure if there's a way to maintain a "share branch".  In practice, I can't use the branch idiom above with a branch filter.  For now, I make a temporary branch and destroy it after.
```
(master)$ git checkout -b subrepo_branch
(subrepo_branch)$ git filter-branch --subdirectory-filter Path/to/subrepo_root
(subrepo_branch)$ git push -u subrepo-remote-label subrepo_branch:master
```

### Add existing repo as subtree
This assumes you've already setup the remote
```
$ git fetch subrepo-remote-label
$ git read-tree --prefix=Path/to/subrepo_root -u subrepo-remote-label/master
$ git commit -m'Add subrepo-remote-label as a subtree at Path/to/subrepo_root'
```

### Cherry-pick a commit instead of filter-branch
I prefer filter-branch, but for reference:
```
(master)$ git checkout subrepo_branch
(subrepo_branch)$ git log master
[note commit(s) you want to cherry pick]
(subrepo_branch)$ git cherry-pick -x -Xsubtree=Path/to/subrepo_root <commit-like, e.g. 8a3bcce, master~3, and the like>
[repeat for each commit you want]
(subrepo_branch)$ git push -u subrepo-remote-label subrepo_branch:master
```
I've also found `--strategy=subtree -Xpatience` to be useful for cherry-pick 
