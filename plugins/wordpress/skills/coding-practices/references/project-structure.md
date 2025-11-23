# Project Structure & Design Best Practices

**Reference Category:** Project Organization
**Applies To:** WordPress Themes, Plugins, Modern Web Projects
**Related:** Composer, npm, Version Control, Modular Code

## Purpose

This reference provides best practices for organizing project files, managing dependencies, integrating third-party services, and writing modular WordPress code.

## When to Use This Reference

- When starting a new WordPress theme or plugin
- When setting up dependency management (Composer, npm)
- When integrating third-party APIs and services
- When structuring modular, maintainable code
- When documenting project integrations

## Key Principles

1. **Consistent Structure** - Follow standardized file organization
2. **Dependency Management** - Use Composer and npm properly
3. **Modular & Decoupled** - Themes and plugins should work independently
4. **Document Integrations** - Maintain INTEGRATIONS.md for third-party services
5. **Security Conscious** - Never commit credentials or API keys

---

## Theme and Plugin File Organization

### Standard Directory Structure

```
|- assets/
|  |- css/
|  |  |- admin/                    # Admin styles
|  |  |- frontend/
|  |  |  |- base/                  # CSS foundation
|  |  |  |- components/            # Component styles
|  |  |  |- global/                # Variables, configs
|  |  |  |- layout/                # Layout helpers
|  |  |  |- templates/             # Template-specific CSS
|  |  |- shared/                   # Shared admin/frontend CSS
|  |- fonts/                       # Custom/hosted fonts
|  |- images/                      # Theme images
|  |- js/
|  |  |- admin/                    # Admin JavaScript
|  |  |- frontend/
|  |  |  |- components/            # Component JS
|  |  |- shared/                   # Shared admin/frontend JS
|  |- svg/                         # Vector images for processing
|- bin/                            # WP-CLI and shell scripts
|- gulp-tasks/                     # Individual Gulp tasks (if using Gulp)
|- includes/
|  |- classes/                     # PHP classes
|- languages/                      # Translation files
|- node_modules/                   # npm packages
|- partials/                       # Template parts
|- templates/                      # Page templates
|- tests/
|  |- php/                         # PHP tests
|  |- js/                          # JavaScript tests
|- vendor/                         # Composer dependencies
|- .babelrc                        # Babel config
|- .editorconfig                   # Editor config
|- .eslintrc                       # ESLint config
|- composer.json                   # Composer manifest
|- package.json                    # npm manifest
|- webpack.config.js               # Webpack config
```

### Benefits

- **Consistency:** Easy for new developers to navigate
- **Separation:** Clear boundaries between admin/frontend/shared code
- **Scalability:** Structure supports growth
- **Maintainability:** Related files grouped logically

---

## Dependencies and Package Management

### npm for JavaScript

Use npm for front-end dependencies and build tools.

```json
{
    "name": "my-theme",
    "version": "1.0.0",
    "dependencies": {
        "lodash": "^4.17.21"
    },
    "devDependencies": {
        "@wordpress/scripts": "^26.0.0",
        "@wordpress/env": "^8.0.0"
    },
    "scripts": {
        "build": "wp-scripts build",
        "start": "wp-scripts start",
        "lint:js": "wp-scripts lint-js",
        "lint:css": "wp-scripts lint-style",
        "format": "wp-scripts format",
        "test:unit": "wp-scripts test-unit-js"
    }
}
```

**Dependencies vs DevDependencies:**
- **dependencies:** Code used in production (lodash, react)
- **devDependencies:** Build tools (webpack, eslint, babel)

### Composer for PHP

Use Composer for PHP dependencies and WordPress plugins/themes.

```json
{
    "name": "company/my-theme",
    "description": "Custom WordPress theme",
    "type": "wordpress-theme",
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "require": {
        "php": ">=7.4",
        "wpackagist-plugin/wordpress-seo": "^20.0",
        "wpackagist-plugin/advanced-custom-fields": "*"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.5",
        "wp-coding-standards/wpcs": "^2.3"
    }
}
```

### Composer-Based WordPress Setup

