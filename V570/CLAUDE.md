# CLAUDE.md — v570 Project

## What This Is

Magento 2.4.7-p3 CE e-commerce for a Spanish school supplies retailer. PHP 8.3, Hyva/McYadra theme (TailwindCSS + AlpineJS), Magestore POS. Bilingual: `en_US` + `es_ES`.

## Module Landscape

124 enabled modules — see `app/etc/config.php`. Custom code in `app/code/`:

| Namespace   | Count | Purpose |
|-------------|-------|---------|
| `W2e`       | 11    | School-specific: BooksCart, DeliveryNote, CustomerChildren, SchoolShipping, NavSync |
| `Bss`       | 10    | BSS customizations: Bizum payment, HyvaRma, Security, DataImport |
| `Magestore` | 91+   | Full POS suite: inventory, fulfillment, gift vouchers, rewards, multi-payment |
| `Hyva`      | 5     | Hyva compatibility bridges |
| Others      | ~7    | Mageants CookieLaw, Oct8ne, Redsys payment |

Third-party packages in `composer.json`. Patches in `patches/`.

## Tech Stack

- **PHP**: 8.3 | **Magento**: 2.4.7-p3 CE | **Search**: Elasticsearch
- **Frontend**: Hyva 1.3.10 → theme `Hyva/McYadra` | config: `app/etc/hyva-themes.json`
- **Auth**: `auth.json` — do not commit

## Key Commands

```bash
php bin/magento cache:flush
php bin/magento setup:di:compile          # after di.xml changes
php bin/magento setup:upgrade             # after db_schema.xml changes
php bin/magento setup:static:deploy -t Hyva/McYadra --no-parent --jobs=4 en_US es_ES -f

bin/hyva.sh          # rebuild TailwindCSS (also: bin/css.sh)
bin/deploy_dev.sh    # dev deploy
bin/live_deploy.sh   # production deploy
```

## Testing & Quality

```bash
vendor/bin/phpunit dev/tests/unit/phpunit.xml.dist
vendor/bin/phpunit dev/tests/integration/phpunit.xml.dist
vendor/bin/phpcs --standard=dev/tests/static/framework/Magento/ruleset.xml app/code
vendor/bin/phpmd app/code text dev/tests/static/testsuite/Magento/Test/Php/_files/phpmd/ruleset.xml
vendor/bin/php-cs-fixer fix app/code --config=.php-cs-fixer.dist.php
```

Pre-commit: `.bss-cli.yaml` — PHPCS (Magento + BSS), PHPMD, PHP Compat (8.2–8.3), PHP-CS-Fixer, CPD. CI (Jenkins) runs the same checks on every PR — fix all violations before merging.

## Code Conventions

- Magento Coding Standard (PHPCS ruleset above) + PSR-2 via `.php-cs-fixer.dist.php`
- Target PHP 8.2–8.3 | no magic numbers
- New modules: correct namespace in `app/code/`, follow existing module structure
- Use **plugins** (around/before/after) or **preferences** — never rewrite vendor classes directly
- No `ObjectManager::getInstance()` outside factories/proxies

## Environment

- Local: `v570.test` (web), `v570-nav.test` (NavSync) | Docker: `.denv.env` (project: `v570`)
- `app/etc/env.php` — DB/cache/session credentials, never commit changes
- `app/etc/config.php` — module enable/disable state, commit this file
- `generated/` and `var/` — runtime artifacts, never commit

## Development Workflow

1. Identify namespace: `W2e` (school features), `Bss` (BSS customizations), or extend existing.
2. Extend via plugin/preference — no vendor edits.
3. `di.xml` changed → `setup:di:compile` | `db_schema.xml` changed → `setup:upgrade` | Hyva/CSS changed → `bin/hyva.sh`.
4. Run `.bss-cli.yaml` locally before pushing.

## Self-test Checklist

For any completed task, cover:

1. **Happy path** — end-to-end: storefront + admin + POS where applicable.
2. **Edge cases** — empty cart, out-of-stock, tier prices, guest vs logged-in, `en_US` vs `es_ES`.
3. **Negative cases** — invalid input, missing ACL, payment failure (Bizum/Redsys).
4. **Performance** — no N+1 queries; cache invalidated correctly; no unnecessary full reindex.
5. **POS** — if touching inventory/orders/pricing, verify Magestore POS flows unaffected.
6. **Hyva frontend** — AlpineJS reactivity works; no console errors; Tailwind classes not purged.
7. **Regression** — SchoolShipping, DeliveryNote, NavSync, and other related modules still function.

## Code Review Criteria

For any code review, always check:

1. **Correctness** — logic matches requirement; all branches handled.
2. **Magento patterns** — DI/plugins/preferences used correctly; no raw `ObjectManager`.
3. **Performance** — no queries in loops; collections/repositories used; cache tags declared.
4. **Security** — ACL checks present; user input sanitized; no raw SQL injection risk.
5. **Bilingual** — all user-facing strings in `__()`; no hardcoded Spanish/English text.
6. **Standard** — passes `.bss-cli.yaml` (PHPCS, PHPMD, PHP-CS-Fixer).
7. **Impact** — effect on Magestore POS, Hyva theme, other `W2e`/`Bss` modules.
