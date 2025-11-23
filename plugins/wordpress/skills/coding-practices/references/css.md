# CSS Best Practices

**Reference Category:** Frontend Development
**Applies To:** CSS3, Sass, PostCSS, Responsive Design
**Related:** Performance, Accessibility, Mobile-First Design

## Purpose

This reference provides CSS best practices for building performant, accessible, and maintainable stylesheets. It covers responsive design, syntax standards, accessibility considerations, and performance optimization.

## When to Use This Reference

- When writing CSS for WordPress themes/plugins
- When implementing responsive, mobile-first designs
- When optimizing CSS performance
- When ensuring accessible animations and color contrast
- When structuring Sass/PostCSS projects

## Key Principles

1. **Mobile First** - Build from smallest viewport up using min-width media queries
2. **Semantic & Maintainable** - Use clear class names, avoid deep nesting
3. **Accessible** - Consider color contrast, reduced motion, keyboard navigation
4. **Performant** - Minimize CSS, optimize selectors, leverage inheritance
5. **Consistent** - Follow syntax standards and naming conventions

---

## Philosophy

CSS should enhance content, not bury it. We prioritize:
- Content and user experience over flashy effects
- Maintainability over cleverness
- Accessibility and performance over bleeding-edge features

---

## Accessibility

### Respect prefers-reduced-motion

Not all users tolerate animations - some trigger vestibular disorders, epilepsy, or migraines.

```css
.animation {
    animation: vibrate 0.3s linear infinite both;
}

@media (prefers-reduced-motion: reduce) {
    .animation {
        animation: none;
    }
}
```

### Color Contrast

Ensure text meets WCAG contrast requirements:
- **Level AA:** 4.5:1 for normal text, 3:1 for large text
- **Level AAA:** 7:1 for normal text, 4.5:1 for large text

