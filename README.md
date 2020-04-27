TEST: DO NOT MERGE !!!

This is a demo showing how to work around GitHub Actions limitation of CI jobs not being able to post comments to the upstream repo's pull requests. The same approach could be used for other actions that require upstream security access.  Use with care to avoid compromising your primary repo's security token.

## Problem
Common scenario in the FOSS community:
* You want community to fork your repo and submit pull requests
* You want your CI job to post run results as a comment to the pull request
* GitHub does not allow GitHub Actions running from a forked repo to modify parent repo, even to post comments on its own PR.

## Workaround
* [Pull request action](https://github.com/nyurik/auto_pr_comments_from_forks/blob/master/.github/workflows/test.yml#L1)  creates an `.md` file with Github markdown comment content, and saves it as an artifact under some name.  This action runs in the context of the forked repo, so it has no way to post a PR comment.
* A regular [cron job](https://github.com/nyurik/auto_pr_comments_from_forks/blob/master/.github/workflows/pr_updater.yml#L1) looks at all the open pull requests and recently completed action runs, looks for the posted artifacts, and copies their content as comments to the corresponding pull requests, updating existing comment on repeated runs.

## Implementation and Testing

The [code](https://github.com/nyurik/auto_pr_comments_from_forks/blob/master/.github/workflows/pr_updater.yml#L1) was written using a bash script with `curl` and `jq`, without any other libraries. It is already running on this repo, and has created [this PR message](https://github.com/nyurik/auto_pr_comments_from_forks/pull/1). Feel free to test it further by forking this repository, creating a dummy change, and creating a pull request. The cron job performs these steps on each run:

* get all open pull requests
* get all recent workflow runs
* match pull requests and their current SHA with the last workflow run for the same SHA
* for each found match of  `<pull-request-number>`  and  `<workflow-run-id>` :
  * download artifact from the workflow run -- expects a single file with markdown content
  * look through existing PR comments to see if we have posted a comment before
    (uses a hidden magical header to identify our comment)
  * either create or update the comment with the new text (if changed)

## Volunteers Needed
It would be great if the same algorithm could be implemented as a proper packaged action, rather than a `curl+jq` hack :)