```
|- composer.json                   # Define dependencies
|- wp-config.php                   # WordPress configuration
|- wp/                             # WordPress core (installed by Composer)
|- wp-content/
|  |- themes/
|  |  |- custom-theme/             # Custom theme
|  |- plugins/
|  |  |- custom-plugin/            # Custom plugin
```

**composer.json example:**

```json
{
    "name": "company/project-slug",
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "extra": {
        "wordpress-install-dir": "wp"
    },
    "require": {
        "johnpbloch/wordpress": "^6.4",
        "wpackagist-plugin/wordpress-importer": "^0.8",
        "wpackagist-plugin/debug-bar": "^1.1"
    }
}
```

### When to Use Package Managers

**Use when:**
- Dependencies are public and available on npm/Packagist
- Team uses consistent tooling
- You need automated updates
- Working on modern hosting (not VIP/restrictive environments)

**Document exceptions:**
- If unable to use package manager, document why
- Document library version and reason for manual inclusion
- Keep in project documentation or README

### Package Selection Criteria

Before adding a package, evaluate:

1. **Active Development?** - Check recent commits, issues
2. **Community Reputation?** - Well-known and trusted?
3. **Security History?** - How quickly are CVEs addressed?
4. **Dependency Count?** - Fewer dependencies = less risk
5. **License?** - Compatible with project requirements?
6. **Forkability?** - Can you maintain if abandoned?

### Version Constraints

**Use semantic versioning constraints:**

```json
{
    "dependencies": {
        "package-name": "^1.2.3"    // Recommended: 1.x.x
    }
}
```

**Version syntax:**
- `^1.2.3` - Compatible with 1.x.x (recommended for semver packages)
- `~1.2.3` - Compatible with 1.2.x (for non-semver like WP plugins)
- `1.2.3` - Exact version (too strict, avoid)
- `*` - Any version (dangerous, never use)

**Commit lock files:**
- `package-lock.json` (npm)
- `composer.lock` (Composer)

These ensure consistent installs across team.

---

## Third-Party Integrations

### INTEGRATIONS.md File

Document ALL third-party integrations in project root.

**Template:**

```markdown
# Third-Party Integrations

## Service Name

Brief description of what the service provides.

### Scheduling
- When and how the integration is triggered
- Frequency of API calls
- Triggering events

### Integration Points
- File paths where integration code exists
- Hooks and filters used
- Functions that interact with service

### Credentials
- Link to credential documentation (not the credentials!)
- Environment variables used
- Required permissions/scopes

### Development vs Production
- Separate dev/test accounts if available
- How to switch between environments

---

## Example: Twitter API

Social media posting integration for auto-publishing posts.

### Scheduling
- Triggered when post is published (publish_post hook)
- Manual tweets via admin button
- Rate limit: 300 tweets per 3 hours

### Integration Points
- `/includes/classes/class-twitter-client.php`
- `/includes/integrations/twitter-hooks.php`
- `publish_post` hook

### Credentials
- See project management tool: Project > Credentials > Twitter
- Environment variables: `TWITTER_API_KEY`, `TWITTER_API_SECRET`

### Development API
- Dev credentials available in 1Password
- Tweets post to @DevAccountName
```

### API Keys and Credentials

**Never hardcode credentials!**

#### ✅ Do: Use environment variables

```php
<?php
// wp-config.php or similar
define( 'TWITTER_API_KEY', getenv( 'TWITTER_API_KEY' ) );
define( 'TWITTER_API_SECRET', getenv( 'TWITTER_API_SECRET' ) );

// In plugin/theme
if ( ! defined( 'TWITTER_API_KEY' ) ) {
    if ( 'production' === wp_get_environment_type() ) {
        error_log( 'Missing TWITTER_API_KEY in production!' );
    }
    // Use dev/fallback key
    define( 'TWITTER_API_KEY', 'dev-key-here' );
}
```

#### ❌ Don't: Hardcode credentials

```php
<?php
// NEVER DO THIS
$api_key = 'sk_live_abc123xyz789';
```

### Environment-Based Execution

Prevent accidental production triggers in dev/staging.

