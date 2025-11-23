# PHP Best Practices for WordPress Development

**Reference Category:** Backend Development
**Applies To:** WordPress Projects, PHP 7.4+, PHP 8.x
**Related:** Security Practices, Performance Optimization, WordPress Coding Standards

## Purpose

This reference provides comprehensive PHP best practices specifically for WordPress development. It covers performance optimization, design patterns, security, code style, and testing approaches that ensure maintainable, secure, and scalable WordPress projects.

## When to Use This Reference

- When writing custom WordPress plugins or themes
- When optimizing database queries and caching strategies
- When implementing security measures (validation, sanitization, nonces)
- When structuring PHP code for maintainability
- When working with WordPress APIs and hooks

## Key Principles

1. **Performance First** - Write efficient database queries, implement proper caching, and optimize data storage
2. **Security Always** - Validate input, sanitize output, escape late, use nonces for state-changing operations
3. **Follow WordPress Standards** - Use WordPress coding standards and documentation practices
4. **Keep It Simple** - Avoid unnecessary abstraction, write clear and maintainable code
5. **Namespace Everything** - Use proper PHP namespacing outside of template files

---

## Performance

### Database Queries with WP_Query

Always use `WP_Query` for database queries and optimize with appropriate parameters.

#### ✅ Do: Optimize WP_Query

```php
$query = new WP_Query( array(
    'post_type' => 'post',
    'posts_per_page' => 20, // Never use -1
    'no_found_rows' => true, // Skip pagination calculation
    'update_post_meta_cache' => false, // Skip if not using post meta
    'update_post_term_cache' => false, // Skip if not using terms
) );
```

#### ❌ Don't: Use inefficient queries

```php
// NEVER do this - can crash site with many posts
$query = new WP_Query( array(
    'posts_per_page' => -1,
) );

// Avoid post__not_in - filter in PHP instead
$query = new WP_Query( array(
    'post__not_in' => $large_array,
) );
```

### Caching with Object Cache

Implement proper caching to reduce database load.

#### ✅ Do: Cache and prime on update

```php
/**
 * Get top commented posts with cache priming on update.
 */
function prefix_get_top_commented_posts( $force_refresh = false ) {
    $cache_key = 'prefix_top_commented_posts';
    $top_posts = wp_cache_get( $cache_key, 'top_posts' );

    if ( true === $force_refresh || false === $top_posts ) {
        $top_posts = new WP_Query( 'orderby=comment_count&posts_per_page=10' );

        if ( ! is_wp_error( $top_posts ) && $top_posts->have_posts() ) {
            wp_cache_set( $cache_key, $top_posts->posts, 'top_posts' );
        }
    }

    return $top_posts;
}

// Prime cache when comment count changes
add_action( 'wp_update_comment_count', function( $post_id, $new, $old ) {
    prefix_get_top_commented_posts( true );
}, 10, 3 );
```

### Cache Remote Requests

Always cache third-party API requests.

```php
function prefix_get_remote_data() {
    $cache_key = 'prefix_remote_data';
    $data = wp_cache_get( $cache_key );

    if ( false === $data ) {
        $response = wp_remote_get( 'https://api.example.com/data' );
        $data = wp_remote_retrieve_body( $response );
        wp_cache_set( $cache_key, $data, '', HOUR_IN_SECONDS );
    }

    return $data;
}
```

### Use isset() Instead of in_array()

`isset()` is O(1), `in_array()` is O(n).

#### ✅ Do: Use isset() with array keys

```php
$allowed = array(
    'admin' => true,
    'editor' => true,
    'author' => true,
);

if ( isset( $allowed[ $user_role ] ) ) {
    // User has permission
}
```

#### ❌ Don't: Use in_array() for large arrays

```php
$allowed = array( 'admin', 'editor', 'author' );

// Slow for large arrays
if ( in_array( $user_role, $allowed ) ) {
    // ...
}
```

---

## Design Patterns

### Namespacing

Always namespace PHP code outside of WordPress template files.

```php
<?php
namespace Company\Project_Name\Feature_Name;

function do_something() {
    // Your code
}
```

Use declarations for external classes:

```php
<?php
namespace Company\Project_Name;
use Company\Common_Library\TwitterAPI;

function post_to_twitter() {
    $api = new TwitterAPI();
    // ...
}
```

### Object Design

Objects should be well-defined, atomic, and fully documented.

```php
<?php
/**
 * Video post wrapper.
 *
 * Wraps WordPress posts and YouTube meta information.
 *
 * @package    ClientTheme
 * @subpackage Content
 */
class PrefixVideo {

    /**
     * WordPress post object.
     *
     * @var WP_Post
     */
    protected $post;

    /**
     * Constructor.
     *
     * @param int|WP_Post $post Post ID or object.
     * @throws Exception If post is invalid.
     */
    public function __construct( $post = null ) {
        if ( null === $post ) {
            throw new Exception( 'Invalid post supplied' );
        }

        $this->post = get_post( $post );
    }
}
```

