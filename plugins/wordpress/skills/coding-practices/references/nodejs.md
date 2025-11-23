# Node.js Best Practices

**Reference Category:** Backend Development, Build Tools
**Applies To:** Node.js LTS versions, npm, Server-side JavaScript
**Related:** JavaScript, Package Management, Security

## Purpose

This reference provides Node.js best practices for version management, security updates, dependency management, and deployment strategies.

## When to Use This Reference

- When setting up Node.js projects
- When managing Node.js versions across team
- When handling npm dependencies and security
- When deploying Node.js applications
- When configuring Node.js environments

## Key Principles

1. **Use LTS Versions Only** - Never use odd-numbered versions in production
2. **Automate Dependency Updates** - Use bots to keep packages current
3. **Security First** - Run audits, fix vulnerabilities promptly
4. **Version Consistency** - Document and enforce Node versions across team
5. **Container Deployments** - Use official Docker images

---

## Node.js Versions

### Only Use LTS Versions

**Production rule:** Only use Active LTS or Maintenance LTS versions.

**Never use:**
- Odd-numbered versions (v11, v13, v15, v17, v19)
- Current (non-LTS) releases in production

**Why?**
- LTS versions receive security updates for 30 months
- Odd versions are experimental and short-lived
- Production stability requires long-term support

### Node.js Release Schedule

