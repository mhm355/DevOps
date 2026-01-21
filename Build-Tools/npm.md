# npm - Node Package Manager

## Files

### `package.json`
Lists packages your project depends on. Create with `npm init`.

```json
{
  "name": "package_name",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.17.1"
  },
  "devDependencies": {
    "jest": "^27.0.6",
    "nodemon": "^2.0.12"
  },
  "author": "Your Name",
  "license": "ISC"
}
```

### `package-lock.json`
Ensures reproducible builds by locking exact versions of dependencies.

```json
{
  "name": "package_name",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "express": {
      "version": "4.17.1",
      "resolved": "https://registry.npmjs.org/express/-/express-4.17.1.tgz",
      "integrity": "sha512-..."
    }
  }
}
```

### `.npmrc`
Configuration file for private npm registries (Nexus, Artifactory, GitHub Packages).

```txt
registry=https://registry.npmjs.org/
save-exact=true
strict-peer-dependencies=false
package-lock=true
```

### `.npmignore`
Exclude files from publishing.

```txt
node_modules/
src/
tests/
docs/
.gitignore
Dockerfile
README.md
```

---

## Directories

| Directory | Purpose |
|-----------|---------|
| `node_modules/` | Installed dependencies (auto-generated, don't commit) |
| `src/` | Main source code |
| `public/` | Static assets (HTML, CSS, images) |
| `config/` | Configuration files |

---

## Modules

### Core Modules
Built-in modules, import with `require`:
```js
const fs = require('fs');
const http = require('http');
const path = require('path');
```

### Third-Party Modules
Downloaded via `npm install`, stored in `node_modules/`.

---

## CLI Commands

### Basic Commands
```bash
npm install              # Install all dependencies
npm install --production # Production only (no devDependencies)
```

### CI/CD Commands
```bash
npm ci                   # Clean install (requires package-lock.json)
npm ci --only=production # Production clean install
npm run lint             # Check code quality
npm test                 # Run tests
npm run build            # Compile for deployment
npm start                # Run the application
npm audit                # Security vulnerability check
```

> **Note:** `npm ci` removes `node_modules/` first, ensuring clean reproducible builds.

---

## yarn

Alternative to npm with parallel package installation.

| Feature | Description |
|---------|-------------|
| **Parallel Install** | Faster than npm |
| **Offline Mode** | Install from cache without internet |
| **Lock File** | `yarn.lock` (same purpose as package-lock.json) |

### Commands
```bash
yarn install  # Install dependencies
yarn build    # Build project
yarn start    # Start application
```
