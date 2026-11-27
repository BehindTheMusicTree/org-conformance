# Contributing

## Branching (GitHub Flow)

Branch from `main` (`feat/`, `fix/`, `docs/`, `chore/`, `ci/`) and open a pull request against `main`. No direct commits to `main`. Pull requests are squash-merged, so the title must be a [conventional commit](https://www.conventionalcommits.org/) subject.

## Testing

Requires `jq`, [mikefarah `yq`](https://github.com/mikefarah/yq), `shellcheck` and `actionlint`.

```bash
actionlint
shellcheck ./*.sh tests/*.sh
./check-concurrency.sh --config-only
./tests/check-concurrency.sh
./tests/check-issue-forms.sh
```

A change to a rule comes with a fixture in `tests/` covering both the accepted and the rejected case.

## Constraints of the enforced workflow

`.github/workflows/org-conformance.yml` is run by an organization ruleset in other repositories:

- only `pull_request`, `pull_request_target` and `merge_group` triggers work, and GitHub ignores their filters;
- it must not use `cancel-in-progress`;
- the job name, `org-conformance`, is the check name;
- a checkout without parameters is the target repository, and scripts must be read at `job.workflow_sha`.

## Releasing

release-please maintains the release pull request, `VERSION` and `CHANGELOG.md`; merging it publishes the tag and the GitHub Release. A release takes effect in other repositories only when the ruleset, managed in the organization's configuration repository, pins it.
