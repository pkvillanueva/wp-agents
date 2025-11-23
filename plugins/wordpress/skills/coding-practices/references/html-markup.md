# HTML Markup Best Practices

**Reference Category:** Frontend Development
**Applies To:** HTML5, Semantic HTML, WordPress Templates
**Related:** Accessibility, SEO, Progressive Enhancement

## Purpose

This reference provides HTML markup best practices for creating accessible, semantic, and SEO-friendly web pages. It covers semantic elements, accessibility standards, forms, media, and SVG usage.

## When to Use This Reference

- When writing WordPress theme templates
- When implementing accessible forms and navigation
- When adding ARIA attributes for screen readers
- When optimizing markup for SEO (Schema.org)
- When working with SVG graphics

## Key Principles

1. **Semantic First** - Use meaningful HTML elements that describe content
2. **Accessibility Always** - Meet WCAG 2.1 Level AA standards minimum
3. **Progressive Enhancement** - Build baseline experience, enhance upward
4. **Minimal & Valid** - Use least markup necessary, validate with W3C
5. **Optimize for Readers** - Code for both humans and browsers

---

## Philosophy

Markup defines the structure and semantics of content - NOT the presentation. CSS handles appearance, JavaScript handles behavior. Keeping these concerns separated improves maintainability and accessibility.

### Progressive Enhancement

Build websites that work for everyone, with enhancements for capable browsers:
1. Start with semantic HTML (baseline)
2. Add CSS for visual enhancement
3. Add JavaScript for interactive enhancement

---

## Accessibility Standards

### WCAG Compliance Levels

**Level A (Minimum):**
- Basic accessibility requirements
- Required by law in many jurisdictions

**Level AA (Recommended):**
- Addresses major accessibility barriers
- Recommended baseline for most projects
- Includes color contrast requirements

**Level AAA (Enhanced):**
- Highest level of accessibility
- Not always achievable for all content

### Target Compliance

- **Minimum:** WCAG 2.1 Level A
- **Recommended:** WCAG 2.1 Level AA
- **Global projects:** Level AA (EU compliance)

---

## Semantic HTML

### Use Semantic Elements

#### ✅ Do: Use meaningful elements

```html
<header role="banner">
    <nav role="navigation">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
        </ul>
    </nav>
</header>

<main role="main">
    <article>
        <h1>Article Title</h1>
        <p>Article content...</p>
    </article>
</main>

<aside>
    <h2>Related Content</h2>
</aside>

<footer role="contentinfo">
    <p>&copy; 2025 Company Name</p>
</footer>
```

#### ❌ Don't: Use divs for everything

```html
<!-- WRONG - no semantic meaning -->
<div class="header">
    <div class="nav">
        <div class="menu-item">Home</div>
    </div>
</div>

<div class="main-content">
    <div class="article">Content</div>
</div>
```

### Semantic Elements Reference

| Element | Purpose |
|---------|---------|
| `<header>` | Site or section header |
| `<nav>` | Navigation links |
| `<main>` | Main content (one per page) |
| `<article>` | Self-contained content |
| `<section>` | Thematic grouping |
| `<aside>` | Tangential content |
| `<footer>` | Site or section footer |
| `<figure>` + `<figcaption>` | Images with captions |

---

## Accessibility Features

### ARIA Landmark Roles

Add ARIA roles for better screen reader navigation.

```html
<header role="banner">
    <!-- Site header -->
</header>

<nav role="navigation">
    <!-- Main navigation -->
</nav>

<main role="main">
    <!-- Primary content -->
</main>

<aside role="complementary">
    <!-- Sidebar content -->
</aside>

<footer role="contentinfo">
    <!-- Site footer -->
</footer>
```

### Skip Links

Allow keyboard users to skip to main content.

```html
<body>
    <a class="skip-link screen-reader-text" href="#main">
        Skip to content
    </a>

    <header>...</header>

    <main id="main" tabindex="-1">
        <!-- Main content -->
    </main>
</body>
```

```css
/* Hide visually, keep for screen readers */
.screen-reader-text {
    position: absolute;
    left: -9999px;
}

.screen-reader-text:focus {
    position: static;
    left: auto;
    /* Style visible focus state */
}
```

### ARIA States and Properties

