git-subrepo Design
==================

How the subrepo commands should work.

Glossary:
* `subrepo` - An external repository integrated as a repo subdirectory
* `subdir` - The directory in the work tree where the subrepo lives
* `upstream` - The remote repo that the subrepo is tracking
* `local` - The local parts of the subrepo
* `.gitrepo` - The subrepo state file
* `remote` - The subrepo url
* `branch` - The remote branch the subrepo is tracking
* `commit` -  The upstream commit id that we last synced to
* `former` - The local commit that we last synced to


## clone

This is the action that adds a new subrepo. It feels very much like a clone, so
we call it that.

Usages:

    # Add a subrepo into a subdir (track remote HEAD branch):
    git subrepo clone <remote> <subdir>
    # Guess subdir from remote url:
    git subrepo clone <remote>
    # Use a named branch instead of the remote HEAD:
    git subrepo clone <remote> [<subdir>] -b <branch>

Steps:
* Assert clean, and chdir to root (adjusting subdir)
* Determine the remote HEAD
  * Else error, branch (-b) needed
* Fetch the remote branch commits
* Subtree merge the remote subrepo history into the subdir
* Squash the history to a single commit
* Merge the subtree commit into the mainline HEAD
* Add a .gitrepo file to the subdir
* Amend the .gitrepo into the HEAD merge commit
* Create a branch .git/refs/subrepo/remote/<subdir>
* Add a remote called subrepo/<subdir>

## pull

Fetch and merge the remote content. This could be a single operation, or may
require a manual process: (see checkout)

Usages:

    # Fetch and rebase subdir/subrepo:
    git subrepo pull --rebase subdir
    # Strategies: --rebase --ours --theirs --merge (recursive):
    git subrepo pull --<merge-strategy> subdir
    # Pull all the subrepos:
    git subrepo pull --<strategy> --all
    # Change the tracking branch and pull it:
    git subrepo pull --<strategy> -b <branch> <subdir>
    # Pull in the hand merged branch:
    # git subrepo pull <subdir>

Steps:
* Assert clean, and chdir to root (adjusting subdir)
* Fetch the remote branch content
* Update remotes and the refs in .git/
  * The might not be the same repo that did the clone
* If not hand merged
  * Create a branch called subrepo/<subdir>
  * Checkout the new branch
  * Apply the merge strategy to the remote branch
  * If not clean
    * Error message
    * Reset to starting state
* Subtree merge the subrepo/<subdir> branch into repo
    git subrepo clone <remote> <subdir>
* Squash the upstream history
* Merge the subtree into mainline
* Update the .gitrepo file
* Amend .gitrepo into HEAD
* Delete the subrepo/<subdir> branch

## push

Push a merged subrepo history to the remote. This could be a single operation,
or may require a manual process.

Usages:

    # Fetch, apply strategy, push to remote:
    git subrepo push <subdir>
    # Do all subrepos:
    git subrepo push --all

Steps:
* Assert clean, and chdir to root (adjusting subdir)
* Update remotes and the refs in .git/
* Fetch the remote branch content
* Make sure we have pulled the latest remote changes
* Checkout subrepo/<subdir> branch
* Rebase the local change s on top
* Push it to subrepo/<subdir> remote

## checkout

Create a branch of the local changes to the subrepo and checkout it. Also fetch
the remote tracking branch, so it can can be used for merges. This command is
used to do a pull/merge by hand. After you merge, use the `pull` command to
finish up.

Usages:

    # Checkout a subdir/subrepo
    git checkout <subdir>

Steps:
* Assert clean, and chdir to root (adjusting subdir)
* Assert repo is on a branch
* Update remotes and the refs in .git/
* Fetch the remote branch content
* Create a branch of local subrepo changes subrepo/local/<subdir>
* Checkout the new branch

## status

Give a full status of specific subrepos or all of them.

Usages:

    # Get status for all subrepos:
    git subrepo status
    # Get status for specific subrepos:
    git subrepo status <subdir> [<subdir> ...]