### Visibility

- Use `public` for public properties/methods
- Use `protected` for internal use (not `private` unless well-documented)
- Use Singletons smartly (see below)
- Register hooks in __construct(), but defer execution to appropriate WordPress hooks

### Smart Singleton Usage

Singletons can be useful when implemented properly. Use them for:

- **Main plugin/application classes** that coordinate initialization
- **Service classes** that manage global state or resources
- **Factory classes** that create and manage object instances

#### ✅ Do: Implement Singleton Properly

```php
<?php
/**
 * Main plugin class.
 *
 * @package MyPlugin
 */
class MyPlugin {

    /**
     * The single instance of the class.
     *
     * @var MyPlugin|null
     */
    protected static $instance = null;

    /**
     * Main instance. Ensures only one instance is loaded or can be loaded.
     *
     * @return MyPlugin Main instance.
     */
    public static function instance() {
        if ( is_null( self::$instance ) ) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    /**
     * Constructor.
     *
     * Set up class properties and hooks.
     */
    private function __construct() {
        $this->init_hooks();
    }

    /**
     * Prevent cloning.
     */
    private function __clone() {}

    /**
     * Prevent unserializing.
     */
    public function __wakeup() {
        throw new \Exception( 'Cannot unserialize singleton' );
    }

    /**
     * Initialize hooks.
     */
    private function init_hooks() {
        add_action( 'init', array( $this, 'init' ) );
        add_action( 'admin_init', array( $this, 'admin_init' ) );
    }

    /**
     * Initialize plugin.
     */
    public function init() {
        // Initialization logic
    }

    /**
     * Initialize admin.
     */
    public function admin_init() {
        // Admin initialization logic
    }
}

// Initialize plugin
MyPlugin::instance();
```

#### When NOT to Use Singletons:

- For simple utility classes (use static methods instead)
- For value objects or data structures
- For classes that should have multiple instances
- When dependency injection would be clearer

### Decouple Plugins and Themes

Use `add_theme_support()` and `current_theme_supports()` to decouple functionality.

**In Theme:**
```php
add_theme_support( 'custom-js-feature' );
```

**In Plugin:**
```php
if ( current_theme_supports( 'custom-js-feature' ) ) {
    // Safe to add feature
    wp_enqueue_script( 'custom-feature', plugins_url( 'js/feature.js', __FILE__ ) );
}
```

---

## Security

### Input Validation and Sanitization

Always validate and sanitize user input before storing in database.

#### ✅ Do: Validate and sanitize

```php
if ( ! empty( $_POST['user_id'] ) ) {
    // Validate it's an integer
    if ( absint( $_POST['user_id'] ) === $_POST['user_id'] ) {
        update_post_meta( $post_id, 'user_id', absint( $_POST['user_id'] ) );
    }
}

if ( ! empty( $_POST['heading'] ) ) {
    update_option( 'site_heading', sanitize_text_field( $_POST['heading'] ) );
}
```

### SQL Query Preparation

Always prepare SQL queries with `$wpdb->prepare()`.

```php
global $wpdb;

$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT id, name FROM $wpdb->posts WHERE ID = %d",
        absint( $post_id )
    )
);
```

### Output Escaping (Late Escaping)

Escape output just before display.

```php
<div class="user-content">
    <?php echo esc_html( get_post_meta( $post_id, 'user_text', true ) ); ?>
</div>

<a href="mailto:<?php echo sanitize_email( $user_email ); ?>">
    Email
</a>

<div class="<?php echo esc_attr( $custom_class ); ?>">
    Content
</div>

<script>
const config = <?php echo wp_json_encode( $config_array ); ?>;
</script>
```

**Never use `esc_js()` - it's for inline attributes only. Use `wp_json_encode()` for JavaScript.**

### Nonces for State Changes

Always use nonces for update/delete operations.

```php
// Generate nonce in form
<form method="post">
    <?php wp_nonce_field( 'save_settings_action' ); ?>
    <input type="text" name="setting_value" />
    <button type="submit">Save</button>
</form>

// Verify nonce on submission
if ( ! empty( $_POST['_wpnonce'] )
     && wp_verify_nonce( $_POST['_wpnonce'], 'save_settings_action' ) ) {
    // Process form
}
```

---

## Internationalization

### Use Translation Functions

Always internationalize strings with a unique text domain.

#### ✅ Do: Use literal strings

