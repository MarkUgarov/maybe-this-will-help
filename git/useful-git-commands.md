# Useful git-commands

## Basics

### Branches
There are two sorts of branches: `local` and `remote`.

Thanks to [this comment on Stackoverflow](https://stackoverflow.com/a/24785777)

#### Remote branches
Remote branches are branches that are stored on a `remote`.

A `remote` is basically everything outside your project-folder, 
but in 99% of the time you will refer to the `origin` on the server that hosts your repository when using this term. 

To keep it simple: When you say "I have to pull from remote" you just want to say that you are copying something from
the server onto your local computer via a simple git command. 

```bash 
git remote -v
```
will show you all remotes that are registered for your project, e.g.
```txt
origin  https://github.com/MarkUgarov/maybe-this-will-help.git (fetch)
origin  https://github.com/MarkUgarov/maybe-this-will-help.git (push)
```


#### Local branches
Local are on your machine.

There are three types of them, even though you will rarely distinguish between them.  
It is more likely you will call it local branches all together and speak about how to have to "commit", "push" and 
"pull" (sometimes "fetch") them, which basically means to bring them up to date with the remote branches. 

##### Non-tracking local branches 

Local branches, that are only on your machine, are called Non-tracking local branches. 
You create one by running 
```bash
git branch <branchname>
```
or check out from your remote machine by running
```bash
git checkout <branch-name> 
```

##### Remote tracking branches
Despite the name, these branches are still local. 

Each remote-tracking branch is stored in a file `.git/refs/remotes/<remote>/`.  
Side note: The `.git`-Folder is located in the root of your git-Project. 

You can run the following command to list all the remote-tracking branches on your machine:
```bash
git branch -r
```
This will give you an output that looks something like this: 
```txt
origin/HEAD -> origin/master
origin/gh-pages
origin/master
```

You won't do any programming on those remote tracking branches, rarely even speak about them.  
You can see them as a local copy of the "real" remote branches. 

##### Tracking local branches
Tracking local branches are associated with another branch, usually a remote-tracking branch.
If you check out a remote branch, this association is usually made for you.  
You can create one on your machine by 
```bash 
git branch --track <branchname> 
```
and see them by running
```bash
git branch -vv
```
The output shows all local branches together with their most recent commit.  
Your current branch will be highlighted by `*` (asterisk).

```txt
  gh-pages 9c6d1dfa [origin/gh-pages] Merge remote-tracking branch 'origin/master' into gh-pages
* master   058e978 [origin/master] Create README.md
```

### Synchronise branches

You can get changes 
* from your branches on `remote` to your local machine
* from your local machine to `remote` 

#### Get from remote to local

##### Fetch

You can update your remote tracking branches by running
```bash
git fetch
```
However, this will not affect your tracking local branches.

If you have multiple remote repositories (which will rarely happen in real life), you can fetch all by
You can update your remote tracking branches by running
```bash
git fetch --all 
```
##### Pull

`pull` uses `fetch` in the background. It downloads all commits of your remote and apply the local tracking branch. 

Simply call 
```bash
git pull
```
to update your current local branch.

#### Merge

TODO

#### Get from local to remote

TODO

## Branch information

### Show associations

Before you start to actually push, pull and commit changes with git, you should have an overview
of the associations between remote and local branches.

You can do that by calling

```bash
git remote show <remote>
```
e.g.
```bash
git remote show origin
```

```txt
* remote origin
  Fetch URL: https://github.com/MarkUgarov/maybe-this-will-help.git
  Push  URL: https://github.com/MarkUgarov/maybe-this-will-help.git
  HEAD branch: master
  Remote branches:
    gh-pages tracked
    master   tracked
  Local branches configured for 'git pull':
    gh-pages merges with remote gh-pages
    master   merges with remote master
  Local refs configured for 'git push':
    gh-pages pushes to gh-pages (up to date)
    master   pushes to master   (up to date)

```

### Information about your local branches

#### Current local branches 
```bash
git branch 
```
shows the current local branches. 
The result may be something like  `master`, `main` or `develop`.
Your current branch will be marked with an `*` (asterisk).

#### Dates of the last commits
```bash
git for-each-ref --sort=-committerdate refs/heads --format='%(committerdate:short) %(refname:short)'
```
shows you the date of the last commits in your local branches. 
This especially helps to find old branches that may be obsolete or to check if your local branches are up-to-date with 
the remotes. 

The output may be something like this: 
```txt
2022-09-08 master
2021-10-15 gh-pages
```
### Information about remote branches

#### All branches

```git branch --all```
shows you all branches, local and remote. 
The current branch will be marked with an  `*` (asterix).

The output could look somewhat like that: 
```txt 
  gh-pages
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/gh-pages
  remotes/origin/master
```

#### Dates of the last commits 
```bash
git for-each-ref --sort=-committerdate refs/heads refs/remotes/ --format='%(committerdate:short) %(refname:short)' 
```
shows you the date of the last commits in your remote branches.
This especially helps to find old branches that may be obsolete or to check if your local branches are up-to-date with
the remotes, but can definitely overwhelm when dealing with a lot of branches. 

The output can look something like the following:

```txt
2022-09-08 master
2022-09-08 origin
2022-09-08 origin/master
2021-10-15 gh-pages
2021-10-15 origin/gh-pages
```

## Graphical representation

```bash
gr = log --decorate --oneline --graph
```
shows you a graphic representation of the commits of your current (local) branch.  
It will also represent different branches which where merged into yours via the Symbols `|` (pipe) and `*` (asterisk)

E.g. the following output represents how the following: 
1. Commit `eaccff5` was first committed to the branch `master`. 
2. Then `master` was merged into `gh-pages` and therefore the Commit now exists in both branches. 
```txt
*   9c6d1dfa (origin/gh-pages, gh-pages) Merge remote-tracking branch 'origin/master' into gh-pages
|\
| * eaccff5 test init
```

You can also choose to see *all* commits of *all* branches, but this might get messy.
```bash
git log --all --decorate --oneline --graph
```

**Tipp**: Some terminals will only show lines until they run out of space and will then await an input (`:`).
You have to hit `enter` to see the next batch or `q` to quit. 



