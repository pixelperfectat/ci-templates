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

## GitHub (OSS modules)
Module `.github/workflows/ci.yml`:
```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: pixelperfect-at-m2-modules/ci-templates/.github/workflows/magento-module.yml@main
    secrets: inherit
```
Works once `ci-templates` is mirrored to GitHub, with `MAGENTO_PUBLIC_KEY`/`MAGENTO_PRIVATE_KEY`
(+ `GITLAB_TOKEN` for private GitLab deps) as Actions secrets.
