---
name: pr-merged
description: "Realign the local checkout after a PR merge: return to the base branch, delete the orphan local branch, check the linked Linear issue's state."
disable-model-invocation: true
---

The PR has been merged and its remote branch has been deleted. Realign the local environment: remove
the orphan local branch left behind and return to the branch the PR was merged into.

1. Identify the orphan branch: the PR's head branch, normally the current branch. Run `git status`;
   if the working tree has uncommitted changes, stop and report them.
2. Confirm the merge and find the base branch with
   `gh pr view <orphan> --json state,baseRefName,title,body`. Continue only when `state` is
   `MERGED`; otherwise stop and report what you found.
3. Switch to the base branch and update it: `git switch <base>`, then `git pull --ff-only`.
4. Drop the stale remote-tracking ref: `git fetch --prune`.
5. Delete the orphan branch with `git branch -d <orphan>`. A squash or rebase merge makes `-d` refuse,
   because the branch's commits are not in the base history; the `MERGED` state from step 2 is the
   confirmation to use `git branch -D <orphan>` instead.
6. Check the linked Linear issue. Find its identifier in the orphan branch name
   (`luca/coine-113-...` → `COINE-113`), the PR title or the PR body; with no identifier, skip this
   step. Read the issue with the Linear MCP tools or the `linear` CLI and set the state the context
   calls for:
   - Usually **Done**, often already set by the merge itself.
   - A different state when the merge did not close the work, even if the merge already moved the
     issue to Done: the PR delivered only part of the issue (acceptance criteria left unchecked and
     outside this PR, further PRs announced), or a criterion needs a check that happens after the
     merge (a verification run on a real project). Derive the state from the issue description, its
     acceptance criteria, the PR body and the conversation.

   When Linear is unreachable, report the identifier and the state to check by hand.

Done when the current branch is the base branch at the same commit as `origin/<base>`, the orphan
branch appears in neither `git branch` nor `git branch -r`, and the linked Linear issue, if any, is in
the state its context calls for. Report the base branch, its HEAD commit, the deleted branch, and the
issue's final state with the reason when it is not Done.
