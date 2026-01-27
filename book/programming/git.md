# Cookbook for Git

## View global config

`git config --list --show-origin`

## Verified Commit

Detailed documents are [here for Github](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification) and [for Gitlab](https://docs.gitlab.com/ee/user/project/repository/signed_commits/). But below is a list of commands to go through.

1. enable the verified commits: `git config --global commit.gpgsign true`
2. Generate GPG keys `gpg --full-generate-key`
	1. find the keyid:

```bash
gpg --list-secret-keys --keyid-format=long
```

	1. print the public key: `gpg --armor --export KEYID`
	2. copy it to your github account settings
3. Make sure it is used
```bash
git config --global user.signingkey KEYID
```
	1. or for the current git repo.
5. Now commit with `git commit -S -m "my cool message"`

## Git remote

```bash
git remote -v
```

`upstream` can be changed to whatever name you want.

```bash
git remote add upstream git@github.com:acts-project/acts.git
git fetch upstream
git checkout -b up_main upstream/main
```
## Tag a version

```bash
git tag -a <tag-name> -m <message>
```

Checkout a tag:
```bash
git checkout tags/v1.1.0 -b v1.1.0
```

## Single branch
add additional branches to the repo.
```bash
git clone xxx.git --single-branch
git remote set-branches --add origin gnn_aas
git fetch origin
git checkout -b gnn_aas origin/gnn_aas
```

## WorkTree
If you want to work on multiple branches at the same time, you can use `git worktree` to create a new working tree for a branch.

In case of increating a worktree for working on an existing branch:
```bash
git worktree add -b <new-branch-name> <path-to-new-worktree> <existing-branch-name>
```

The following showing an example of how to use `worktree` to develop Athena.

1. One-time setup (base repo + remotes):
    ```bash
    # Base repo (shared .git object store)
    git clone ssh://git@gitlab.cern.ch:7999/atlas/athena.git /global/cfs/cdirs/m3443/usr/xju/athena_dev/athena_base
    cd /global/cfs/cdirs/m3443/usr/xju/athena_dev/athena_base

    # Add your fork
    git remote add xju ssh://git@gitlab.cern.ch:7999/xju/athena.git
    git fetch --all --prune

    # (Optional) defaults
    git config remote.pushDefault xju
    git config fetch.prune true
    ```
2. keep a production worktree for upstream main branch:
    ```bash
    # Production worktree
    git worktree add /global/cfs/cdirs/m3443/usr/xju/athena_dev/athena_main upstream-main

    # create a local branch that tracks origin/main
    git -C /global/cfs/cdirs/m3443/usr/xju/athena_dev/athena_main switch -c upstream-main --track origin/main
    ```
3. create a new feature branch from upstream and push to your fork:
    ```bash
    # Always sync first in the base repo
    cd /global/cfs/cdirs/m3443/usr/xju/athena_dev/athena_base
    git fetch --all --prune

    # Create a worktree with a new branch off upstream main
    git worktree add -b new-feature /pscratch/sd/x/xju/athena_dev/20250808_removeTritonInterface origin/main
    cd /pscratch/sd/x/xju/athena_dev/20250808_removeTritonInterface
    git push -u xju HEAD
    ```
4. Regular sync: rebase your branch on latest upstream
    ```bash
    git fetch origin
    git rebase origin/main

    # resolve conflicts, then push
    git push --force-with-lease
    ```
5. Clean up
    ```bash
    git worktree list
    git worktree remove PATH
    git worktree prune
    ```

## FOA

### Invalid file
```bash
git hash-object -w file-path
```

>[!info] Information
>Hello


[name of the link](https://google.com)