Tools:
- [Contrast Ratio Checker](https://leaverou.github.io/contrast-ratio/)
- [Tanaguru Contrast Finder](http://contrast-finder.tanaguru.com/)

---

## Responsive Design (Mobile First)

### Use Min-Width Media Queries

Build from mobile up - add complexity at larger breakpoints.

#### ✅ Do: Mobile first

```scss
.component {
    // Mobile styles (base)
    display: block;
    padding: 1rem;

    @media (min-width: 768px) {
        // Tablet
        display: flex;
        padding: 2rem;
    }

    @media (min-width: 1024px) {
        // Desktop
        max-width: 1200px;
        margin: 0 auto;
    }
}
```

#### ❌ Don't: Desktop first or mixed directions

```scss
// WRONG - max-width requires overrides
.component {
    max-width: 1200px; // Desktop

    @media (max-width: 1024px) {
        max-width: 100%; // Override
    }
}
```

### Breakpoints

Use variables/mixins for consistency.

```scss
// Variables
$breakpoint-tablet: 768px;
$breakpoint-desktop: 1024px;
$breakpoint-wide: 1280px;

// Or Sass mixin
@mixin respond-to($breakpoint) {
    @media (min-width: $breakpoint) {
        @content;
    }
}

// Usage
.header {
    font-size: 1.5rem;

    @include respond-to($breakpoint-desktop) {
        font-size: 2rem;
    }
}
```

### Media Query Placement

Place media queries inside component styles, not in separate files.

```scss
// ✅ DO THIS - media query with component
.card {
    padding: 1rem;
    background: white;

    @media (min-width: 768px) {
        padding: 2rem;
    }
}

.button {
    font-size: 14px;

    @media (min-width: 768px) {
        font-size: 16px;
    }
}

// ❌ DON'T DO THIS - separate media query file
// _768px.scss
.card {
    padding: 2rem;
}
.button {
    font-size: 16px;
}
```

---

## Syntax and Formatting

### Basic Syntax Rules

```css
/* ✅ CORRECT */
.selector-one,
.selector-two,
.selector-three {
    display: block;
    color: #333;
    background-color: #fff;
    box-shadow: 0 1px 5px #000, 1px 2px 5px #ccc;
}

/* ❌ AVOID */
.selector-one, .selector-two {
display:block;color:#333;background-color:#fff}
```

**Rules:**
- One selector per line
- One declaration per line
- Space before opening brace
- Space after colon
- Space after comma in values
- Lowercase values (except font names)
- No units for zero values
- End all declarations with semicolon
- Use double quotes

### Declaration Ordering

Order declarations alphabetically or by type. Be consistent.

**By Type:**
1. Positioning
2. Box Model
3. Typography
4. Visual
5. Misc

```css
.element {
    /* Positioning */
    position: absolute;
    top: 0;
    left: 0;
    z-index: 10;

    /* Box Model */
    display: flex;
    width: 100%;
    padding: 1rem;
    margin: 0 auto;

    /* Typography */
    font-family: Arial, sans-serif;
    font-size: 16px;
    line-height: 1.5;

    /* Visual */
    background-color: #fff;
    border: 1px solid #ccc;
    color: #333;

    /* Misc */
    cursor: pointer;
}
```

### Nesting (Sass/PostCSS)

Nest only when necessary - avoid deep nesting.

**When to nest:**
- Pseudo-classes
- Pseudo-elements
- Component states
- Media queries

```scss
.button {
    padding: 1rem;
    background: blue;

    // Pseudo-classes
    &:hover {
        background: darkblue;
    }

    // Pseudo-elements
    &::before {
        content: '';
    }

    // State classes
    &.is-active {
        background: green;
    }

    // Media queries
    @media (min-width: 768px) {
        padding: 1.5rem;
    }
}
```

#### ❌ Avoid excessive nesting

```scss
// WRONG - too deep!
.header {
    .nav {
        .menu {
            .item {
                .link {
                    color: blue; // 5 levels deep!
                }
            }
        }
    }
}

// CORRECT - flat structure
.header-nav-link {
    color: blue;
}
```

### Naming Conventions

Use lowercase with hyphens. Consider BEM for complex components.

```css
/* Simple naming */
.button-primary { }
.card-header { }
.nav-menu { }

/* BEM naming */
.block { }
.block__element { }
.block--modifier { }

/* Example */
.card { }
.card__header { }
.card__body { }
.card--featured { }
```

**Avoid:**
- camelCase
- Advertisement/Ad class names (blocked by adblockers)
- Generic names like `.left`, `.red`

---

## Performance

### Minimize Network Requests

```scss
// Concatenate CSS files
// Minify stylesheets
// Use GZIP compression
// Encode small images as data URIs (sparingly)
```

### CSS Specificity

Keep specificity low for easier overrides.

#### ✅ Do: Use low-specificity classes

```css
/* Specificity: 0,0,1,0 */
.button {
    background: blue;
}
```

#### ❌ Don't: Over-specify

```css
/* Specificity: 0,1,3,4 - too high! */
div div header#header div ul.nav-menu li a.button {
    background: blue;
}
```

### Efficient Selectors

```css
/* ✅ Efficient */
.nav-item { }
.button { }

/* ❌ Inefficient */
header nav ul li a { }
* { }
[type="text"] { }
```

### Use Inheritance

Don't repeat inherited properties.

```css
/* ❌ WRONG - repeating */
.sibling-1 {
    font-family: Arial, sans-serif;
}
.sibling-2 {
    font-family: Arial, sans-serif;
}

/* ✅ CORRECT - inherit */
.parent {
    font-family: Arial, sans-serif;
}
```

### CSS Over Assets

Use CSS instead of images when possible.

```css
/* Gradients instead of images */
.gradient {
    background: linear-gradient(to bottom, #1e5799 0%, #7db9e8 100%);
}

/* CSS shapes instead of images */
.triangle {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid blue;
}
```

### Animations

Use CSS for simple state changes, JavaScript for complex animations.

**Optimize animations:**
- Use `transform` and `opacity` only (GPU-accelerated)
- Avoid animating `left`, `top`, `width`, `height`, `margin`

```css
/* ✅ GOOD - GPU accelerated */
.element {
    transform: translateX(0);
    transition: transform 0.3s ease;
}

.element:hover {
    transform: translateX(10px);
}

/* ❌ AVOID - forces repaints */
.element {
    left: 0;
    transition: left 0.3s ease;
}

.element:hover {
    left: 10px;
}
```

---

## Documentation

### Commenting

Use clear comments to section and explain CSS.

```css
/**
 * Header Component
 *
 * Main site header with logo and navigation
 */
.site-header {
    position: sticky;
    top: 0;
}

/* Select list items 4 to 8, included */
li:nth-child(n+4):nth-child(-n+8) {
    color: red;
}
```

---

## Common Patterns

### Utility Classes

```css
/* Display */
.hide { display: none; }
.show { display: block; }

/* Spacing */
.mt-1 { margin-top: 0.5rem; }
.mt-2 { margin-top: 1rem; }
.mb-1 { margin-bottom: 0.5rem; }

/* Text */
.text-center { text-align: center; }
.text-right { text-align: right; }
```

### Component Structure

```scss
.component {
    // Base styles
    display: block;

    // Elements
    &__header {
        font-size: 1.5rem;
    }

    &__body {
        padding: 1rem;
    }

    // Modifiers
    &--featured {
        border: 2px solid gold;
    }

    // States
    &.is-active {
        background: lightblue;
    }

    // Responsive
    @media (min-width: 768px) {
        display: flex;
    }
}
```

---

## Quick Reference Checklist

### Responsive Design
- ✅ Use mobile-first approach
- ✅ Use min-width media queries consistently
- ✅ Place media queries inside components
- ✅ Use variables for breakpoints
- ✅ Test on real devices

### Performance
- ✅ Keep specificity low
- ✅ Use efficient selectors (classes)
- ✅ Leverage inheritance
- ✅ Animate transform/opacity only
- ✅ Minimize and concatenate CSS

### Accessibility
- ✅ Respect prefers-reduced-motion
- ✅ Ensure color contrast meets WCAG
- ✅ Don't rely on color alone for meaning
- ✅ Test with keyboard navigation
- ✅ Provide focus indicators

### Code Quality
- ✅ Follow syntax standards
- ✅ Use consistent naming
- ✅ Avoid deep nesting
- ✅ Comment complex selectors
- ✅ Use linting (stylelint)

---

## Tools

### CSS Reset
- Use [Normalize.css](https://necolas.github.io/normalize.css/)

### Linting
- Use [Stylelint](https://stylelint.io/)

### Testing
- [WAVE](http://wave.webaim.org/) - Accessibility checker
- [Lighthouse](https://developers.google.com/web/tools/lighthouse) - Performance audit

---

## Further Reading

- [CSS Guidelines](https://cssguidelin.es/)
- [CSS Tricks](https://css-tricks.com/)
- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [Can I Use](https://caniuse.com/) - Browser support tables
