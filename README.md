# git-edit
Small script to make editing commits faster

## Installation
Download [`git-edit`](https://raw.githubusercontent.com/samiraguiar/git-edit/master/git-edit) to a directory that is in your `PATH` environment variable. This will allow you to invoke it as if it was a built-in Git command (e.g. `git edit -o`).

## Usage

```
usage: git-edit [-h] [-o] [-c] [<revision>]

Edit a commit in a Git repository

positional arguments:
  <revision>            the revision to edit

optional arguments:
  -h, --help            show this help message and exit
  -o, --commit-original
                        commit the current staged changes with the message of
                        the revision being edited
  -c, --continue        finish editing the commit
```

## Examples

Suppose we have the following history in a Git repository:

```
A -- B -- C -- D
               ↑
              HEAD
```

and we want to split commit `B` into two commits, `B1` and `B2`. We want `B1` to have a single change: introduce a file named `file1` that was previously added in `B`, and we want `B2` to contain the rest of the changes **reusing the message** from `B`. `git-edit` makes it simple:

```bash
$ git edit B
```

This is the equivalent of doing:
```bash
# start rebasing from the parent of B
$ git rebase -i B^
# (edit the rebase TODO list so that the B hash is marked with "edit")
# undo B so we can edit it
$ git reset --soft HEAD
```

We can now commit `B1`:

```bash
$ git commit -m 'Adding file 1' -- file1
```

And create `B2` with the same message from `B` and the rest of the staged changes:
```bash
$ git edit -o
```

This is the equivalent of doing:
```bash
$ git commit --reuse-message=B
```

(if we don't want to reuse the message, we can just manually commit the changes with another message, e.g. `git commit -m 'Some other message'`)

Finally we can finish editing:
```bash
# this just cleans up a file used for store purposes and end the rebase process
$ git edit -c
```

The history now looks like this:
```
A -- B1 -- B2 -- C -- D
                      ↑
                     HEAD
```

`B1` contains a part of the changes from `B`, while `B2` contains the rest. `B2` also has the same message as `B`.

## TODO
- [ ] Linter up with pylint/pycodestyle
- [ ] Document functions
- [ ] Fix --reword: need to use a real tty otherwise git complains
- [ ] Handle merge commits or commits whose parent is a merge commit
- [ ] Support editing multiple commits
- [x] Add an --abort switch
- [ ] Handle root commit
- [ ] Color git-status output (called just after edit has started)
- [ ] Print a short summary of the newly created commits (hashes and msg) or some message if they were all removed
