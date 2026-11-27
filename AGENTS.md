# Org Conformance

Single implementation of the BehindTheMusicTree conformance rules, run in every repository by an organization ruleset. Follow [CONTRIBUTING.md](CONTRIBUTING.md). Every change reaches all repositories once the ruleset pins its release.

Run `actionlint` for workflow changes, `shellcheck ./*.sh tests/*.sh` and the scripts in `tests/`. This repository is public: exemption reasons must not disclose anything private about a repository. release-please owns `VERSION` and `CHANGELOG.md`; do not edit them by hand.
