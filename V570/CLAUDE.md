# CLAUDE.md — v570 Project

## What This Is

Magento 2.4.7-p3 (Community Edition) e-commerce platform for a Spanish school supplies retailer. PHP 8.3, Hyva/McYadra custom frontend theme, Magestore POS system. Bilingual: `en_US` + `es_ES`.

## Module Landscape

124 enabled modules across 8 vendors — see `app/etc/config.php` for the full list.

Custom code lives in `app/code/`:

| Namespace | Count | Purpose |
|-----------|-------|---------|
| `W2e`     | 11    | School-specific features: BooksCart, DeliveryNote, CustomerChildren, SchoolShipping, NavSync, etc. |
| `Bss`     | 10    | BSS Commerce customizations: Bizum payment, HyvaRma, Security, DataImport, etc. |
| `Magestore` | 91+ | Full POS suite: inventory, fulfillment, gift vouchers, rewards, multi-payment terminals |
| `Hyva`    | 5     | Hyva compatibility bridges for third-party modules |
| Others    | ~7    | Mageants CookieLaw, Oct8ne, Redsys payment |

Third-party Composer packages declared in `composer.json`. Patches in `patches/`.

## Tech Stack

- **PHP**: 8.3 | **Magento**: 2.4.7-p3 CE
- **Frontend**: Hyva 1.3.10 → custom theme `Hyva/McYadra` (TailwindCSS + AlpineJS)
- **Search**: Elasticsearch
- **Theme config**: `app/etc/hyva-themes.json`
- **Auth for private repos**: `auth.json` (do not commit)

## Key Commands

```bash
# Magento CLI
php bin/magento <command>
php bin/magento cache:flush
php bin/magento setup:di:compile
php bin/magento setup:upgrade

# Static asset deployment (always specify theme + locales)
php bin/magento setup:static:deploy -t Hyva/McYadra --no-parent --jobs=4 en_US es_ES -f

# Hyva CSS compilation (TailwindCSS)
bin/hyva.sh          # or: bin/css.sh

# Deployment helpers
bin/deploy_dev.sh    # dev environment
bin/live_deploy.sh   # production
```

## Testing

```bash
# Unit tests
vendor/bin/phpunit dev/tests/unit/phpunit.xml.dist

# Integration tests (requires DB)
vendor/bin/phpunit dev/tests/integration/phpunit.xml.dist

# Static analysis
vendor/bin/phpcs --standard=dev/tests/static/framework/Magento/ruleset.xml app/code
vendor/bin/phpmd app/code text dev/tests/static/testsuite/Magento/Test/Php/_files/phpmd/ruleset.xml

# PHP CS Fixer
vendor/bin/php-cs-fixer fix app/code --config=.php-cs-fixer.dist.php

# Acceptance tests (MFTF)
dev/tests/acceptance/
```

Pre-commit validation is defined in `.bss-cli.yaml` — runs PHPCS (Magento + BSS standards), PHPMD, PHP Compatibility (8.2–8.3), PHP-CS-Fixer, and copy/paste detection.

## CI/CD

Jenkins pipeline (`Jenkinsfile`): Build → `composer install` → `setup:di:compile` → Validate (PR only) → Deploy.

PR validation runs `.bss-cli.yaml` checks automatically. Fix all violations before merging.

## Code Conventions

- Follow **Magento Coding Standard** (PHPCS ruleset: `dev/tests/static/framework/Magento/ruleset.xml`)
- PHP style enforced by `.php-cs-fixer.dist.php` (PSR-2 + Magento extensions)
- Target PHP 8.2–8.3 compatibility
- No magic numbers (PHP Magic Number Detector is enabled)
- New modules go under the appropriate namespace in `app/code/`; follow existing module structure

## Environment

- Local domain: `v570.test` (web), `v570-nav.test` (NavSync)
- Docker config: `.denv.env` (project name: `v570`)
- Runtime config: `app/etc/env.php` (DB credentials, cache, session — never commit changes)
- Module enable/disable state: `app/etc/config.php` (commit this file)

## Gotchas

- After adding/modifying DI config (`di.xml`), run `setup:di:compile`.
- After schema changes (`db_schema.xml`), run `setup:upgrade`.
- Hyva theme changes need TailwindCSS rebuild — run `bin/hyva.sh` or `bin/css.sh`.
- The Magestore POS suite is a large dependency; avoid modifying vendor files — use plugins/preferences instead.
- `generated/` and `var/` are runtime artifacts — never commit them.

