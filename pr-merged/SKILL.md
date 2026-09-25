---
name: pr-merged
description: "Realign the local checkout after a PR merge: return to the base branch and delete the orphan local branch."
disable-model-invocation: true
---

The PR has been merged and its remote branch has been deleted. Realign the local environment: remove
the orphan local branch left behind and return to the branch the PR was merged into.

1. Identify the orphan branch: the PR's head branch, normally the current branch. Run `git status`;
   if the working tree has uncommitted changes, stop and report them.
2. Confirm the merge and find the base branch with
   `gh pr view <orphan> --json state,baseRefName`. Continue only when `state` is `MERGED`; otherwise
   stop and report what you found.
3. Switch to the base branch and update it: `git switch <base>`, then `git pull --ff-only`.
4. Drop the stale remote-tracking ref: `git fetch --prune`.
5. Delete the orphan branch with `git branch -d <orphan>`. A squash or rebase merge makes `-d` refuse,
   because the branch's commits are not in the base history; the `MERGED` state from step 2 is the
   confirmation to use `git branch -D <orphan>` instead.

Done when the current branch is the base branch at the same commit as `origin/<base>`, and the orphan
branch appears in neither `git branch` nor `git branch -r`. Report the base branch, its HEAD commit
and the deleted branch.
