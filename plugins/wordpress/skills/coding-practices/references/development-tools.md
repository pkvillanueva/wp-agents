# Development Tools Best Practices

**Reference Category:** Development Environment, Tooling
**Applies To:** WordPress Development, Build Tools, Testing
**Related:** Local Development, Version Control, Automation

## Purpose

This reference provides recommendations for development tools, including local environments, scaffolding, task runners, version control, command-line tools, and testing utilities.

## When to Use This Reference

- When setting up new WordPress development environment
- When choosing build tools and task runners
- When implementing testing strategies
- When selecting version control workflows
- When evaluating development tooling

## Key Principles

1. **Consistency** - Use recommended tools across projects for team efficiency
2. **Automation** - Automate repetitive tasks (builds, tests, deployments)
3. **Testing** - Implement accessibility and visual regression testing
4. **Version Control** - Use Git with clear workflow practices
5. **Documentation** - Document tool setup and usage

---

## Local Development Environments

### Option 1: Local WP (Recommended for Beginners)

**Tool:** [Local WP](https://localwp.com/)

**Features:**
- One-click WordPress installation
- Easy site management
- Multiple PHP versions
- Built-in WP-CLI
- SSL support
- Email testing
- Available for Mac, Windows, Linux

**Why Local WP?**
- Simple setup process
- Great for beginners
- Active development and support
- Free and reliable
- No Docker knowledge required

### Option 2: @wordpress/env (Recommended for Developers)

**Tool:** [@wordpress/env](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)

**Features:**
- Docker-based WordPress environment
- Configured via .wp-env.json
- Easy switching between projects
- Multiple WordPress/PHP versions
- Includes WP-CLI
- Test plugin/theme in isolation

**Installation:**

```bash
npm install -g @wordpress/env
```

**Usage:**

```bash
# Start environment (from plugin/theme directory)
wp-env start

# Stop environment
wp-env stop

# Reset environment
wp-env clean all
```

**When to use @wordpress/env:**
- Plugin/theme development
- Testing across multiple WordPress versions
- CI/CD pipelines
- Lightweight Docker-based workflow
- Working with @wordpress/scripts

**When to use Local WP:**
- Full-site development
- Client projects with many plugins
- Need GUI for site management
- Prefer visual interface over CLI

---

## Scaffolding

### Create Block Tool

**Official tool:** [@wordpress/create-block](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-create-block/)

**Purpose:**
- Scaffold WordPress blocks quickly
- Pre-configured build setup
- WordPress best practices included
- Modern development workflow

**Usage:**

```bash
# Create a new block
npx @wordpress/create-block my-block

# Create with template
npx @wordpress/create-block --template @wordpress/create-block-tutorial-template
```

---

## Task Runners and Build Tools

### @wordpress/scripts (Recommended)

**Official tool:** [@wordpress/scripts](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/)

**Purpose:** Zero-config build tool for WordPress development

**Features:**
- Asset bundling (JS, CSS)
- Modern JavaScript transpilation (Babel)
- Sass/SCSS compilation
- CSS post-processing (autoprefixer)
- ESLint and Stylelint integration
- Jest testing framework
- Hot module replacement
- Production builds with minification

**Installation:**

```bash
npm install @wordpress/scripts --save-dev
```

**Usage in package.json:**

```json
{
    "scripts": {
        "build": "wp-scripts build",
        "start": "wp-scripts start",
        "check-engines": "wp-scripts check-engines",
        "check-licenses": "wp-scripts check-licenses",
        "format": "wp-scripts format",
        "lint:css": "wp-scripts lint-style",
        "lint:js": "wp-scripts lint-js",
        "lint:pkg-json": "wp-scripts lint-pkg-json",
        "test:e2e": "wp-scripts test-e2e",
        "test:unit": "wp-scripts test-unit-js"
    }
}
```

**Why @wordpress/scripts?**
- Zero configuration needed
- Official WordPress tooling
- Automatic updates with WordPress releases
- Consistent with WordPress core development
- Community-supported

### @wordpress/env (Local Development)

**Tool:** [@wordpress/env](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)

**Purpose:** Local WordPress environment using Docker

**Installation:**

```bash
npm install @wordpress/env --save-dev
```

**Usage:**

```bash
# Start environment
wp-env start

# Stop environment
wp-env stop

# Clean environment (reset)
wp-env clean

# Run WP-CLI commands
wp-env run cli wp plugin list

# Access database
wp-env run cli wp db export
```

**Configuration (.wp-env.json):**

```json
{
    "core": "WordPress/WordPress#6.4",
    "phpVersion": "8.0",
    "plugins": [
        ".",
        "https://downloads.wordpress.org/plugin/wordpress-seo.latest-stable.zip"
    ],
    "themes": ["https://downloads.wordpress.org/theme/twentytwentythree.zip"]
}
```

**Why @wordpress/env?**
- Docker-based, works on all platforms
- Easy configuration
- Quick setup and teardown
- Integrates with @wordpress/scripts
- Official WordPress tooling

### Custom Webpack (When Needed)

**Use custom Webpack config when:**
- Very specific build requirements
- Complex module federation
- Specialized loaders beyond WordPress defaults
- Migrating legacy projects

**Note:** @wordpress/scripts uses Webpack internally and can be extended.

---

## Package Managers

### npm (JavaScript)

**Purpose:** Manage JavaScript dependencies

**Commands:**
```bash
# Install dependencies
npm install

# Add dependency
npm install lodash

# Add dev dependency
npm install --save-dev webpack

# Update packages
npm update

# Check for vulnerabilities
npm audit
```

### Composer (PHP)

**Purpose:** Manage PHP dependencies and WordPress plugins/themes

**Commands:**
```bash
# Install dependencies
composer install

# Add dependency
composer require monolog/monolog

# Add dev dependency
composer require --dev phpunit/phpunit

# Update packages
composer update
```

**WordPress Integration:**

Use [WPackagist](https://wpackagist.org/) to install WordPress plugins/themes via Composer.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "require": {
        "wpackagist-plugin/wordpress-seo": "^20.0",
        "wpackagist-theme/twentytwentythree": "*"
    }
}
```

---

## Version Control

### Git (Standard)

**Official tool:** [Git](https://git-scm.com/)

**Policy:** Use Git command line. GUIs are permitted but not supported internally.

**Why command line?**
- Universal across systems
- Better understanding of Git concepts
- Easier to troubleshoot
- More powerful and flexible

**Common Commands:**

```bash
# Clone repository
git clone <url>

