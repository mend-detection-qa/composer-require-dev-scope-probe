# require-dev-scope

## Feature exercised

This probe exercises Composer's two-scope dependency model: packages declared in `require` must be detected with group `main`, and packages declared in `require-dev` (plus all their transitive dependencies) must be detected with group `dev`.

## Probe metadata

| Field | Value |
|---|---|
| pattern | `require-dev-scope` |
| pm | composer |
| schema_version | 1.0 |
| php_min | 8.1 |
| composer_min | 2.6 |
| target | remote |
| repo | https://github.com/mend-detection-qa/composer-require-dev-scope-probe |

## Package manifest

### `require` (group: main)

| Package | Version | Direct |
|---|---|---|
| `monolog/monolog` | 3.10.0 | yes |
| `psr/log` | 3.0.2 | no (transitive via monolog) |

### `require-dev` (group: dev)

| Package | Version | Direct |
|---|---|---|
| `phpunit/phpunit` | 10.5.63 | yes |
| `myclabs/deep-copy` | 1.13.4 | no |
| `nikic/php-parser` | v4.19.5 | no |
| `phar-io/manifest` | 2.0.4 | no |
| `phar-io/version` | 3.2.1 | no |
| `phpunit/php-code-coverage` | 10.1.16 | no |
| `phpunit/php-file-iterator` | 4.1.0 | no |
| `phpunit/php-invoker` | 4.0.0 | no |
| `phpunit/php-text-template` | 3.0.1 | no |
| `phpunit/php-timer` | 6.0.0 | no |
| `sebastian/cli-parser` | 2.0.1 | no |
| `sebastian/code-unit` | 2.0.0 | no |
| `sebastian/code-unit-reverse-lookup` | 3.0.0 | no |
| `sebastian/comparator` | 5.0.5 | no |
| `sebastian/complexity` | 3.2.0 | no |
| `sebastian/diff` | 5.1.1 | no |
| `sebastian/environment` | 6.1.0 | no |
| `sebastian/exporter` | 5.1.4 | no |
| `sebastian/global-state` | 6.0.2 | no |
| `sebastian/lines-of-code` | 2.0.2 | no |
| `sebastian/object-enumerator` | 5.0.0 | no |
| `sebastian/object-reflector` | 3.0.0 | no |
| `sebastian/recursion-context` | 5.0.1 | no |
| `sebastian/type` | 4.0.0 | no |
| `sebastian/version` | 4.0.1 | no |
| `theseer/tokenizer` | 1.3.1 | no |

## Expected dependency tree

The `composer.lock` file contains:
- `packages[]` — 2 production packages: monolog/monolog 3.10.0 and its transitive dep psr/log 3.0.2.
- `packages-dev[]` — 26 development packages: phpunit/phpunit 10.5.63 and all its transitives.

Every package in `packages[]` must be reported with `group = "main"`.
Every package in `packages-dev[]` must be reported with `group = "dev"`.

No package appears in both arrays, so no dual-scope ambiguity is present.

## Failure modes this probe detects

1. **Flat scope** — Mend reports all 28 packages under group `main`, dropping `dev` distinction entirely.
2. **Dev-as-prod** — Mend reports the 26 `require-dev` packages as `main` instead of `dev`.
3. **Dev packages dropped** — Mend reports only the 2 `main` packages and silently omits the 26 `dev` packages.
4. **Wrong transitive scope** — Mend correctly scopes `phpunit/phpunit` as `dev` but incorrectly promotes its transitives (e.g. `sebastian/*`, `myclabs/deep-copy`) to `main`.