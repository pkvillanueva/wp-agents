# JavaScript Best Practices

**Reference Category:** Frontend Development
**Applies To:** ES6+, Modern JavaScript, WordPress Development
**Related:** React, TypeScript, Performance Optimization

## Purpose

This reference provides modern JavaScript best practices for WordPress and web development. It covers ES6+ features, design patterns, performance optimization, security, and client-side data handling.

## When to Use This Reference

- When writing modern JavaScript for WordPress themes/plugins
- When implementing client-side data requests (fetch, AJAX)
- When optimizing JavaScript performance
- When structuring modular JavaScript code
- When securing client-side JavaScript

## Key Principles

1. **Use Modern JavaScript** - Leverage ES6+ features with proper transpilation
2. **Modular Code** - Use ES6 modules and avoid global namespace pollution
3. **Performance Matters** - Cache selectors, debounce/throttle, lazy load
4. **Security First** - Avoid XSS, sanitize dynamic content, use CSP
5. **Clean Code** - Follow linting rules, document complex logic

---

## Modern JavaScript Patterns

### Use ES6 Classes

```javascript
class Component {
    constructor(el) {
        this.el = el;
        this.init();
    }

    init() {
        console.log('Component initialized');
    }
}

// Usage
const myComponent = new Component(document.querySelector('.component'));
```

**When to use classes:**
- Creating discrete components
- Components that inherit behaviors
- Components that function as single units

**When NOT to use classes:**
- Simple utility functions
- One-off operations

### Arrow Functions

Use arrow functions for cleaner syntax and lexical `this` binding.

#### ✅ Do: Use arrow functions appropriately

```javascript
// Good for callbacks
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);

// Good for preserving context
class Timer {
    constructor() {
        this.seconds = 0;
        setInterval(() => {
            this.seconds++; // `this` refers to Timer instance
        }, 1000);
    }
}

// Multi-line with explicit return
const processData = (data) => {
    const processed = data.filter(item => item.active);
    return processed.map(item => item.value);
};
```

#### ❌ Don't: Use when you need dynamic `this`

```javascript
// WRONG - arrow function doesn't rebind `this`
button.addEventListener('click', () => {
    this.classList.toggle('active'); // `this` is not the button!
});

// CORRECT - traditional function
button.addEventListener('click', function() {
    this.classList.toggle('active'); // `this` is the button
});
```

### Template Literals

Use template literals for string concatenation.

```javascript
// ❌ Old way
const message = 'Hello, ' + firstName + ' ' + lastName + '!';

// ✅ Modern way
const message = `Hello, ${firstName} ${lastName}!`;

// Multiline strings
const html = `
    <div class="card">
        <h2>${title}</h2>
        <p>${description}</p>
    </div>
`;
```

### Destructuring

Extract values from arrays and objects cleanly.

```javascript
// Array destructuring
const [a, b, c, d] = [1, 2, 3, 4];

// Object destructuring
const { name, email, role } = user;

// With default values
const { theme = 'light', language = 'en' } = settings;

// In function parameters
function greet({ name, age }) {
    return `Hello ${name}, you are ${age} years old`;
}
```

### ES6 Modules

Always use import/export for modular code.

```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// app.js
import { add, subtract } from './math.js';

// Import specific methods from libraries
import map from 'lodash/map';
import debounce from 'lodash/debounce';
```

#### ❌ Avoid Barrel Files

Don't create index.js files that re-export everything.

```javascript
// ❌ DON'T DO THIS - barrel file
// utils/index.js
export * from './date';
export * from './time';
export * from './cache';

// ❌ Imports everything
import { formatDate } from './utils';

// ✅ DO THIS - direct imports
import { formatDate } from './utils/date';
```

---

## Design Patterns

### Module Pattern

Create self-contained modules.

```javascript
// datastructure.js
const data = {}; // Private

const process = (value) => {
    // Private function
    return value.toUpperCase();
};

// Public API
export const getData = (field) => {
    return process(data[field]);
};

export const addData = (field, value) => {
    data[field] = value;
};
```

### Factory Pattern Over Classes (When Appropriate)

