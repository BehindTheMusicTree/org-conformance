# org-conformance

Organization-wide conformance checks of [BehindTheMusicTree](https://github.com/BehindTheMusicTree), run by an organization ruleset (`workflows` rule). Repositories add no caller workflow.

## Checks

[`.github/workflows/org-conformance.yml`](.github/workflows/org-conformance.yml) runs on pull requests and merge groups. Its single job, `org-conformance`, reports three findings (all run even when one fails). Scripts, rules and exemptions are read at the workflow's own commit (`job.workflow_sha`), so the release pinned by the ruleset selects them.

### Agent instructions

`AGENTS.md`/`CLAUDE.md` adapters, checked by the agent-config engine pinned in [`rules.json`](rules.json) (`agent_config`).

### Workflow concurrency

[`check-concurrency.sh`](check-concurrency.sh). Every workflow triggered by `push` or `pull_request` declares workflow-level `concurrency` in one of three forms, where `<stem>` is the workflow file name without `.yml`/`.yaml`:

| Form | `group` | `cancel-in-progress` | Applies to |
| --- | --- | --- | --- |
| CI | `<stem>-${{ github.event_name == 'pull_request' && format('pr-{0}', github.event.pull_request.number) \|\| github.ref }}` | `${{ github.event_name == 'pull_request' }}` | default |
| Event | `<stem>-${{ github.sha }}` | `false` | workflows in which any job calls a `release_workflows` entry of `rules.json` (the other jobs share its per-commit group); `event` exemptions |
| Queue | `<stem>-${{ github.ref }}` | `false` | `queue` exemptions |

The CI group uses the PR number for every pull request event, including `closed`, and the full ref for pushes.

Deploy and publication workflows without a `pull_request` trigger use the CI form: for push runs it resolves to `<stem>-${{ github.ref }}` without cancellation, so runs of a ref are serialized and never cancelled.

A workflow triggered only by `workflow_call` must not declare workflow-level concurrency: its callers do.

### Shared issue forms

[`check-issue-forms.sh`](check-issue-forms.sh). GitHub replaces the whole organization default folder when a repository has a valid local template or `config.yml` in `.github/ISSUE_TEMPLATE`, so any repository other than `.github` fails on a file there, in `ISSUE_TEMPLATE/` or `docs/ISSUE_TEMPLATE/`, or on `ISSUE_TEMPLATE.md` (root, `docs/` or `.github/`). The `.github` repository must instead keep `bug-report.yml` (type `Bug`, label `needs-triage`), `feature-request.yml` (`Feature`, `needs-triage`), `tracking-issue.yml` (`Task`, `tracking`), `follow-up-action.yml` (`Task`, `follow-up`) as valid single-document YAML forms, and a `config.yml` with `blank_issues_enabled: true`.

## Exemptions

Exemptions from the CI concurrency form are central, in [`exemptions.json`](exemptions.json), keyed by repository name. A repository cannot exempt itself: add or remove an entry with a pull request here.

```json
{
  "concurrency": {
    "exemptions": {
      "some-repository": [
        { "workflow": "ruleset-check.yml", "form": "queue", "reason": "Pull request runs must not be cancelled." }
      ]
    }
  }
}
```

Unknown keys, a missing `reason` or `form`, duplicates and stale entries (missing workflow, no `push`/`pull_request` trigger, already a release caller) are violations. A stale entry is reported on the repository's own check; `--org` also reports entries of unknown or archived repositories.

## Command line

```bash
./check-concurrency.sh --local <checkout> --repo <name>   # one checkout
./check-concurrency.sh --org                              # default branch of every non-archived repository
./check-concurrency.sh --config-only                      # validate rules.json and exemptions.json
./check-issue-forms.sh --local <checkout> --repo <name>
```

`RULES_FILE` and `EXEMPTIONS_FILE` default to the files next to the script.
