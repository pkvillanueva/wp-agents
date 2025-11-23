# React Best Practices

**Reference Category:** Frontend Development
**Applies To:** React 16.8+, React 18+, WordPress Block Development
**Related:** JavaScript Best Practices, TypeScript,accessibility Principles

## Purpose

This reference provides comprehensive best practices for React development, with special attention to WordPress/Gutenberg integration. It covers component design, hooks, state management, routing, accessibility, and performance optimization.

## When to Use This Reference

- When building stateful user interfaces in WordPress
- When creating custom Gutenberg blocks
- When developing React-based admin interfaces
- When implementing client-side routing
- When building headless WordPress frontends

## Key Principles

1. **Functional Components with Hooks** - Prefer functional components over class components
2. **Resilient Components** - Write components that handle re-renders gracefully
3. **Accessibility First** - Ensure keyboard navigation, screen reader support, and ARIA compliance
4. **State Management** - Use Context API before reaching for Redux
5. **Keep It Simple** - Avoid over-engineering component architecture

---

## Component Types

### Functional Components (Recommended)

Use functional components with hooks for all new development.

#### ✅ Do: Use functional components with hooks

```jsx
import React, { useState } from 'react';

const SearchInput = () => {
    const [searchTerm, setSearchTerm] = useState('');

    const handleChange = (e) => {
        setSearchTerm(e.target.value);
    };

    return (
        <div className="search-input">
            <input onChange={handleChange} value={searchTerm} />
        </div>
    );
};

export default SearchInput;
```

**Why functional components?**
- Less boilerplate code
- Easier to test and understand
- Better code reuse with custom hooks
- Avoids `this` binding complexities

### Class Components (Limited Use Cases)

Only use class components when you need:
- Error boundaries (`componentDidCatch`)
- Specific lifecycle methods not available in hooks

```jsx
import React, { Component } from 'react';

class ErrorBoundary extends Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false };
    }

    componentDidCatch(error, errorInfo) {
        this.setState({ hasError: true });
        // Log error to service
    }

    render() {
        if (this.state.hasError) {
            return <h1>Something went wrong.</h1>;
        }

        return this.props.children;
    }
}
```

### PureComponents

**Generally not recommended** - functional components with proper hooks usage are better.

---

## State Management

### Context API (Recommended First)

Use React Context API for shared state before reaching for external libraries.

```jsx
import React, { createContext, useContext, useState, useEffect } from 'react';

// Create context
export const UserContext = createContext();

// Custom hook for consuming context
export const useUser = () => {
    const data = useContext(UserContext);
    return data?.user || null;
};

// Provider component
const UserProvider = ({ children }) => {
    const [user, setUser] = useState(null);

    useEffect(() => {
        fetchUser().then(setUser);
    }, []);

    return (
        <UserContext.Provider value={{ user }}>
            {children}
        </UserContext.Provider>
    );
};

// Usage in components
const UserMenu = () => {
    const user = useUser();

    return user ? <div>Logged in as {user.name}</div> : <div>Not logged in</div>;
};

const App = () => (
    <UserProvider>
        <UserMenu />
    </UserProvider>
);
```

**Context API is best when:**
- You need to share data across components
- You want to avoid prop drilling
- You don't need advanced features like time-travel debugging

**Create specialized contexts:**
- Separate contexts for different data domains (user, posts, settings)
- Prevents unnecessary re-renders
- Easier to reason about data flow

### Redux (When Context Isn't Enough)

Use Redux when you need:
- Time-travel debugging
- Complex state transitions based on actions
- Middleware for side effects
- State snapshots and rehydration

**Redux Principles:**
1. Single source of truth
2. State is read-only
3. Changes via pure functions (reducers)

**Important:** Never put side effects in reducers. Use middleware like Redux Saga or Thunks.

---

## Writing Resilient Components

### Principle 1: Don't Stop the Data Flow

Props and state can change - components should handle this.

#### ✅ Do: Use effects properly

```jsx
const Profile = ({ userId }) => {
    const [user, setUser] = useState(null);

    // Re-fetch when userId changes
    useEffect(() => {
        fetchUser(userId).then(setUser);
    }, [userId]); // Dependency array

    return <div>{user?.name}</div>;
};
```

#### ❌ Don't: Ignore prop changes

```jsx
const Profile = ({ userId }) => {
    const [user, setUser] = useState(null);

    // Only fetches on mount!
    useEffect(() => {
        fetchUser(userId).then(setUser);
    }, []); // Empty array = runs once

    return <div>{user?.name}</div>;
};
```