```javascript
// Option 1: Factory function
export function createModal(selector, options = {}) {
    const element = document.querySelector(selector);

    return {
        open() {
            element.classList.add('is-open');
        },
        close() {
            element.classList.remove('is-open');
        }
    };
}

// Usage
const modal = createModal('#myModal');
modal.open();
```

### Don't Pollute Window Object

Use modules and closures to keep variables private.

```javascript
// ❌ WRONG - pollutes global scope
for (var i = 0; i < 10; i++) {
    // ...
}
console.log(window.i); // 9 - leaked!

// ✅ CORRECT - use modules
(function() {
    for (let i = 0; i < 10; i++) {
        // ...
    }
})();
console.log(typeof window.i); // 'undefined'
```

---

## Security

### Avoid innerHTML (XSS Risk)

#### ❌ Don't: Use innerHTML with user content

```javascript
// DANGEROUS - XSS vulnerability!
const userInput = '<img src=x onerror="alert(\'hacked\')">';
element.innerHTML = userInput;
```

#### ✅ Do: Use textContent or createElement

```javascript
// Safe - doesn't parse HTML
element.textContent = userInput;

// Or create elements via DOM API
const div = document.createElement('div');
const link = document.createElement('a');
link.href = url;
link.textContent = linkText;
div.appendChild(link);
```

#### When innerHTML Is Necessary

Use DOMPurify or parse/sanitize first.

```javascript
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(dirtyHTML);
element.innerHTML = clean;
```

---

## Performance

### Only Load What You Need

#### ✅ Do: Import specific functions

```javascript
import map from 'lodash/map';
import debounce from 'lodash/debounce';
```

#### ❌ Don't: Import entire libraries

```javascript
import _ from 'lodash'; // Loads all of Lodash!
```

### Cache DOM Selections

```javascript
// ❌ WRONG - reselects every click
hideButton.addEventListener('click', () => {
    const menu = document.getElementById('menu');
    menu.style.display = 'none';
});

// ✅ CORRECT - cache the selection
const menu = document.getElementById('menu');
const hideButton = document.querySelector('.hide-button');

hideButton.addEventListener('click', () => {
    menu.style.display = 'none';
});
```

### Event Delegation

Add one listener to parent instead of many to children.

```javascript
// ✅ One event listener on parent
document.getElementById('menu').addEventListener('click', (event) => {
    const { target, currentTarget } = event;

    if (target.nodeName === 'LI') {
        // Handle click on list item
        console.log('Clicked:', target.textContent);
    }
});
```

### Debounce and Throttle

Control function execution rate for expensive operations.

```javascript
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// Debounce: Execute after quiet period
const searchHandler = debounce((query) => {
    // API call
}, 300);

input.addEventListener('input', (e) => searchHandler(e.target.value));

// Throttle: Execute at most once per interval
const scrollHandler = throttle(() => {
    // Update UI based on scroll
}, 100);

window.addEventListener('scroll', scrollHandler);
```

### requestAnimationFrame for Animations

```javascript
let lastScrollY = 0;

function updateParallax() {
    const scrollY = window.scrollY;

    // Update element positions
    element.style.transform = `translateY(${scrollY * 0.5}px)`;

    lastScrollY = scrollY;
    requestAnimationFrame(updateParallax);
}

requestAnimationFrame(updateParallax);
```

---

## Client-Side Data Requests

### Use Fetch API (Modern Environments)

```javascript
// GET request
fetch('/api/posts')
    .then(response => response.json())
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error('Error:', error);
    });

// POST request
fetch('/api/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({ title: 'New Post', content: 'Content here' }),
})
    .then(response => response.json())
    .then(data => console.log(data));

// With async/await
async function fetchPosts() {
    try {
        const response = await fetch('/api/posts');
        const posts = await response.json();
        return posts;
    } catch (error) {
        console.error('Error fetching posts:', error);
    }
}
```

### Polyfills for Older Browsers

For IE11 and older:

```bash
npm install --save promise-polyfill whatwg-fetch
```

```javascript
// Import at top of entry file
import 'promise-polyfill/src/polyfill';
import 'whatwg-fetch';
```

