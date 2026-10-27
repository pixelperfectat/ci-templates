# pixelperfectat/ci-templates

Reusable CI for PixelPerfect Magento modules — maintain the pipeline in one place; every module
references it. The real checks live in each module's `composer` scripts; this only carries the
runner + stage wiring.

## GitLab (private modules)
Module `.gitlab-ci.yml`:
```yaml
include:
  - project: pixelperfect-at-m2-modules/ci-templates
    ref: main
    file: /gitlab/magento-module.yml
```
Stages: `validate → lint → analyse → test` (module `composer` scripts on `markoshust/magento-php:8.3-fpm`)
plus `create-release` — on a `X.Y.Z` tag push it publishes a GitLab Release from the matching
`CHANGELOG.md` section. Requires `MAGENTO_PUBLIC_KEY` / `MAGENTO_PRIVATE_KEY` CI variables; private
inter-module deps resolve via `CI_JOB_TOKEN` (allowlist this group under each depended-on repo).

Pinned to `ref: main` for auto-propagation — a pipeline change here reaches every module on its next
run. Pin a module to a tag instead if you want controlled, Renovate-bumped updates.

## Commit-message lint

A `commitlint` job in the `validate` stage (GitLab) / a `commitlint` job (GitHub) rejects commits with a **wrong subject format** — subject must follow `[optional-issue] <Keyword> <text>` where Keyword is one of: `Add Implement Introduce Update Refactor Improve Change Bump Deprecate Remove Delete Drop Fix Correct Resolve Security`.

`Merge branch …` / `Merge remote-tracking …` commits, and on GitHub the `Merge pull request #N from …` commits the merge button writes, are exempt from this rule.

The job uses `alpine:latest` with `git` (GitLab) / `ubuntu-latest` (GitHub); no Composer install. On an MR/PR it checks every commit since the diff-base; on a branch push it uses `CI_COMMIT_BEFORE_SHA` / `github.event.before` and falls back to the tip commit for new branches.

## GitHub (OSS modules)
Module `.github/workflows/ci.yml`:
```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: pixelperfectat/ci-templates/.github/workflows/magento-module.yml@<tag>
    secrets: inherit
```
Two jobs: `commitlint`, and `check` with the same gates as GitLab: PHP 8.3 with Xdebug for coverage,
the coding-standard checks (`check-module-boundaries`, `check-xml-class-refs`,
`check-framework-internals`), then the module's `composer check`, which includes Infection.

The module needs `pixelperfectat/coding-standard` `^0.2` in `require-dev` (from Packagist) and a
`check` script that runs Infection. Set `MAGENTO_PUBLIC_KEY`/`MAGENTO_PRIVATE_KEY` as Actions secrets
(+ `GITLAB_TOKEN` only for a private GitLab dependency).

The workflow is used from the GitHub copy of this repo, which GitLab push-mirrors: `main` and tags
only. Work and tag on GitLab; never push to GitHub directly.
