# Build Tools

Build tools automate the process of compiling, testing, and packaging code.

## Contents

| Tool | Language | File |
|------|----------|------|
| npm/yarn | JavaScript/Node.js | [npm.md](./npm.md) |
| pip | Python | [pip.md](./pip.md) |
| Maven | Java | [maven.md](./maven.md) |

## Quick Comparison

| Feature | npm | yarn | pip | Maven |
|---------|-----|------|-----|-------|
| Lock File | `package-lock.json` | `yarn.lock` | - | - |
| Config File | `package.json` | `package.json` | `requirements.txt` | `pom.xml` |
| Install Cmd | `npm install` | `yarn install` | `pip install -r` | `mvn install` |
| CI Install | `npm ci` | `yarn --frozen-lockfile` | `pip install` | `mvn dependency:go-offline` |
