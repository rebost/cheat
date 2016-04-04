Git
===

Import one repository (with history) into another:
http://stackoverflow.com/questions/1683531/how-to-import-existing-git-repository-into-another

Delete untracked files (be careful!)::

    git clean -fdx

Prune branches that have been merged and are no longer upstream::

    http://devblog.springest.com/a-script-to-remove-old-git-branches

Prune branches that track remote branches that no longer exist
(http://kparal.wordpress.com/2011/04/15/git-tip-of-the-day-pruning-stale-remote-tracking-branches/)::

    $ git remote prune origin --dry-run
    $ git remote prune origin

Easier access to pull requests on Github.  Add to config::

    # This will make pull requests visible in your local repo
    # with branch names like 'origin/pr/NNN'
    # WARNING: This also breaks adding a new remote called "origin" manually
    # because git thinks there already is one.  Comment this out temporarily
    # in that case, unless you can think of a better solution.
    [remote "pulls"]
        fetch = +refs/pull/*/head:refs/remotes/origin/pr/*

Handy aliases for config::

    [alias]
    lg = log --oneline --graph --date-order
    lgd = log --oneline --graph --date-order --format=format:\"%ai %d %s\"

    cb = checkout -b
    cd = checkout develop
    co = checkout

    gd = !git fetch origin && git checkout develop && git pull origin develop
    gm = !git fetch origin && git checkout master && git pull origin master

    # push -u the current branch
    pu = "!CURRENT=$(git symbolic-ref --short HEAD) && git push -u origin $CURRENT"

    # push -f
    pf = push -f

    # Find the common ancestor of HEAD and develop and show a diff
    # from that to HEAD
    dd = "!git diff $(git merge-base develop HEAD)"
    # Find the common ancestor of HEAD and master and show a diff
    # from that to HEAD
    dm = "!git diff $(git merge-base master HEAD)"

    # These need 'hub' installed.
    # Create pull request against develop.  Must pass issue number.
    #pr = pull-request -b develop -i
    # Create pull request against develop, not passing issue number:
    pr = pull-request -b develop

    # Checkout pull request
    # Assume origin/pr/NN is pull request NN
    # Need a bash function because we need to concatenate something to $1
    #cpr = "!f() {set -x;git checkout origin/pr/$1; };f"
    cpr = "!gitcpr"

    # Undo any uncommited changes
    abort = checkout -- .

Submodules
----------

This will typically fix things::

    git submodule update --init --recursive


(and yes, you need --init every time)

Add a new submodule [http://git-scm.com/book/en/Git-Tools-Submodules]
::

    $ git submodule add git@github.com:mozilla/basket-client basket-client

Combining feature branches
--------------------------

Suppose you have branch A and branch B, which branched off of master
at various times, and you want to create a branch C that contains
the changes from both A & B.

According to Calvin: checkout the first branch, then git checkout -b BRANDNEWBRANCH. then rebase it on the second.


(SEE DIAGRAMS BELOW)

Example::

    # Start from master
    git checkout master
    git pull [--rebase]

    # Create the new branch from tip
    git checkout -b C

    # rebase A on master
    git checkout A
    git rebase -i master
    # merge A into C
    git checkout C
    git merge A

    # rebase B
    git checkout B
    git rebase -i master
    # merge B into C
    git checkout C
    git merge B

    # I think???
    # Review before using, and verify the result

Combining git branches diagrams

Start::

    o - o - o - o <--- master
     \   \
      \   o - o - o  <--- A
       o - o - o <--- B

Rebase A on master::

                     master
                     /
    o - o - o - o - o - o - o <--- A
     \
      o - o - o <--- B

Create new branch N from master::

                    master
                     /
    o - o - o - o - o - o - o <--- A
     \               \
      \               N
       \
        o - o - o <--- B

Switch to N and merge A::

                    master
                     /
    o - o - o - o - o - o - o <--- A
     \               \
      \               o - o - o  <--- N  (includes A)
       \
        o - o - o <--- B

Rebase B on master::

                    master
                     /
    o - o - o - o - o - o - o - o <--- A
                    |\
                    |  o - o - o <--- N (includes A)
                    \
                      o - o - o  <--- B

On N, merge B::

                    master
                    /
    o - o - o - o - o - o - o - o <--- A
                    |\
                    | o - o - o -  o - o - o <--- N (includes A and B)
                    \
                     o - o - o  <--- B

Delete A and B if desired.