![Node.js Release Schedule](https://raw.githubusercontent.com/nodejs/Release/master/schedule.svg?sanitize=true)

**Key dates:**
- **Active LTS:** Full support, recommended for production
- **Maintenance LTS:** Security updates only
- **End of Life:** No updates, migrate away

### Starting New Projects

Always use the most recent Active LTS version when starting a project.

**As of 2025:**
- ✅ Node.js 20.x (Active LTS)
- ✅ Node.js 18.x (Maintenance LTS)
- ❌ Node.js 21.x (Odd - don't use)
- ❌ Node.js 17.x (EOL)

---

## Version Management

### Declare Supported Version

Every project must specify its Node version in `package.json`.

```json
{
    "name": "my-project",
    "version": "1.0.0",
    "engines": {
        "node": ">=18.0.0"
    }
}
```

### Use .nvmrc File

Create a `.nvmrc` file in project root for NVM users.

```
18.19.0
```

**Usage:**
```bash
# Switch to project's Node version
nvm use

# Or auto-switch with shell integration
# https://github.com/nvm-sh/nvm#deeper-shell-integration
```

### Shell Integration (Optional)

Auto-switch Node versions when changing directories:

```bash
# Add to ~/.bashrc or ~/.zshrc
# Automatically call "nvm use" when entering directory with .nvmrc
cdnvm() {
    cd "$@" || return
    nvm_path=$(nvm_find_up .nvmrc | tr -d '\n')

    if [[ -e $nvm_path/.nvmrc ]]; then
        nvm use
    fi
}

alias cd='cdnvm'
```

---

## Security Updates

### Node.js Server Updates

**Production servers must:**
- Run LTS versions only
- Auto-update to latest LTS minor versions
- Monitor security advisories
- Plan major version upgrades annually

**Update strategy:**
```bash
# Check current version
node --version

# Update via package manager (example: Ubuntu)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify update
node --version
```

### Dependency Security

JavaScript packages release security patches constantly. Stay current.

---

## Dependency Management

### npm audit

Regularly check for vulnerabilities.

```bash
# Check for vulnerabilities
npm audit

# Auto-fix non-breaking vulnerabilities
npm audit fix

# Fix including breaking changes (review carefully!)
npm audit fix --force
```

### Automated Dependency Updates

**Use GitHub/GitLab bots:**
- [Dependabot](https://github.com/dependabot) (GitHub)
- [Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate/)

**Setup:**
1. Enable bot in repository
2. Configure update frequency (weekly recommended)
3. Review and merge PRs promptly

### Manual Update Before Bot Setup

Before enabling automated updates, manually update all dependencies:

```bash
# Update all dependencies
npm update

# Check for major version updates
npm outdated

# Update to latest (including major versions)
npm install package-name@latest
```

**Why?**
- Reduces initial PR volume from bot
- Easier to review incremental updates
- Catches up project to current versions

### audit-ci for CI/CD

Block merges when high/critical vulnerabilities exist.

```bash
# Install audit-ci
npm install --save-dev audit-ci

# Add to package.json scripts
{
    "scripts": {
        "audit:ci": "audit-ci --moderate"
    }
}
```

**CI Configuration (.github/workflows/audit.yml):**
```yaml
name: Security Audit

on: [push, pull_request]

jobs:
    audit:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v2
            - uses: actions/setup-node@v2
              with:
                  node-version: '20'
            - run: npm ci
            - run: npm run audit:ci
```

**Severity levels:**
- `--low` - Block all vulnerabilities
- `--moderate` - Block moderate and above (recommended)
- `--high` - Block high and critical only
- `--critical` - Block critical only

---

## Package Management

### Use package-lock.json

Always commit `package-lock.json` to version control.

**Why?**
- Ensures consistent installs across team
- Locks down exact dependency versions
- Prevents unexpected updates

### npm ci vs npm install

**Use `npm ci` in CI/CD:**
```bash
# CI/CD pipelines
npm ci
```

**Use `npm install` locally:**
```bash
# Local development
npm install
```

**Differences:**
- `npm ci` - Faster, requires package-lock.json, deletes node_modules first
- `npm install` - Updates package-lock.json, slower, preserves node_modules

### Dependency Types

Understand `dependencies` vs `devDependencies`.

```json
{
    "dependencies": {
        "express": "^4.18.0",
        "lodash": "^4.17.21"
    },
    "devDependencies": {
        "webpack": "^5.75.0",
        "eslint": "^8.30.0",
        "@types/node": "^18.11.18"
    }
}
```

**Dependencies:**
- Required for application to run
- Installed in production
- Examples: express, lodash, react

**DevDependencies:**
- Only needed for development/build
- Not installed in production
- Examples: webpack, eslint, testing tools

### Semantic Versioning

Understand version ranges in package.json.

```json
{
    "dependencies": {
        "package": "^1.2.3"
    }
}
```

**Version syntax:**
- `1.2.3` - Exact version (too strict)
- `^1.2.3` - Compatible with 1.x.x (recommended)
- `~1.2.3` - Compatible with 1.2.x
- `*` or `latest` - Any version (dangerous!)

**Recommended:** Use `^` for most dependencies.

---

## Deployment

### Docker Deployments (Recommended)

Use official Node.js Docker images.

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

**Why Docker?**
- Consistent environments (dev/prod parity)
- Automatic Node.js version management
- Easy scaling and orchestration
- Pulls latest LTS minor versions

### Environment Variables

Never hardcode configuration - use environment variables.

```javascript
// server.js
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const apiKey = process.env.API_KEY;

if (!dbUrl || !apiKey) {
    throw new Error('Missing required environment variables');
}
```

**Use .env files locally:**
```bash
# .env (never commit!)
PORT=3000
DATABASE_URL=postgresql://localhost/mydb
API_KEY=your-secret-key
```

```javascript
// Load environment variables
require('dotenv').config();
```

---

## Best Practices Summary

### Version Management
- ✅ Use LTS versions only (never odd numbers)
- ✅ Declare version in package.json `engines`
- ✅ Create .nvmrc file
- ✅ Update production servers to latest LTS minor
- ✅ Plan annual major version upgrades

### Dependency Security
- ✅ Run `npm audit` regularly
- ✅ Use Dependabot or Renovate
- ✅ Use `audit-ci` in CI pipeline
- ✅ Fix high/critical vulnerabilities immediately
- ✅ Review moderate vulnerabilities within week

### Package Management
- ✅ Commit package-lock.json
- ✅ Use `npm ci` in CI/CD
- ✅ Use `^` for dependency versions
- ✅ Separate dependencies from devDependencies
- ✅ Keep packages up to date

### Deployment
- ✅ Use official Docker images
- ✅ Use environment variables for config
- ✅ Never commit secrets
- ✅ Use .env files locally
- ✅ Validate required env vars on startup

---

## Common Commands

```bash
# Check Node version
node --version

# Check npm version
npm --version

# Install dependencies
npm install

# Install for CI (faster, requires lock file)
npm ci

# Check for vulnerabilities
npm audit

# Fix vulnerabilities automatically
npm audit fix

# Check outdated packages
npm outdated

# Update package
npm update package-name

# Install specific version
npm install package-name@1.2.3

# Install and save to dependencies
npm install --save package-name

# Install and save to devDependencies
npm install --save-dev package-name

# Run script from package.json
npm run script-name

# Remove package
npm uninstall package-name
```

---

## Tools and Resources

### Version Management
- [nvm](https://github.com/nvm-sh/nvm) - Node Version Manager
- [n](https://github.com/tj/n) - Alternative version manager
- [fnm](https://github.com/Schniz/fnm) - Fast Node Manager (Rust-based)

### Security
- [npm audit](https://docs.npmjs.com/cli/audit) - Built-in vulnerability scanning
- [audit-ci](https://www.npmjs.com/package/audit-ci) - Audit in CI/CD
- [Snyk](https://snyk.io/) - Advanced security platform
- [Dependabot](https://github.com/dependabot) - Automated updates

### Resources
- [Node.js Release Schedule](https://nodejs.org/en/about/releases/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [npm Documentation](https://docs.npmjs.com/)

---

## Further Reading

- [Node.js Official Docs](https://nodejs.org/docs/)
- [npm Best Practices](https://docs.npmjs.com/cli/best-practices)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [12-Factor App](https://12factor.net/)
