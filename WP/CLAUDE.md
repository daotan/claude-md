# CLAUDE.md — WordPress Project

## What This Is

WordPress site built with a page builder (Elementor or Avada). Custom theme extends the builder's parent theme. Custom functionality lives in a child theme and/or a site-specific plugin.

## Project Structure

```
wp-content/
├── themes/
│   ├── avada/               # or elementor/hello-elementor — DO NOT edit
│   ├── avada-child/         # or hello-elementor-child — ALL custom CSS/PHP goes here
│   └── ...
├── plugins/
│   ├── elementor/           # or fusion-builder (Avada) — DO NOT edit
│   ├── site-plugin/         # site-specific custom code (CPTs, hooks, shortcodes)
│   └── ...
├── uploads/                 # media — never commit
└── ...
wp-config.php                # credentials — never commit
.env or .env.local           # local overrides — never commit
```

> Custom code belongs in the **child theme** and/or **site-specific plugin only**.
> Never modify parent theme files or plugin vendor files — updates will overwrite them.

## Tech Stack

- **CMS**: WordPress (latest stable)
- **Page Builder**: Elementor Pro **or** Avada (Fusion Builder) — confirm which per project
- **PHP**: 8.1+
- **Build tools**: (if present) npm/Vite or Webpack for child theme assets

## Key Commands

```bash
# Install PHP dependencies (if composer.json exists in child theme/plugin)
composer install

# Install JS dependencies & build assets (if package.json exists)
npm install
npm run build        # production build
npm run dev          # watch mode

# WP CLI (if available)
wp cache flush
wp plugin activate <slug>
wp theme activate <slug>
wp search-replace 'old-domain.com' 'new-domain.com' --all-tables

# Database export/import
wp db export backup.sql
wp db import backup.sql
```

## Development Workflow

1. **Never edit in production directly.** Use staging or local (LocalWP / Lando / Docker).
2. Child theme for style overrides: `wp-content/themes/<child>/style.css` and `functions.php`.
3. Custom post types, taxonomies, shortcodes, hooks → site-specific plugin (keeps theme-independent).
4. Elementor custom widgets → register via `Elementor\Plugin::instance()->widgets_manager->register()`.
5. Avada custom elements → use Fusion Builder element API in the site plugin.
6. After pulling DB from staging/prod, run `wp search-replace` to fix serialized URLs.

## Page Builder Conventions

### Elementor
- Layout is stored as post meta (`_elementor_data`) — avoid manual DB edits.
- Global styles/colors/fonts live in **Elementor → Site Settings** — document them, don't hardcode hex values.
- Custom CSS per-widget via the "Advanced → Custom CSS" tab; global custom CSS via **Elementor → Custom CSS**.
- Kit export (`wp-content/uploads/elementor/css/`) is regenerated — do not commit.

### Avada
- Layout is stored via Fusion Builder shortcodes in `post_content` — readable in DB.
- Global options in **Avada → Theme Options** (stored in `wp_options` as `avada_options`).
- Child theme overrides: copy template from `avada/templates/` to `avada-child/templates/` and modify.
- Fusion sliders and forms are stored as CPTs — export via Avada tools before DB wipe.

## Code Conventions

- PHP: follow [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/) — tabs for indentation, Yoda conditions.
- Prefix all functions, hooks, classes, and options with a project slug (e.g., `mypfx_`) to avoid collisions.
- Sanitize/escape all input and output: `sanitize_text_field()`, `esc_html()`, `esc_url()`, `wp_kses_post()`.
- Use `wp_enqueue_scripts` / `wp_enqueue_style` — never hardcode `<script>` or `<link>` tags.
- Nonces for all AJAX and form submissions.
- No direct DB queries when a WP API exists (`WP_Query`, `get_option`, `update_post_meta`).

## Static Analysis & Linting

```bash
# PHPCS with WordPress standard (if configured)
vendor/bin/phpcs --standard=WordPress wp-content/themes/my-child-theme
vendor/bin/phpcs --standard=WordPress wp-content/plugins/site-plugin

# PHP CS Fixer (if .php-cs-fixer.dist.php exists)
vendor/bin/php-cs-fixer fix wp-content/themes/my-child-theme

# ESLint (if .eslintrc exists)
npm run lint
```

## Environment & Config

- **Local**: `.env` or `wp-config-local.php` — never commit credentials.
- **Staging/Prod**: `wp-config.php` injected via server or CI secrets.
- `WP_DEBUG`, `WP_DEBUG_LOG` — enable locally, disable on production.
- Uploads (`wp-content/uploads/`) — sync via rsync or S3, not git.

## Gotchas

- Elementor regenerates CSS on save — `wp-content/uploads/elementor/css/` is runtime output, exclude from git.
- Avada theme options are stored in `wp_options` — export them via **Avada → Export** before wiping DB.
- Serialized data in the DB breaks if you manually search-replace URLs — always use `wp search-replace`.
- Page builder data is tightly coupled to plugin version — don't upgrade builder plugins on production without testing on staging first.
- `functions.php` in the child theme is loaded **in addition to**, not instead of, the parent — no need to copy everything.
- ACF field groups: export as JSON to `acf-json/` folder in the child theme for version control.
