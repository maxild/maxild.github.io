+++
title = "Advanced Git"
date = 2020-01-01

#[taxonomies]
#categories = ["git"]
#tags = ["tag1"]
+++

## Best practice recap (TL;DR)

### Adding or cloning

* Initial add: `git submodule add <url> <path>`
* Initial container clone: `git clone --recursive <url> [<path>]`

### Grabbing updates inside a submodule

* `cd path/to/module`
* `git fetch`
* `git checkout -q <commit-sha1>`
* `cd -`
* `git commit -am "Updated submodule X to: blah blah"`

### Grabbing container updates

* `git pull`
* `git submodule sync --recursive`
* `git submodule update --init --recursive`

### Updating a submodule inside container code

* `git submodule update --remote --rebase -- path/to/module`
* `cd path/to/module`
* Local work, testing, eventually staging
* `git commit -am "Update to central submodule: blah blah"`
* `git push`
* `cd -`
* `git commit -am "Updated submodule X to: blah blah"`

### Permanently removing a submodule

* `git submodule deinit path/to/module`
* `git rm path/to/module`
* `git commit -am "Removed submodule X"`

<!-- more -->

## Submodules

Git submodules are implemented using two moving parts:

* the .gitmodules file and
* a special kind of tree object (16000, the socalled 'gitlink' object).

The container repo will get a new `.gitmodules` file containing

```bash
$ cat .\.gitmodules
[submodule "themes/aria"]
  path = themes/aria
  url = https://github.com/maxild/hexo-theme-aria.git
```

The actual sha1 of the child repo that is 'added/cloned in' is stored in the
object database of the container repo, as can be seen by doing a

```bash
$ git ls-tree master themes/aria

# Git knows themes/aria is a submodule, because it is recorded in the index with a special mode "160000"
160000 commit df308920c1ecaf759a2b2b4b00b765e069f59eb4  themes/aria
```

and a `rev-parse` to find the current commit/sha1

```bash
$ cd themes/aria
$ git rev-parse master

df308920c1ecaf759a2b2b4b00b765e069f59eb4
```

to see that container is tracking the child repo sha1 in its own object
database (`.git` folder). So the container is tracking the exact reference
(sha1) to the child/submodule repo in a socalled gitlink object. You can
think of this 'gitlink' object like a binary file. Perhaps later you will
get merge conflicts when different developers each are trying to reference
different commits of the child repo.

If you are the maintainer of both the container repo and the sub/child repo,
then maybe you are going to update the child repo inside the container.
If pushing new commits to the master branch og the child repo, then the
container repo is tracking the old commit/sha1 in its object database.
This can be seen by doing a `status` in the container. GIT will show that
the two commit id's are out of sync, by saying

```bash
modified: themes/aria (new commits)
```

This is a problem for other developers working on the container project,
because they will not pick up the changes to the 3rd party child repo.

To update the container dependency we need to do an `add` on the submodule

```bash
git add themes/aria
```

When you now commit the message should say "Updating reference/sha1 of
themes/aria submodule".

If we at any time checkout a different commit in the child/submodule,
we need to `git add <submodule>; git commit` the change in the container repo.
This could be either going forward og backward in the child repo history.

BUT we have a problem: None of the work in the container (the reference) and
the new commits in the child repo have been pushed to Github.  We need to
push the changes in the child repo first. First after sharing the child
commit/sha1, can we push the changed reference in the container repo.

> Always push submodule changes first

Feature branching in submodules

> You can work on feature branches of the child repo _inside_ the container
repo (even if the feature branches a pushed to Github) without affecting
other developers using the container repo.

If the feature branch dependency should be propagated to other developers
using the container repo, you can of course `add` the submodule, and commit
changes.

> Submodule dependency is always a commit/sha1, never a branch.

## Cloning a repo with submodules

`clone` will not clone any submodules by default. You will have to use

```bash
git submodule init
```

to update your containers `.git./config` with

```ini
[submodule "themes/aria"]
  url = https://github.com/maxild/hexo-theme-aria.git
  active = true
```