```php
<?php
// Only send emails in production
if ( 'production' === wp_get_environment_type() ) {
    wp_mail( $to, $subject, $message );
}

// Only post to social media in production
if ( 'production' === wp_get_environment_type() && defined( 'TWITTER_API_KEY' ) ) {
    send_tweet( $message );
}
```

**Use `wp_get_environment_type()`:**
- Returns: `'local'`, `'development'`, `'staging'`, or `'production'`
- Set via `WP_ENVIRONMENT_TYPE` constant
- Available since WordPress 5.5

---

## Modular Code

### Plugins Should Stand Alone

Plugins must function when theme is deactivated or changed.

#### ✅ Do: Use hooks for theme communication

**Plugin:**
```php
<?php
// Plugin exposes functionality via action/filter
function my_plugin_get_data() {
    return array( 'foo' => 'bar' );
}
add_filter( 'my_plugin_data', 'my_plugin_get_data' );
```

**Theme:**
```php
<?php
// Theme consumes via filter
$data = apply_filters( 'my_plugin_data', array() );
if ( ! empty( $data ) ) {
    // Use data
}
```

#### ❌ Don't: Call plugin functions directly in theme

```php
<?php
// WRONG - breaks if plugin disabled
$data = my_plugin_get_data();
```

### Themes Should Not Require Plugins

Themes should degrade gracefully when plugins are disabled.

```php
<?php
// Check if plugin provides functionality
if ( function_exists( 'my_plugin_function' ) ) {
    my_plugin_function();
} else {
    // Fallback behavior
    echo 'Feature unavailable';
}

// Or use actions/filters
do_action( 'my_plugin_action' ); // Safe even if plugin disabled
```

### Decouple with add_theme_support

Use `add_theme_support()` and `current_theme_supports()` to coordinate features.

**Theme:**
```php
<?php
// Theme declares support
add_theme_support( 'custom-social-sharing' );
```

**Plugin:**
```php
<?php
// Plugin only adds feature if theme supports it
if ( current_theme_supports( 'custom-social-sharing' ) ) {
    add_action( 'wp_footer', 'add_social_sharing_buttons' );
}
```

---

## Project Configuration Files

### .editorconfig

Maintain consistent coding styles across editors.

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
insert_final_newline = true
charset = utf-8

[*.{php,js,css,scss}]
indent_style = tab
indent_size = 4
end_of_line = lf
```

Developers install EditorConfig plugin for their IDE from [editorconfig.org](https://editorconfig.org/).

### composer.json (Required)

Every project must include `composer.json`, even if no dependencies.

```json
{
    "name": "company/project-name",
    "description": "Project description",
    "type": "wordpress-theme",
    "license": "GPL-2.0-or-later",
    "require": {
        "php": ">=7.4"
    }
}
```

**Why?**
- Allows project to be pulled into other projects via Composer
- Documents PHP version requirements
- Standardizes project metadata

---

## Quick Reference Checklist

### File Organization
- ✅ Follow standard directory structure
- ✅ Separate admin/frontend/shared code
- ✅ Keep related files together
- ✅ Use meaningful directory names

### Dependency Management
- ✅ Use npm for JavaScript dependencies
- ✅ Use Composer for PHP dependencies
- ✅ Commit lock files
- ✅ Document packages if not using package manager
- ✅ Evaluate packages before adding

### Third-Party Integrations
- ✅ Create INTEGRATIONS.md file
- ✅ Document all external services
- ✅ Never commit credentials
- ✅ Use environment variables
- ✅ Check `wp_get_environment_type()` before triggers

### Modular Code
- ✅ Plugins work without themes
- ✅ Themes work without plugins
- ✅ Use hooks for communication
- ✅ Use `add_theme_support()` for features
- ✅ Check function/feature existence before use

### Configuration
- ✅ Include .editorconfig
- ✅ Include composer.json
- ✅ Include package.json
- ✅ Document environment variables
- ✅ Provide .env.example

---

## Further Reading

- [Composer Documentation](https://getcomposer.org/doc/)
- [npm Documentation](https://docs.npmjs.com/)
- [WPackagist](https://wpackagist.org/) - WordPress plugins/themes for Composer
- [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)
- [EditorConfig](https://editorconfig.org/)