# Create branch
git checkout -b feature/my-feature

# Stage changes
git add .

# Commit changes
git commit -m "Add feature X"

# Push to remote
git push origin feature/my-feature

# Pull latest changes
git pull origin main

# View status
git status

# View commit history
git log --oneline

# Create tag
git tag v1.0.0

# Push tags
git push --tags
```

### Git Workflow Best Practices

**Branch naming:**
- `feature/description` - New features
- `fix/description` - Bug fixes
- `hotfix/description` - Urgent production fixes
- `chore/description` - Maintenance tasks

**Commit messages:**
- Present tense: "Add feature" not "Added feature"
- Concise subject line (50 chars or less)
- Detailed description if needed (wrap at 72 chars)
- Reference issue numbers

**Example:**
```
Add user authentication feature

- Implement login form
- Add password reset functionality
- Create user session management

Fixes #123
```

---

## Command Line Tools

### WP-CLI

**Official tool:** [WP-CLI](https://wp-cli.org/)

**Purpose:** WordPress command-line interface

**Included in:** Local WP (automatic)

**Common Commands:**

```bash
# Install WordPress
wp core install --url=example.com --title=Site --admin_user=admin

# Install plugin
wp plugin install wordpress-seo --activate

# Install theme
wp theme install twentytwentythree --activate

# Export/import database
wp db export backup.sql
wp db import backup.sql

# Search and replace
wp search-replace 'oldurl.com' 'newurl.com'

# Clear cache
wp cache flush

# Generate test content
wp post generate --count=100

# Run custom scripts
wp eval-file scripts/my-script.php
```

**Why WP-CLI?**
- Fast database operations
- Automate repetitive tasks
- Essential for large sites (VIP, WP Engine)
- Scriptable and CI/CD friendly

---

## Accessibility Testing

### Browser Extensions

**Headings Map:**
- [Chrome Extension](https://chrome.google.com/webstore/detail/headingsmap/)
- [Firefox Extension](https://addons.mozilla.org/firefox/addon/headingsmap/)
- Shows heading structure of page

**WAVE:**
- [Web Tool](http://wave.webaim.org/)
- [Chrome Extension](https://chrome.google.com/webstore/detail/wave-evaluation-tool/)
- Identifies accessibility errors

**axe DevTools:**
- [Chrome Extension](https://chrome.google.com/webstore/detail/axe-devtools/)
- [Firefox Extension](https://addons.mozilla.org/firefox/addon/axe-devtools/)
- Comprehensive accessibility auditing

**Lighthouse:**
- Built into Chrome DevTools
- Audits accessibility, performance, SEO
- Generate reports

### Color Contrast Tools

**Contrast Ratio:**
- [Tool](https://leaverou.github.io/contrast-ratio/)
- Check color contrast compliance

**Tanaguru Contrast Finder:**
- [Tool](http://contrast-finder.tanaguru.com/)
- Find accessible color alternatives

### Screen Readers

**VoiceOver (macOS/iOS):**
- Built into macOS
- [WebAIM Guide](http://webaim.org/articles/voiceover/)

**NVDA (Windows):**
- Free screen reader
- [Download](https://www.nvaccess.org/)
- [WebAIM Guide](http://webaim.org/articles/nvda/)

**JAWS (Windows):**
- Professional screen reader
- [WebAIM Guide](http://webaim.org/articles/jaws/)

### Automated Testing

**Pa11y:**
- [Tool](https://github.com/pa11y/pa11y)
- Command-line accessibility tester
- Integrates with CI/CD

```bash
# Install
npm install -g pa11y

# Test URL
pa11y https://example.com