Use ARIA to describe component states.

```html
<!-- Tabs example -->
<ul role="tablist">
    <li role="presentation">
        <a href="#panel-1" role="tab" aria-selected="true" id="tab-1">
            Panel 1
        </a>
    </li>
    <li role="presentation">
        <a href="#panel-2" role="tab" aria-selected="false" id="tab-2">
            Panel 2
        </a>
    </li>
</ul>

<div role="tabpanel" aria-hidden="false" aria-labelledby="tab-1" id="panel-1">
    <h2>Panel 1 Content</h2>
</div>

<div role="tabpanel" aria-hidden="true" aria-labelledby="tab-2" id="panel-2">
    <h2>Panel 2 Content</h2>
</div>
```

**Important:** Add `aria-hidden` dynamically with JavaScript, not in HTML. If JS fails to load, content shouldn't be hidden from screen readers.

---

## Forms

### Accessible Form Structure

Every input needs a label, use fieldsets for grouping.

```html
<form method="post" action="/submit">
    <fieldset>
        <legend>Personal Information</legend>

        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required />

        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required />
    </fieldset>

    <fieldset>
        <legend>Contact Preferences</legend>

        <input type="checkbox" id="newsletter" name="newsletter" />
        <label for="newsletter">Subscribe to newsletter</label>

        <input type="checkbox" id="updates" name="updates" />
        <label for="updates">Receive product updates</label>
    </fieldset>

    <button type="submit">Submit</button>
</form>
```

### Form Validation and Error Messages

```html
<label for="username">Username:</label>
<input
    type="text"
    id="username"
    name="username"
    aria-required="true"
    aria-invalid="true"
    aria-describedby="username-error"
/>
<span id="username-error" role="alert">
    Username must be at least 5 characters
</span>
```

---

## Media

### Images

Always provide alternative text.

```html
<!-- Decorative images -->
<img src="decoration.png" alt="" />

<!-- Informative images -->
<img src="chart.png" alt="Sales increased 40% in Q4 2024" />

<!-- Images with captions -->
<figure>
    <img src="photo.jpg" alt="Mountain landscape at sunset" />
    <figcaption>Sunset over the Rocky Mountains</figcaption>
</figure>
```

### Video

Provide captions and transcripts.

```html
<video controls>
    <source src="video.mp4" type="video/mp4" />
    <track kind="captions" src="captions.vtt" srclang="en" label="English" />
    <p>Your browser doesn't support video. <a href="video.mp4">Download video</a></p>
</video>

<!-- Include transcript below video -->
<details>
    <summary>Video Transcript</summary>
    <p>Transcript text here...</p>
</details>
```

### Audio

```html
<audio controls>
    <source src="podcast.mp3" type="audio/mpeg" />
    <p>Your browser doesn't support audio. <a href="podcast.mp3">Download audio</a></p>
</audio>
```

**Accessibility Requirements:**
- Captions for video with dialogue
- Transcripts for audio/video content
- Don't autoplay with sound (WCAG 1.4.2)

---

## SVG

### SVG for Scalable Graphics

Use SVG for logos, icons, and illustrations.

**Benefits:**
- Scalable to any size
- Small file size
- CSS-styleable
- Animatable

**Limitations:**
- Not for photos (use JPG/PNG)
- Security risk if from untrusted sources
- WordPress blocks SVG uploads by default

### Inline SVG Accessibility

#### Decorative SVG

```html
<svg aria-hidden="true" focusable="false">
    <use xlink:href="#icon-star"></use>
</svg>
```

#### Meaningful SVG

```html
<svg role="img" aria-labelledby="title-id desc-id">
    <title id="title-id">Chart Title</title>
    <desc id="desc-id">Bar chart showing sales data for 2024</desc>
    <!-- SVG content -->
</svg>
```

#### SVG with Links

```html
<a href="/twitter" aria-label="Follow us on Twitter">
    <svg><use xlink:href="#icon-twitter"></use></svg>
</a>
```

### SVG Sprites

Use SVG sprites for icon systems.

```html
<!-- svg-defs.svg (include once) -->
<svg style="display: none;">
    <symbol id="icon-home" viewBox="0 0 24 24">
        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
    </symbol>
</svg>

<!-- Usage -->
<svg class="icon">
    <use xlink:href="#icon-home"></use>
</svg>
```