### Principle 2: Always Be Ready to Render

Components should handle being rendered multiple times.

```jsx
// Good: Safe to render multiple times
const Button = ({ onClick, label }) => {
    return <button onClick={onClick}>{label}</button>;
};

// Be careful with side effects - use useEffect
const Timer = () => {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const id = setInterval(() => setCount(c => c + 1), 1000);
        return () => clearInterval(id); // Cleanup
    }, []);

    return <div>{count}</div>;
};
```

### Principle 3: No Component is a Singleton

Don't assume your component will only render once on a page.

```jsx
// ✅ Each instance manages its own state
const Dropdown = ({ options }) => {
    const [isOpen, setIsOpen] = useState(false);

    return (
        <div>
            <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
            {isOpen && <ul>{/* options */}</ul>}
        </div>
    );
};

// Multiple dropdowns work independently
<Dropdown options={options1} />
<Dropdown options={options2} />
```

### Principle 4: Keep Local State Isolated

Don't lift state unless necessary.

```jsx
// ✅ Modal state is local to Modal component
const Modal = ({ content }) => {
    const [isOpen, setIsOpen] = useState(false);

    return (
        <>
            <button onClick={() => setIsOpen(true)}>Open</button>
            {isOpen && <div className="modal">{content}</div>}
        </>
    );
};

// ❌ Don't lift state unnecessarily
const App = () => {
    const [modal1Open, setModal1Open] = useState(false);
    const [modal2Open, setModal2Open] = useState(false);
    // This doesn't scale!
};
```

---

## Routing

### React Router (Recommended)

Use React Router for client-side routing in SPAs.

```jsx
import { BrowserRouter, Route, Switch, Link } from 'react-router-dom';

const App = () => (
    <BrowserRouter>
        <nav>
            <Link to="/">Home</Link>
            <Link to="/about">About</Link>
        </nav>
        <Switch>
            <Route exact path="/" component={Home} />
            <Route path="/about" component={About} />
        </Switch>
    </BrowserRouter>
);
```

### Routing Accessibility

**Critical requirements:**
1. Update page title on route change
2. Manage focus to new content
3. Announce page changes to screen readers

```jsx
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

const Page = ({ title, children }) => {
    const location = useLocation();
    const contentRef = useRef(null);

    useEffect(() => {
        // Update title
        document.title = title;

        // Move focus to content
        if (contentRef.current) {
            contentRef.current.focus();
        }

        // Announce to screen readers
        const announcement = document.createElement('div');
        announcement.setAttribute('role', 'status');
        announcement.setAttribute('aria-live', 'polite');
        announcement.textContent = `Navigated to ${title}`;
        document.body.appendChild(announcement);

        setTimeout(() => announcement.remove(), 1000);
    }, [location, title]);

    return (
        <main ref={contentRef} tabIndex="-1">
            {children}
        </main>
    );
};
```

---

## Accessibility

### Use Semantic HTML and Fragments

```jsx
// ✅ Do: Use semantic HTML
const Article = () => (
    <article>
        <h1>Title</h1>
        <p>Content</p>
    </article>
);

// ✅ Use fragments to avoid breaking semantics
const List = () => (
    <ul>
        {items.map(item => (
            <React.Fragment key={item.id}>
                <li>{item.name}</li>
                <li>{item.description}</li>
            </React.Fragment>
        ))}
    </ul>
);

// Or shorthand
<>
    <li>Item 1</li>
    <li>Item 2</li>
</>
```

### Accessible Forms

```jsx
const LoginForm = () => (
    <form>
        <label htmlFor="username">Username:</label>
        <input id="username" type="text" name="username" />

        <label htmlFor="password">Password:</label>
        <input id="password" type="password" name="password" />

        <button type="submit">Login</button>
    </form>
);
```

### Focus Management

```jsx
import { useRef, useEffect } from 'react';

const Modal = ({ isOpen, onClose }) => {
    const closeButtonRef = useRef(null);

    useEffect(() => {
        if (isOpen && closeButtonRef.current) {
            closeButtonRef.current.focus();
        }
    }, [isOpen]);

    if (!isOpen) return null;

    return (
        <div className="modal" role="dialog" aria-modal="true">
            <button ref={closeButtonRef} onClick={onClose}>
                Close
            </button>
            <div>Modal content</div>
        </div>
    );
};
```

### Use jsx-a11y ESLint Plugin

Install and configure the accessibility linter:

```bash
npm install --save-dev eslint-plugin-jsx-a11y
```

---

## Server-Side Rendering (SSR)

### When to Use SSR

Use SSR when:
- SEO is critical (especially for non-Google search engines)
- First contentful paint performance is critical
- You need to support search engines that don't execute JavaScript

### When NOT to Use SSR

Avoid SSR when:
- You're only improving SEO for a few marketing pages (use prerendering instead)
- Third-party libraries require browser globals
- Team lacks experience with SSR complexities

### Prerendering Alternative

For static marketing pages, use prerendering instead of full SSR:

```bash
npm install react-snapshot --save-dev
```

Prerendering generates static HTML at build time, simpler than runtime SSR.

---

## Gutenberg Integration

### @wordpress/element

Gutenberg uses `@wordpress/element` as an abstraction over React.

```jsx
import { useState } from '@wordpress/element';

const MyBlock = () => {
    const [value, setValue] = useState('');

    return (
        <input
            value={value}
            onChange={(e) => setValue(e.target.value)}
        />
    );
};
```

### Higher-Order Components

Gutenberg provides HOCs for common patterns:

```jsx
import { withState } from '@wordpress/compose';

const MyComponent = withState({
    isOpen: false,
})(({ isOpen, setState }) => (
    <button onClick={() => setState({ isOpen: !isOpen })}>
        {isOpen ? 'Close' : 'Open'}
    </button>
));
```

### Including/Excluding Blocks

Filter allowed blocks in Gutenberg:

```php
// functions.php - Allow specific blocks only
function my_allowed_blocks( $allowed_blocks, $editor_context ) {
    return array(
        'core/paragraph',
        'core/heading',
        'core/image',
        'custom/my-block',
    );
}
add_filter( 'allowed_block_types_all', 'my_allowed_blocks', 10, 2 );
```

**Prefer allowlist over blocklist** - explicitly allow blocks instead of blocking them.

---

## Performance Tips

### Avoid Unnecessary Renders

```jsx
import { memo } from 'react';

// Memoize expensive components
const ExpensiveComponent = memo(({ data }) => {
    return <div>{/* Complex rendering */}</div>;
});

// Use useCallback for functions
const Parent = () => {
    const handleClick = useCallback(() => {
        // Handle click
    }, []); // Dependencies

    return <ExpensiveComponent onClick={handleClick} />;
};
```

### Code Splitting

```jsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

const App = () => (
    <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
    </Suspense>
);
```

---

## Common Pitfalls to Avoid

### ❌ Don't: Mutate State Directly

```jsx
// WRONG
const addItem = () => {
    items.push(newItem);
    setItems(items);
};

// CORRECT
const addItem = () => {
    setItems([...items, newItem]);
};
```

### ❌ Don't: Use Index as Key

```jsx
// WRONG - causes issues when list order changes
{items.map((item, index) => (
    <div key={index}>{item}</div>
))}

// CORRECT
{items.map(item => (
    <div key={item.id}>{item}</div>
))}
```

### ❌ Don't: Call Hooks Conditionally

```jsx
// WRONG
if (condition) {
    useState(false);
}

// CORRECT
const [value, setValue] = useState(false);
if (condition) {
    // Use value
}
```

---

## Quick Reference Checklist

### Component Best Practices
- ✅ Use functional components with hooks
- ✅ Keep components small and focused
- ✅ Extract custom hooks for reusable logic
- ✅ Use prop-types or TypeScript for type checking
- ✅ Memoize expensive computations

### State Management
- ✅ Start with useState
- ✅ Use Context API for shared state
- ✅ Only use Redux when Context isn't enough
- ✅ Keep state as local as possible
- ✅ Never mutate state directly

### Accessibility
- ✅ Use semantic HTML elements
- ✅ Add ARIA attributes where needed
- ✅ Manage focus for modals and route changes
- ✅ Test with keyboard navigation
- ✅ Test with screen readers

### Performance
- ✅ Use React.memo for expensive components
- ✅ Implement code splitting
- ✅ Use useCallback and useMemo appropriately
- ✅ Avoid inline functions in render
- ✅ Optimize list rendering

---

## Further Reading

- [React Hooks Documentation](https://reactjs.org/docs/hooks-intro.html)
- [Writing Resilient Components](https://overreacted.io/writing-resilient-components/)
- [React Accessibility](https://reactjs.org/docs/accessibility.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Gutenberg Block Editor Handbook](https://developer.wordpress.org/block-editor/)