# Test with specific standard
pa11y --standard WCAG2AA https://example.com
```

**Add to package.json:**

```json
{
    "scripts": {
        "test:a11y": "pa11y --standard WCAG2AA https://staging.example.com"
    }
}
```

---

## Visual Regression Testing

### BackstopJS (Recommended)

**Tool:** [BackstopJS](https://github.com/garris/BackstopJS)

**Purpose:**
- Compare screenshots before/after changes
- Catch unintended visual changes
- Test responsive breakpoints

**Setup:**

```bash
# Install
npm install --save-dev backstopjs

# Initialize
npx backstop init

# Create reference screenshots
npx backstop reference

# Run tests (compare to reference)
npx backstop test
```

**Configuration (backstop.json):**

```json
{
    "viewports": [
        {
            "name": "phone",
            "width": 375,
            "height": 667
        },
        {
            "name": "tablet",
            "width": 768,
            "height": 1024
        },
        {
            "name": "desktop",
            "width": 1920,
            "height": 1080
        }
    ],
    "scenarios": [
        {
            "label": "Homepage",
            "url": "https://example.com",
            "selectors": ["body"]
        },
        {
            "label": "About Page",
            "url": "https://example.com/about",
            "selectors": ["body"]
        }
    ]
}
```

**Use cases:**
- Plugin updates
- Theme changes
- CSS refactoring
- Dependency updates

---

## Testing Frameworks

### PHP Testing

**PHPUnit:**
- [Official Docs](https://phpunit.de/)
- Unit testing for PHP
- WordPress integration via WP_UnitTestCase

### JavaScript Testing

**Jest:**
- [Official Docs](https://jestjs.io/)
- Testing framework by Facebook
- Works with React

**Testing Library:**
- [Official Docs](https://testing-library.com/)
- Test React components
- Focus on user behavior

---

## Linting and Code Quality

### ESLint (JavaScript)

**Configuration (.eslintrc.json):**

```json
{
    "extends": ["plugin:@wordpress/eslint-plugin/recommended"],
    "rules": {
        "no-console": "warn"
    }
}
```

**Or use @wordpress/scripts (includes ESLint):**

```json
{
    "scripts": {
        "lint:js": "wp-scripts lint-js",
        "format": "wp-scripts format"
    }
}
```

**Commands:**

```bash
# Lint files
npm run lint:js

# Auto-fix issues
npm run lint:js -- --fix

# Format code
npm run format
```

### Stylelint (CSS)

**Configuration:**

```json
{
    "extends": "stylelint-config-standard",
    "rules": {
        "indentation": "tab"
    }
}
```

### PHP CodeSniffer

**Install:**

```bash
composer require --dev squizlabs/php_codesniffer
composer require --dev wp-coding-standards/wpcs
```

**Configure:**

```bash
# Set WordPress coding standards
./vendor/bin/phpcs --config-set installed_paths vendor/wp-coding-standards/wpcs

# Run
./vendor/bin/phpcs --standard=WordPress path/to/code
```

---

## Quick Reference

### Local Development
- ✅ Use Local WP or @wordpress/env
- ✅ Install WP-CLI for command-line operations
- ✅ Enable SSL for local development
- ✅ Test across multiple PHP versions
- ✅ Use Docker for consistent environments

### Build Tools
- ✅ Use @wordpress/scripts for WordPress projects
- ✅ Use @wordpress/env for local development
- ✅ Automate asset building
- ✅ Minify for production
- ✅ Use source maps for debugging

### Version Control
- ✅ Use Git command line
- ✅ Follow branch naming conventions
- ✅ Write clear commit messages
- ✅ Use .gitignore properly
- ✅ Never commit node_modules or vendor

### Testing
- ✅ Run accessibility tests with Pa11y
- ✅ Use visual regression testing for CSS changes
- ✅ Test with screen readers
- ✅ Check color contrast
- ✅ Test keyboard navigation

### Code Quality
- ✅ Use ESLint for JavaScript
- ✅ Use Stylelint for CSS
- ✅ Use PHPCS for PHP
- ✅ Run linters in pre-commit hooks
- ✅ Fix linting errors before committing

---

## Tool Installation Checklist

### Required
- [ ] Git
- [ ] Node.js (LTS version)
- [ ] npm
- [ ] Composer
- [ ] Local WP or Docker (for @wordpress/env)

### Recommended
- [ ] @wordpress/scripts (build tools)
- [ ] @wordpress/env (local environment)
- [ ] WP-CLI (included in Local WP and @wordpress/env)
- [ ] nvm (Node Version Manager)
- [ ] EditorConfig plugin for IDE
- [ ] ESLint plugin for IDE
- [ ] Git GUI (optional, for visualization)

### Browser Extensions
- [ ] WAVE
- [ ] axe DevTools
- [ ] Headings Map
- [ ] Lighthouse (built into Chrome)

---

## Further Reading

- [Local WP Documentation](https://localwp.com/help-docs/)
- [WP-CLI Commands](https://developer.wordpress.org/cli/commands/)
- [Git Documentation](https://git-scm.com/doc)
- [WebAIM Accessibility Resources](https://webaim.org/)
- [@wordpress/scripts Documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/)
- [@wordpress/env Documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)
