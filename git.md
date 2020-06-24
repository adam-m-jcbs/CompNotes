# Git Notes

Various notes on aspects of git that I keep forgetting or know will be useful.

## useful one-liners (you can alias, but I prefer memorizing first)

```
# Initial config
git config --global user.email "you@email.biz"
git config --global user.name "You There"

# Create and checkout a branch
git checkout -b new_branch

# Push a new branch to remote, setting up tracking
git push -u <remote, e.g. origin> new_branch
```

## Submodules

I love git submodules, but they're also quite bewildering at times.  So I've
written on some of the issues I ran into and documented useful configuration.
git has very good online docs, and [this section on
submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) is a helpful
reference.

[useful medium article](https://medium.com/@dopeddude/git-submodule-with-git-hooks-for-scalable-repos-50924f969937)

### Why submodules

My use-case is that I want to be able to break a huge repo I use to manage all
my research up into little shareable parts, or, conversely, to add little
shareable parts to my huge repo.  Thus, I can have, say, a git repo I host on my
own machine (isn't on GitHub) that contains the source for all my papers.  I can
then add a colleague's git repo for us collaborating on a specific paper as a
submodule to my repo for all of my papers.  Or, if I have a repo for the code
and results of a particular project, I'll often have some directories that contain
massive files.  Especially if I'm analyzing 3D hydro simulations or trying to
render them, I can quickly generate massive output files.  I can keep these
massive files under version control without always lugging them around by breaking them
out into a submodule.

#### Why not LFS for large files?

Yes, there are newer ways of dealing
with large files, namely implementations of GitHub's Large File Storage
extension.  But, this goes beyond vanilla `git`, which I like to stick to
whenever I can.  Also, GitHub's pricing scheme for large file storage renders it
unusable.  You can easily go over the data bandwidth cap for free accounts after
a few people pull your repo and its large files.  My options now are to go to
bitbucket, where most of my colleagues aren't, or to maintain  my own
installations of the `git-lfs` extension on all machines I host a repo on.  None
of these options sounds better to me than just using submodules to manage large
files.

### Basics

+ Getting a repo's submodules
Without intervention, a freshly pulled repo will only have empty directory stubs
at the locations of submodules along with references to the submodule repo
tucked away in the superproject's `.git/` (`git` calls the project containing a
submodule the superproject).  To get the submodules, you can  

`$ git clone --recurse-submodules <git repo>`

`--recursive` is the old way to do this, for `git` versions 1.6.5-2.12

If you've already cloned, you can got into the repo root directory and do

`$ git submodule update --init`

This updates all submodules in the superproject, initializing them if they
haven't been yet.  To also do this for any nested submodules _in_ the
submodules, you can add `--recursive`

### Keep superproject updated!

Don't forget that if you make changes to a submodule or update it from the
remote, **you have to tell the superproject**.  You'll notice any differences
between the superproject's recorded commit and the submodule's current state
when you do `git status`.  To tell the superproject you want it to now point to
the new submodule state, do

```
$ git add /path/to/submod
$ git commit -m'Update my submodule'
$ git push
```

This can be a bit annoying, as you have to update, commit, and push the
superproject every time you do the same in the submodule (assuming you want what
I do, which is for the superproject upstream to always track the latest from all
the submodule upstreams).  But, I get it.  In most codebases and use-cases,
submodules suddenly changing every time their upstream tracking branch changes
would be maddening, as it could suddenly break what you're working on in the
superproject.

### Nice log/debug options

TODO: document the nice display customization mentioned in the docs.

### Hooks

You can use `git`
[hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to customize
all sorts of git behavior, and they're one way of customizing/configuring
submodule behavior.

### Caution on default behavior, and how to change it

The default behavior of submodules is very confusing and inconvenient to me,
though I understand why they do it.  It makes more sense for a very common
developer use-case of just adding required libraries that you aren't developing
and don't change as frequently as the source.

In this case, it's very important to have the specific library version your
project depends on, and when committing, merging, pushing, pulling, etc, you
don't typically expect to make any changes to the required libraries.  In git
terms, this means the submodule points to a _specific commit_.  This means when
you update the submodule, you don't pull the latest commit of the `master` branch,
but instead pull the specific commit recorded in the superproject.  This is the
(easily changed) default behavior/assumption for most `git submodule` commands.

My use-case, as mentioned, is different.  My superproject is usually a repo I'm
actively committing and pushing to _as are_ my submodules.  My submodules _are_
frequently changing and being actively developed by me.  Unlike with required
third-party libraries,  I _do_ want my
superproject to always update to the tip of `master` (or some other active
branch) for all submodules. 

Below, I go over some ways to achieve this desired behavior.

#### Superproject configuration

You can configure some commands' default behavior directly in the git repo.
This is my preference when possible, so that my config is under version control
and the behavior is shared with the repo.

To do this, you need to configure the `.gitmodules` file which is created in any
repo with submodules.  It contains entries of the form

```
[submodule "Name"]
    path = <relative path to submodule root directory>
    url = <submodule git repo's url>
    option1 = value
    option2 = value
    ...
```

`git`'s excellent docs cover [the options](https://git-scm.com/docs/gitmodules)
in detail.  Most of them can be configured/specified at the global level, repo
level, and commandline level.  I tend to prefer configuration in the repo's
`.gitmodules` file.

To get the behavior I want (keep submodules up-to-date with the tip of a
tracking branch, and don't bug me about nonsense)  I use these options:

[submodule "Name"]
    path = <relative path to submodule root directory>
    url = <submodule git repo's url>
    ignore = untracked
    branch = development
    update = rebase
```

+ `ignore`

This controls how much is ignored when `git` wants to report the status of
submodule changes. `untracked` tells it to leave me alone about all the
untracked files in the submodule.  I don't care.  Another attractive option is
`dirty`, which won't bother you about either untracked files or a dirty working
tree, instead only telling you when changes committed in the submodule differ
from what the superproject is tracking.

+ `branch`

This is the remote branch to be tracked by the submodule.  By default, it's
`master`.  But if you have a branch you tend to work in (e.g. `development`) or
want to make sure a dependency has been tested (e.g. `stable`), you can specify
a different branch.  

In sufficiently recent versions of `git`, you can use the
special value `.`.  This will use the remote branch that matches the submodule's
current branch (if I'm reading the docs right. they use the rather confusing
description of "A special value of . is used to indicate that the name of the
branch in the submodule should be the same name as the current branch in the
current repository.").

My reading of the docs suggest that if you want `master`, it's the default so
there's no need to specify it.  But my experience and StackOverflow implies you
should specify it even if you want `master`.  It seems to trigger the desired
behavior of staying updated with the latest commit in `origin`'s `master`.

+ `update`

My expectation was that if you configure the submodule `branch` option, and then do

`$ git submodule update --remote`,

then all submodules will be updated to the latest commit from the proper branch.
However, if you change into the submodule directory you'll find you're in a
detached HEAD state, pointing explicitly to the commit recorded by the
superproject.  To avoid a detached HEAD, you need to configure your submodule's
`update` option to be `merge` or `rebase`.  _Now_ a remote submodule update will
_sometimes_ get your submodule with HEAD attached to the tip of the desired branch, after
either merging or rebasing the remote's changes into the local submodule repo.

Even still, I often find myself with a detached HEAD, and this is scary.
Sometimes this HEAD will be pointing to a commit ahead of, say, the `master`
tip.  I'm still working on understanding submodule well enough to know when to
expect a detached HEAD and how to avoid it.

#### Track upstream
[This](https://stackoverflow.com/questions/9189575/git-submodule-tracking-latest)
is good on how to get a submodule to track upstream (keep the commit recorded in
the superproject as master:HEAD).  If setup properly, you should only need to do

`git submodule update --remote`

to update all submodule commits recorded in the superproject to the submodules'
respective HEADs.

To make sure a submodule is properly tracking upstream/origin, do something like

```
cd mysubmod
git branch -u origin/master master
```

See also your `.gitmodules`.  I believe in newest gits, you can set the tracked
branch to `.`, which should pull the latest changes from the submodule's
_current_ branch when you do `git submodule update --remote`.
