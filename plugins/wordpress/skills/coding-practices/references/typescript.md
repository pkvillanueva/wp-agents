# TypeScript Best Practices

**Reference Category:** Frontend Development
**Applies To:** TypeScript 4.x+, React, Next.js, JavaScript-First Projects
**Related:** JavaScript, React, Type Safety

## Purpose

This reference provides TypeScript best practices for JavaScript-first projects. It covers when to use TypeScript, typing strategies, escape hatches, React-specific patterns, and configuration recommendations.

## When to Use This Reference

- When building JavaScript-first projects (Next.js, headless CMS, SPAs)
- When adding type safety to React applications
- When struggling with TypeScript compilation errors
- When configuring tsconfig.json
- When deciding between TypeScript and JavaScript

## Key Principles

1. **Favor Efficiency Over Strict Typing** - Don't let TypeScript block development
2. **Never Ship Compilation Errors** - Use escape hatches if needed, but fix eventually
3. **Use Practical Types** - Simple types > overly complex types
4. **TypeScript for JavaScript-First** - Not required for standard WordPress projects
5. **Escape Hatches Are OK** - `any`, `as unknown as`, `@ts-expect-error` when stuck

---

## Why TypeScript?

### Benefits

1. **Confidence** - Catch errors at compile time
2. **Refactoring** - Easier and safer code changes
3. **Documentation** - Types serve as inline documentation
4. **Tooling** - Better IDE autocomplete and IntelliSense

### When to Use TypeScript

**Required for:**
- Next.js projects
- HeadstartWP framework projects
- Headless CMS with React frontends
- Large-scale React SPAs

**Not required for:**
- Standard WordPress themes/plugins
- Block development (limited type support)

---

## Core Principles

### 1. Never Ship Compilation Errors

TypeScript must compile without errors before deployment.

#### ❌ Don't: Ignore compilation errors

```bash
# WRONG - bypasses type checking
next build --no-type-check
```

#### ✅ Do: Fix errors or use escape hatches

```typescript
// Use escape hatches when stuck
const data = apiResponse as unknown as ExpectedType;
```

### 2. Favor Efficiency Over Strict Typing

Time-box typing challenges - don't block development for perfect types.

**When stuck on typing:**
1. Try for 15-30 minutes
2. Use an escape hatch with a TODO comment
3. Continue development
4. Revisit during code review

---

## Escape Hatches

### 1. Type Casting: `as unknown as`

Force a type when TypeScript can't infer correctly.

```typescript
// When types don't match expected
const response = await fetch('/api/data');
const data = await response.json() as unknown as MyDataType;

// Or chain: first to unknown, then to expected type
const processed = complexTransform(input) as unknown as ProcessedType;
```

**When to use:**
- API responses with known structure
- Complex transformations
- Third-party library type mismatches

### 2. The `any` Type

Disable type checking for a variable.

```typescript
function parseUnknownJSON(jsonString: string): any {
    return JSON.parse(jsonString);
}

// Use when structure is truly unknown
const config: any = JSON.parse(remoteConfig);
```