### XMLHttpRequest (Legacy Fallback)

Only use for very old browser support.

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/posts', true);

xhr.onload = function() {
    if (xhr.status === 200) {
        const data = JSON.parse(xhr.responseText);
        console.log(data);
    }
};

xhr.send();
```

### GraphQL with WPGraphQL

For WordPress projects, use WPGraphQL for efficient data fetching.

```javascript
const query = `
    query GetPosts {
        posts {
            nodes {
                id
                title
                content
            }
        }
    }
`;

fetch('/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query }),
})
    .then(res => res.json())
    .then(({ data }) => {
        console.log(data.posts);
    });
```

---

## Code Organization

### Avoid Deep Nesting

#### ❌ Don't: Callback hell

```javascript
getData((data) => {
    processData(data, (processed) => {
        saveData(processed, (result) => {
            updateUI(result);
        });
    });
});
```

#### ✅ Do: Use Promises or async/await

```javascript
// With Promises
getData()
    .then(processData)
    .then(saveData)
    .then(updateUI)
    .catch(handleError);

// With async/await
async function handleData() {
    try {
        const data = await getData();
        const processed = await processData(data);
        const result = await saveData(processed);
        updateUI(result);
    } catch (error) {
        handleError(error);
    }
}
```

---

## Common Pitfalls

### ❌ Don't: Use var

```javascript
// WRONG
var count = 0;

// CORRECT
const count = 0; // For constants
let counter = 0; // For variables that change
```

### ❌ Don't: Modify arrays/objects directly

```javascript
// WRONG
const items = [1, 2, 3];
items.push(4); // Mutation!

// CORRECT
const items = [1, 2, 3];
const newItems = [...items, 4]; // New array
```

### ❌ Don't: Use == (loose equality)

```javascript
// WRONG
if (value == '5') // true for both '5' and 5

// CORRECT
if (value === '5') // Only true for string '5'
```

---

## Quick Reference Checklist

### Modern JavaScript
- ✅ Use const/let instead of var
- ✅ Use arrow functions where appropriate
- ✅ Use template literals for strings
- ✅ Use destructuring for objects/arrays
- ✅ Use ES6 modules (import/export)
- ✅ Use async/await for async operations

### Performance
- ✅ Cache DOM selections
- ✅ Use event delegation
- ✅ Debounce/throttle expensive operations
- ✅ Use requestAnimationFrame for animations
- ✅ Import only what you need from libraries
- ✅ Lazy load heavy features

### Security
- ✅ Avoid innerHTML with user content
- ✅ Use textContent or createElement
- ✅ Sanitize HTML if innerHTML is necessary
- ✅ Validate and escape user input
- ✅ Use Content Security Policy headers

### Code Quality
- ✅ Use ESLint with recommended config
- ✅ Follow consistent code style
- ✅ Document complex logic
- ✅ Write modular, testable code
- ✅ Keep functions small and focused

---

## Tools and Linting

### ESLint Configuration

**Using WordPress ESLint plugin:**

```json
{
    "extends": ["plugin:@wordpress/eslint-plugin/recommended"],
    "rules": {
        "no-console": "warn",
        "no-var": "error",
        "prefer-const": "error",
        "no-new": "error"
    }
}
```

**Or use @wordpress/scripts (includes ESLint):**

```bash
npm install @wordpress/scripts --save-dev
```

```json
{
    "scripts": {
        "lint:js": "wp-scripts lint-js",
        "format": "wp-scripts format"
    }
}
```

### Recommended Libraries

- **Utilities:** Lodash (import specific functions)
- **HTTP:** Fetch API (with polyfills if needed)
- **State Management:** React Context API, Redux (if needed)
- **Testing:** Jest, Testing Library
- **Build Tools:** @wordpress/scripts (for WordPress projects)

---

## Further Reading

- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [JavaScript.info - Modern JavaScript Tutorial](https://javascript.info/)
- [WordPress JavaScript Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/javascript/)
- [You Don't Know JS (Book Series)](https://github.com/getify/You-Dont-Know-JS)