```php
$message = sprintf(
    __( '%d minutes remaining', 'plugin-domain' ),
    $minutes
);

$message = sprintf(
    _n( '%d item', '%d items', $count, 'plugin-domain' ),
    $count
);
```

#### ❌ Don't: Use variables

```php
// Translation tools can't parse this
$string = __( "$minutes minutes", 'plugin-domain' );

// Or this
define( 'MESSAGE', 'Minutes left' );
__( MESSAGE, 'plugin-domain' );
```

### Escape Translated Strings

Use combined translation/escaping functions.

```php
<h1><?php esc_html_e( 'Page Title', 'plugin-domain' ); ?></h1>

<input value="<?php esc_attr_e( 'Default Value', 'plugin-domain' ); ?>" />

$title = esc_html__( 'Title', 'plugin-domain' );
```

---

## Code Style & Documentation

Follow [WordPress PHP Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/).

### Document Everything

Provide verbose docblocks explaining WHY code exists and WHAT it does.

```php
<?php
/**
 * Mark specific post meta keys as protected.
 *
 * Post meta can be public or protected. Meta holding internal or
 * read-only data should be protected via an underscore prefix or
 * this filter. This prevents exposure through Custom Fields or REST API.
 *
 * @param bool   $protected   Whether key is protected.
 * @param string $meta_key    The meta key being checked.
 * @return bool               Modified protected status.
 */
function protect_internal_meta( $protected, $meta_key ) {
    $protected_keys = array(
        'internal_id',
        'api_sync_data',
        'calculated_score',
    );

    if ( in_array( $meta_key, $protected_keys, true ) ) {
        $protected = true;
    }

    return $protected;
}
add_filter( 'is_protected_meta', 'protect_internal_meta', 10, 2 );
```

### Naming Conventions

Follow WordPress naming conventions for consistency and readability.

#### Class Names: PascalCase

Use PascalCase (UpperCamelCase) for class names - capitalize each word with no underscores.

```php
<?php
// ✅ Do: Use PascalCase for class names
class MyPluginSettings {
    // ...
}

class VideoPostHandler {
    // ...
}

class TwitterAPIClient {
    // ...
}

// ❌ Don't: Use underscores in class names
class My_Plugin_Settings {  // Wrong
    // ...
}
```

#### Functions and Variables: snake_case

Use lowercase with underscores (snake_case) for functions, methods, and variables.

```php
<?php
// Functions
function prefix_get_user_data() {
    // ...
}

// Variables
$user_name = 'John';
$post_count = 10;
```

#### Constants: UPPERCASE

Use uppercase with underscores for constants.

```php
<?php
define( 'PLUGIN_VERSION', '1.0.0' );
define( 'API_ENDPOINT', 'https://api.example.com' );
```

---

## What to Avoid

### Don't Use Heredoc/Nowdoc

They prevent late escaping.

#### ❌ Don't:
```php
$html = <<<HTML
<div class="{$class}">Content</div>
HTML;
```

#### ✅ Do:
```php
$html = '<div class="' . esc_attr( $class ) . '">Content</div>';

// Or use get_template_part() for complex HTML
```

### Avoid Sessions

Sessions don't scale and conflict with caching. Use cookies or client-side storage instead.

If sessions are absolutely necessary, use Redis/Memcache session handlers, never database.

---

## Quick Reference

### Data Storage

- **Options**: Settings, small key-value data (keep under 500 rows, avoid autoload for large data)
- **Post Meta**: Data specific to individual posts
- **Taxonomies**: Groupings/classifications across multiple posts
- **Custom Post Types**: Variable amounts of structured data
- **Object Cache**: Temporary data, query results

### Query Optimization Checklist

- ✅ Use `WP_Query` over `get_posts()` or raw `$wpdb`
- ✅ Set `no_found_rows => true` when not paginating
- ✅ Disable unused caches (`update_post_meta_cache`, `update_post_term_cache`)
- ✅ Never use `posts_per_page => -1`
- ✅ Avoid `post__not_in` - filter in PHP instead
- ✅ Prefer taxonomies over post meta for queries
- ✅ Cache query results
- ✅ Prime caches proactively on content updates

### Security Checklist

- ✅ Validate all input
- ✅ Sanitize before storing in database
- ✅ Escape just before output (late escaping)
- ✅ Use nonces for state-changing operations
- ✅ Prepare all SQL queries
- ✅ Never trust user input
- ✅ Use `wp_json_encode()` for JavaScript data

---

## Further Reading

- [WordPress PHP Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/)
- [WordPress Plugin Handbook - Security](https://developer.wordpress.org/plugins/security/)
- [Data Sanitization/Escaping](https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/)
- [WordPress VIP Code Review Guidelines](https://docs.wpvip.com/technical-references/code-review/)