**Best practice:**
- Enable `noImplicitAny: true` in tsconfig
- Explicitly type as `any` (don't rely on implicit)
- Add TODO comments

```typescript
// TODO(username): Type this properly once schema is known
const apiData: any = await fetchFromUnknownAPI();
```

### 3. The `// @ts-expect-error` Comment

Suppress errors on a single line.

```typescript
// TODO(username): object not typed correctly but has `function` method
// @ts-expect-error
const result = object.function();
```

**Use sparingly** - only when other escape hatches don't work.

---

## TypeScript and React

### Use `type` for Props

Prefer `type` over `interface` for React component props.

```typescript
type ButtonProps = {
    label: string;
    onClick: () => void;
    disabled?: boolean;
};

export const Button: React.FC<ButtonProps> = ({ label, onClick, disabled = false }) => {
    return (
        <button onClick={onClick} disabled={disabled}>
            {label}
        </button>
    );
};
```

**When to use `interface`:**
- When props will be extended by other components
- When building a library/component system

### Use `React.FC` for Components

```typescript
import React from 'react';

type CardProps = {
    title: string;
    content: string;
};

export const Card: React.FC<CardProps> = ({ title, content }) => {
    return (
        <div className="card">
            <h2>{title}</h2>
            <p>{content}</p>
        </div>
    );
};
```

**Note:** React 18+ no longer includes `children` automatically in `React.FC`.

### Explicitly Declare `children`

Don't rely on `PropsWithChildren` - be explicit.

```typescript
type LayoutProps = {
    children: React.ReactNode;
    sidebar?: React.ReactNode;
};

export const Layout: React.FC<LayoutProps> = ({ children, sidebar }) => {
    return (
        <div className="layout">
            <main>{children}</main>
            {sidebar && <aside>{sidebar}</aside>}
        </div>
    );
};
```

### Use Function Default Arguments

Don't use `defaultProps` (deprecated in React 19).

```typescript
type ButtonProps = {
    label?: string;
    variant?: 'primary' | 'secondary';
};

export const Button: React.FC<ButtonProps> = ({
    label = 'Click Me',
    variant = 'primary',
}) => {
    return <button className={`btn btn-${variant}`}>{label}</button>;
};
```

### Avoid Prop Spreading

```typescript
// ❌ AVOID - loses type safety
type PostProps = {
    title: string;
    content: string;
    author: string;
};

const Post: React.FC<PostProps> = (props) => {
    return <PostContent {...props} />;
};

// ✅ PREFER - explicit props
const Post: React.FC<PostProps> = ({ title, content, author }) => {
    return <PostContent title={title} content={content} author={author} />;
};
```

### Co-locate Types with Components

Keep component types in the same file unless shared.

```typescript
// Button.tsx
type ButtonProps = {
    label: string;
};

export const Button: React.FC<ButtonProps> = ({ label }) => {
    return <button>{label}</button>;
};
```

**Only hoist types when:**
- Used across multiple files
- Avoiding circular dependencies
- Building a shared type library

---

## Recommended tsconfig.json Settings

```json
{
    "compilerOptions": {
        "target": "ES2020",
        "lib": ["ES2020", "DOM"],
        "module": "ESNext",
        "moduleResolution": "node",
        "jsx": "react-jsx",
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "noFallthroughCasesInSwitch": true,
        "noUnusedParameters": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    }
}
```

### Key Settings Explained

1. **`noImplicitAny: true`**
   - Forces explicit `any` declarations
   - Makes it easier to find code needing types

2. **`strictNullChecks: true`**
   - Requires handling `null` and `undefined`
   - Prevents common runtime errors

3. **`noFallthroughCasesInSwitch: true`**
   - Ensures switch statements have breaks or defaults

4. **`noUnusedParameters: true`**
   - Prevents unused function parameters

---

## Global Type Definitions

### Don't Pollute Global Scope

Only create global types when extending existing globals (like `window`).

```typescript
// globals.d.ts
interface Window {
    gtag?: (...args: any[]) => void;
    dataLayer?: any[];
}

// Usage
if (window.gtag) {
    window.gtag('event', 'page_view');
}
```

### Include in tsconfig.json

```json
{
    "include": [
        "src/globals.d.ts",
        "**/*.ts",
        "**/*.tsx"
    ]
}
```

---

## TypeScript and CI

### Run Type Checking in CI

Always run `tsc --noEmit` in CI pipelines.

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
    typecheck:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v2
            - uses: actions/setup-node@v2
            - run: npm ci
            - run: npm run typecheck

# package.json
{
    "scripts": {
        "typecheck": "tsc --noEmit"
    }
}
```

**Why?**
- Catches type errors before merging
- Prevents broken production builds
- Enforces type safety across team

---

## Common Patterns

### Typing API Responses

```typescript
type Post = {
    id: number;
    title: string;
    content: string;
    author: {
        id: number;
        name: string;
    };
};

async function fetchPost(id: number): Promise<Post> {
    const response = await fetch(`/api/posts/${id}`);
    return response.json() as Promise<Post>;
}
```

### Typing Event Handlers

```typescript
type FormProps = {
    onSubmit: (data: FormData) => void;
};

const Form: React.FC<FormProps> = ({ onSubmit }) => {
    const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
        e.preventDefault();
        const formData = new FormData(e.currentTarget);
        onSubmit(formData);
    };

    return <form onSubmit={handleSubmit}>...</form>;
};
```

### Typing useState

```typescript
// Type inference works
const [count, setCount] = useState(0); // count: number

// Explicit typing
const [user, setUser] = useState<User | null>(null);

// With default value
const [theme, setTheme] = useState<'light' | 'dark'>('light');
```

### Typing useRef

```typescript
import { useRef } from 'react';

const Component = () => {
    // For DOM elements
    const inputRef = useRef<HTMLInputElement>(null);

    // For mutable values
    const countRef = useRef<number>(0);

    return <input ref={inputRef} />;
};
```

---

## Tools

### TypeScript Error Translator
- [ts-error-translator.vercel.app](https://ts-error-translator.vercel.app/)
- Converts complex TS errors to plain English

### Pretty TS Errors (VSCode Extension)
- Improves error readability in VSCode
- [Marketplace Link](https://marketplace.visualstudio.com/items?itemName=yoavbls.pretty-ts-errors)

---

## Quick Reference Checklist

### When Writing TypeScript
- ✅ Use `type` for React props
- ✅ Use `React.FC` for components
- ✅ Explicitly declare `children`
- ✅ Use function defaults instead of `defaultProps`
- ✅ Avoid prop spreading
- ✅ Co-locate types with components

### When Stuck
- ✅ Time-box typing efforts (15-30 min)
- ✅ Use `as unknown as ExpectedType` first
- ✅ Use `any` if structure is truly unknown
- ✅ Use `@ts-expect-error` as last resort
- ✅ Add TODO comments with username

### Configuration
- ✅ Enable `noImplicitAny`
- ✅ Enable `strictNullChecks`
- ✅ Run `tsc --noEmit` in CI
- ✅ Keep tsconfig strict but practical

---

## Further Reading

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Total TypeScript](https://www.totaltypescript.com/)
- [Type Challenges](https://github.com/type-challenges/type-challenges)
