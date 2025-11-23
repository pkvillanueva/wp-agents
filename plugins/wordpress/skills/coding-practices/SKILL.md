---
name: coding-practices
description: ALWAYS use this skill when writing, reviewing, editing, or analyzing ANY code in this WordPress project. Provides mandatory best practices, coding standards, and security guidelines for PHP, JavaScript, TypeScript, React, CSS, and HTML. Apply proactively for all file modifications, code generation, code reviews, refactoring, debugging, building features, implementing functionality, or answering code-related questions. Covers WordPress plugins, themes, Gutenberg blocks, WooCommerce, performance optimization, security (sanitization, validation, escaping, nonces, XSS prevention), accessibility (WCAG), and WordPress Coding Standards. Required for .php, .js, .jsx, .ts, .tsx, .css, and .html files.
---

# Coding Practices

This skill provides comprehensive best practices, patterns, and standards for WordPress development across multiple languages and technologies.

## When to Use This Skill

Use this skill when you need guidance on:

- Writing or reviewing code in PHP, JavaScript, TypeScript, React, CSS, or HTML
- Performance optimization for database queries, caching, or frontend rendering
- Security implementation including validation, sanitization, XSS prevention, or nonce usage
- Project architecture and file organization
- Accessibility compliance (WCAG, ARIA, semantic HTML)
- Modern tooling setup (build tools, linters, type checking)
- Best practices for any aspect of WordPress or web development

## Reference Documents

All best practices are organized in detailed reference documents:

### Backend Development

- **[PHP Best Practices](references/php.md)** - Performance, security, design patterns, WordPress standards, testing

### Frontend Development

- **[JavaScript Best Practices](references/javascript.md)** - ES6+ features, performance, security, modular code
- **[TypeScript Best Practices](references/typescript.md)** - Typing strategies, React patterns, configuration
- **[React Best Practices](references/react.md)** - Components, hooks, state management, accessibility, Gutenberg
- **[CSS Best Practices](references/css.md)** - Architecture, responsive design, modern CSS features
- **[HTML Markup Best Practices](references/html-markup.md)** - Semantic HTML, accessibility, SEO, performance

### Project Organization

- **[Project Structure & Design](references/project-structure.md)** - File organization, dependencies, architecture
- **[Node.js Best Practices](references/nodejs.md)** - Package management, build tools, workflows
- **[Development Tools](references/development-tools.md)** - Linting, bundlers, testing, CI/CD

## Core Principles

All references follow these fundamental principles:

1. **Performance First** - Write efficient code that scales
2. **Security Always** - Validate input, sanitize output, escape late
3. **Accessibility** - Ensure all users can access and use your code
4. **Maintainability** - Write clear, documented, testable code
5. **Standards Compliance** - Follow WordPress and web standards
6. **Keep It Simple** - Avoid over-engineering and unnecessary complexity

## Instructions

**CRITICAL: This skill MUST be applied to ALL code-related tasks in this WordPress project.**

When working on ANY task involving code:

1. **Always read relevant reference files FIRST** - Before writing or reviewing any code, use the Read tool to access the appropriate reference documents
2. **Identify the relevant technologies** - Determine which languages/frameworks apply (may be multiple)
3. **Apply ALL applicable patterns and principles** - Follow the examples, best practices, and security guidelines from the reference files
4. **Validate before completion** - Always check that security, performance, and accessibility requirements are met
5. **Reference the specific guidelines** - When applying best practices, mention which reference file and section guided your implementation

**This skill is mandatory and should be invoked proactively without waiting for the user to request code review or best practices.**

## Common Use Cases

- **Building a Gutenberg Block**: Read [react.md](references/react.md), [javascript.md](references/javascript.md), [php.md](references/php.md)
- **Creating a REST API**: Read [php.md](references/php.md), [javascript.md](references/javascript.md)
- **Building a Theme**: Read [project-structure.md](references/project-structure.md), [php.md](references/php.md), [html-markup.md](references/html-markup.md), [css.md](references/css.md)
- **Performance Optimization**: Read [php.md](references/php.md), [javascript.md](references/javascript.md), [css.md](references/css.md)
- **Security Implementation**: Read [php.md](references/php.md), [javascript.md](references/javascript.md)

## Validation Checklist

Before completing any task, verify:

- [ ] Code follows standards in relevant reference documents
- [ ] Security measures implemented (validation, sanitization, escaping)
- [ ] Performance optimizations applied where relevant
- [ ] Accessibility requirements met
- [ ] Code is documented and maintainable