### Optimize SVG

Always run SVG through optimization tools:
- [SVGOMG](https://jakearchibald.github.io/svgomg/)
- Or use build tools (gulp-svgmin, etc.)

---

## Schema.org Markup

Add structured data for search engines.

### JSON-LD Format (Recommended)

```html
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Article Headline",
    "author": {
        "@type": "Person",
        "name": "Author Name"
    },
    "datePublished": "2024-01-15",
    "image": "https://example.com/image.jpg"
}
</script>
```

### Common Schema Types

- **Article** - Blog posts, news articles
- **LocalBusiness** - Stores, restaurants
- **Product** - E-commerce items
- **Event** - Conferences, concerts
- **Recipe** - Cooking instructions
- **FAQPage** - FAQ content

Validate with [Google Structured Data Testing Tool](https://search.google.com/structured-data/testing-tool).

---

## Best Practices

### Minimal & Valid Markup

Use the least markup necessary.

```html
<!-- ❌ Excessive markup -->
<div class="wrapper">
    <div class="container">
        <div class="content-wrapper">
            <div class="text">Hello</div>
        </div>
    </div>
</div>

<!-- ✅ Minimal markup -->
<p class="text">Hello</p>
```

Validate with [W3C Validator](https://validator.w3.org/).

### Optimize Readability

Use proper indentation and structure.

#### ❌ Bad

```php
<ul>
<?php
foreach( $items as $item ) {
  echo '<li>' . esc_html( $item ) . '</li>';
}
?>
</ul>
```

#### ✅ Good

```php
<ul>
    <?php foreach ( $items as $item ) : ?>
        <li><?php echo esc_html( $item ); ?></li>
    <?php endforeach; ?>
</ul>
```

### Classes for CSS, IDs for JavaScript

```html
<button class="btn btn-primary" id="js-submit-button">
    Submit
</button>
```

Prefix JavaScript IDs with `js-` to indicate usage.

---

## Quick Reference Checklist

### Semantic HTML
- ✅ Use semantic elements (`<header>`, `<nav>`, `<main>`, etc.)
- ✅ One `<main>` per page
- ✅ Logical heading hierarchy (`<h1>` to `<h6>`)
- ✅ Use `<article>` for self-contained content
- ✅ Use `<figure>` with `<figcaption>` for images

### Accessibility
- ✅ Add ARIA landmark roles
- ✅ Include skip links
- ✅ Every form input has a `<label>`
- ✅ Use `alt` text for images
- ✅ Provide captions for video
- ✅ Meet WCAG 2.1 Level AA minimum
- ✅ Test with keyboard navigation
- ✅ Test with screen readers

### Forms
- ✅ Use `<label for="id">` with inputs
- ✅ Group related inputs with `<fieldset>`
- ✅ Use appropriate input types
- ✅ Add `aria-required`, `aria-invalid` for validation
- ✅ Provide clear error messages

### Media
- ✅ Always include `alt` attributes
- ✅ Use empty alt (`alt=""`) for decorative images
- ✅ Provide transcripts for audio/video
- ✅ Don't autoplay media with sound
- ✅ Optimize images for web

### SVG
- ✅ Use `aria-hidden="true"` for decorative SVG
- ✅ Add `<title>` and `<desc>` for meaningful SVG
- ✅ Use `aria-label` for linked SVG
- ✅ Optimize with SVGOMG
- ✅ Use SVG sprites for icon systems

---

## Tools

### Validation
- [W3C HTML Validator](https://validator.w3.org/)
- [Google Structured Data Testing Tool](https://search.google.com/structured-data/testing-tool/)

### Accessibility Testing
- [WAVE](http://wave.webaim.org/)
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [Pa11y](https://github.com/pa11y/pa11y)

### Screen Readers
- VoiceOver (macOS/iOS)
- NVDA (Windows - free)
- JAWS (Windows)

---

## Further Reading

- [MDN HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WebAIM](https://webaim.org/) - Accessibility resources
- [Schema.org](https://schema.org/) - Structured data vocabulary
- [Accessible SVGs](https://css-tricks.com/accessible-svgs/)