You will have to use `submodule update` to checkout files. In practice,
when dealing with submodule-using repos, we usually group the two commands
(init and update) in one:

```bash
git clone
git submodule update --init
git submodule update --init --recursive
```

A shortcut is using `--recursive` on `clone`

```bash
git clone --recursive
```

Note that we are now on a detached head inside the submodule

```bash
git status

HEAD detached at fe64799
```

## Updating a repo with submodules

If we want to pull down upstream changes to the child project we `cd` into
the child/submodule folder, and

```bash
git fetch
git log --one-line -10
```

find the commit `<sha1>` we want to depend on, and

```bash
git checkout -q <sha1>
```

(The -q is only there to spare us Git blabbering about how we’re ending up
on a detached head. Usually this would be a healthy reminder, but on this one
we know what we’re doing.)

After that we `cd` back to the container, and the following commands will
tell us that the submodule is out of sync with the 'gitlink' object in the
database

```bash
git status
git submodule status
git diff --submodule=log
```

(try all of the 3 commands above)

In `git status` output, we see a `_new commits_` change type, which means the
referenced commit changed. To update our submodule reference

```bash
git add .
git commit -m "Updated reference to theme/aria submodule"
```

## Pulling a repo with submodules

If pulling down upstream changes, all submodules will also be fetched.

```bash
git pull
```

This behavior became the default with Git 1.7.5, with the configuration
setting `fetch.recurseSubmodules` now defaulting to `on-demand`: if a
container project gets updates to referenced submodule commits, these
submodules get fetched automatically.

> Git auto-fetches, but does not auto-update.

This is the massive danger: if you do not explicitly update the submodule's
working directory, your next container commit will regress the submodule.
Therefore always:

```bash
git submodule update
```

As long as we are trying to form generic good habits, the preferred command
here would be a git `submodule update --init --recursive`, in order to
auto-init any new submodule, and to recursively update these if need be.

## Submodule update...better headline here?

The command `submodule update` is doing a checkout in the child repo under
the hood.

```bash
git submodule update
git submodule update --recursive
```

When you run `submodule update`, it checks out the specific version of the
child project, but not within a branch. This is called having a detached
head — it means the HEAD file points directly to a commit, not to a symbolic
reference. Updating submodules will therefore checkout referenced commit
(not necessarily the latest commit of the child repo). You can see what commit
is referenced in the container project by running (within the "super project"):

```bash
git submodule status
git submodule status themes/aria
```

> Each time you execute a git command in the "super project" which could
modify that submodule commit SHA1, you need a `git submodule update`.

So you need a `submodule update` every time you pull down a submodule change
in the container/main project.

Git 1.8.2 features a new option `--remote` that will fetch the latest
changes from upstream in each submodule, rebase them in, and check out the
latest revision of the submodule.

```bash
git submodule update --rebase --remote
```

This is equivalent to running git pull in each submodule, which is generally
exactly what you want.

## Removing a submodule

This one is easy

```bash
git rm themes/aria
```

In addition to stripping the submodule from the working directory, the
command will update the `.gitmodules` file so it does not reference the
submodule anymore. What is odd though, is that the local config retains
submodule information. You can ensure local config cleanup

```bash
git submodule deinit themes/aria
git rm themes/aria
```

before commiting

```bash
git commit -m "Removed the themes/aria submodule"
[master 31cb27d] Removed the themes/aria submodule
 2 files changed, 4 deletions(-)
 delete mode 160000 themes/aria
```

Regardless of your approach, the submodule’s repo remains present in
`.git/modules/themes/aria`, but you are free to kill that whenever you want
(not that important).

## Things to watch out for

1. Updating submodules will check out the submodules at the commit they are
currently pointing to (not necessarily the latest commit).
1. Always create a branch when you work in a submodule directory with
`checkout -b`. When you do the `submodule update` a second time, it will still
revert your work, but at least you have a pointer to get back to.
1. Switching branches with submodules in them can also be tricky. If you
create a new branch, add a submodule there, and then switch back to a branch
without that submodule, you still have the submodule directory as an
untracked directory.
1. Always publish (push) the submodule change before publishing (push) the
change to the superproject that references it.
