
 

# 1. Build Tools


## 1. **npm** : Node Package Manager

### Files:

* `package.json` file : lists the packages your project depends on , specifies versions of a package. can be created by `npm init`

```json
{
  "name": "package_name",
  "version": "1.0.0",
  "main": "index.js",  // the entry point to your application
  "scripts": {         // command-line scripts to automate tasks.
    "start": "node index.js",
    "test": "jest"
  },
  "dependencies": {     //  packages needed to run the app  in production
    "express": "^4.17.1"
  },
  "devDependencies": {    // packages needed for development and testing
    "jest": "^27.0.6",
    "nodemon": "^2.0.12"
  },
  "author": "Your Name",
  "license": "ISC"
 }
```

* `package-lock.json` file : Ensures reproducible builds, locking versions of dependencies.

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
      "integrity": "sha512-...",

      //....
    }
  }
}
```

* `.npmrc` file : The configuration file is essential if the company uses a private npm registry (like Nexus, Artifactory, or GitHub Packages)

```txt
registry=https://registry.npmjs.org/
save-exact=true
strict-peer-dependencies=false
package-lock=true

```


* `.npmignore` file :	Exclude files from publishing

```txt
node_modules/
src/
tests/
docs/
.gitignore
Dockerfile
README.md
```

### Directories:

  * `node_modules/` : Contains all installed dependencies and packages, automatically generated, don’t commit it to the repo.

  * `src/` : contain the main source code.

  * `public` : Contains static assets like HTML, CSS, images

  * `config/` : Configuration files for environment variables, database connections



#### modules : a reusable block of code in a file

  1. Core Modules : built in modules, import them using the require keyword. "fs, http, path"

```js
  const fs = require('fs');
```

  2. Third-Party Modules (from package.json) : downloaded by command `npm install` and will be stored in `node_modules` folder 


### Command-Line Interface (CLI):


  * `npm install`    install  dependencies. Packages install to `node_modules` folder

  * `npm install --production`     only installs the packages listed in "dependencies" in package.json.

  * used in CI/CD based on the script section in package.json :

    * `npm ci`    `ci` is clean install , used in automated environments, CI/CD , project must have an existing package-lock.json file, remove `node_modules` folder before `npm ci`

    * `npm ci --only=production`  

    * `npm run lint`  Check code quality

    * `npm test`  Run tests 

    * `npm run build`   compiles the application and prepares it for deployment

    * `npm start` Run the application from source code or the build output

    * `npm audit` Security vulnerability check




## 2. **yarn**

* It installs packages in parallel, faster than npm

* can install locally cached packages without an internet connection.

* `yarn.lock` file same as package-lock.json

* `yarn install , yarn build , yarn start` 

    
    
  
## 3. **pip** : package installer for Python

### Files:

* `requirements.txt` file : lists Python dependencies in a text file. Each line specifies a package name with optional version

  * limitation : only lists packages to install without broader project metadata. It cannot specify Python version requirements, project authors, licensing, or build configurations. 


* `pyproject.toml` file :  all project configuration into a single file.


### Command-Line Interface (CLI):

`pip install package_name` install the latest version from PyPI

`pip uninstall package_name`

`pip install package_name --upgrade`  upgrade the package

`pip install -r requirements.txt`  used to install all project dependencies from the requirements file.

`pip install -r requirements.txt --cache-dir .pip_cache` enables caching of downloaded packages

`pip download --cache-dir .pip_cache -r requirements.txt` re-download packages without installing them

`pip freeze > requirements.txt` generates a locked requirements file with exact versions of all installed packages

`pip check` validates that installed packages have compatible dependencies 

`pip list --outdated` identifies packages with available updates


## 4. **maven**

* Build automation and dependency management tool essential for Java project.

* maven lifecycle

  * Clean Lifecycle :

    - clean - deletes the target directory containing all generated build artifacts. `mvn clean`

  * build lifecycle :


    - validate - validate the project is correct and all necessary information is available
    
    - compile - compile the source code of the project, output is `target/`

    - test - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed

    - package - take the compiled code and package it in its distributable format, such as a JAR. (`file.jar` or `war`)

    - verify - run any checks on results of integration tests to ensure quality criteria are met

    - install - install the package into the local repository, for use as a dependency in other projects locally

    - deploy - done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.



### Files:

* `pom.xml` file :(Project Object Model ) Defines dependencies, plugins, Java version, packaging (JAR/WAR)

  sections of `pom.xml`

  1. Project Info

    - groupId → company/domain name 

    - artifactId → unique name for your project

    - version → release version 

    - packaging → type of output (jar, war).

  2. Dependencies

    - Maven will download these automatically from Maven Central or a private repo

  3. Build & Plugins

    - Plugins extend Maven functionality.

  4. Properties

    - define reusable variables throughout the pom.xml

  5. Profiles / Environment

    - different build settings for dev, test, prod

* `mvnw / mvnw.cmd`

  - Wrapper scripts → lets you run `./mvnw clean package` without requiring Maven installed


### Directories:

* `src/`

  * `src/main/java/`  contains application source code (.java files) , Maven compiles it  to `target/classes` using `mvnw compile`

  * `src/main/resources/`  Stores configuration and resource files (application.properties / application.yml) , Maven copies it to `target/classes` when compile

  * `src/main/webapp/` used specifically for web projects

  * `src/test/java/`  Contains unit and integration test code (UserServiceTest.java)

* `target/` generated after running `mvnw build`

  * `target/app.jar` or `.war` final build artifact, used to run the application.

  * `target/classes/` → compiled .class files.


* `.mvn/`

  - Appears if you use Maven Wrapper (mvnw).

  - Contains wrapper JAR and config to ensure consistent Maven version across environments.


* `~/.m2/repository/` local cache of dependencies.


### Command-Line Interface (CLI): 


  - `mvn clean` → removes old build files `target/`

  - `mvn compile` → compiles source code.

  - `mvn test` → runs unit tests.

  - `mvn package` → packages compiled code into .jar or .war.

  - `mvn install` → installs the artifact into the local repository `~/.m2/repository`

  - `mvn spring-boot:run` → runs Spring Boot apps directly (if plugin added).

  - `mvn dependency:go-offline -B` downloads all the required dependencies and plugins for the project based on `pom.xml` file, Cache them locally in `~/.m2/repository`, `-B` batch mode makes Maven run non-interactively.

  - `mvn clean package -DskipTests` Build and package the project, but don’t run the unit tests.



# 2. Docker

## Basics

## Dockerfile

### 1. nodejs application

* use small base image with sha256 digest  `FROM node:20-alpine@sha256:[digest-here]`

* copy only `package.json`  and `package-lock.json` before copying the source code, leverages Docker's layer caching—dependencies only rebuild when package files change.

* use `RUN npm ci --only=production` : eliminates devDependencies, reducing image size 

* use `ENV NODE_ENV production`

* Don’t run containers as root 

```dockerfile
FROM node:20.9.0-bullseye-slim
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY --chown=node:node . /usr/src/app    # by default files owned by root user
RUN npm ci --only=production
USER node
```

* Safely terminate Node.js container : don’t use 

```dockerfile
CMD “npm” “start”

CMD [“yarn”, “start”]

CMD “node” “server.js”

CMD “start-app.sh”

```
__use__ 

```dockerfile
CMD ["node","file.js"]   # if "scripts": { "start": "node app.js"} found in package.json file
```
 
this because CMD or ENTRYPOINT runs as Process ID 1 (PID 1). This process is special because it's responsible for receiving signals from the Docker daemon like SIGTERM at `npm start` npm becomes the PID1  This means your app never gets the message to shut down gracefully.After a timeout period (usually 10 seconds), Docker gives up and sends a SIGKILL, forcefully terminating the container. This can lead to data corruption, dropped connections, or incomplete tasks.


### 2. python application

* Choose the minimum base image that meets the requirements  like `FROM python:3.10-slim@sha256:2bac43769ace90ebd3ad83e5392295e25dfc58e58543d3ab326c3330b505283d`

* copy requirement file first 
```dockerfile
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt    # to prevent pip from storing downloaded packages in the image

# Poetry projects, copy pyproject.toml and poetry.lock first
# install dependencies with `poetry install --no-dev --no-root`
```

* use  `CMD [ "gunicorn", "--bind", "0.0.0.0:5000", "manage:app" ]` for run the application instead of `python3 app.py` 

  - Gunicorn (Green Unicorn) is a WSGI (Web Server Gateway Interface) server designed for synchronous Python web frameworks

  - Uvicorn is an ASGI (Asynchronous Server Gateway Interface) server specifically designed for modern asynchronous Python frameworks like FastAPI, Starlette, and async Django

  - For development and testing, use `Uvicorn`

  - For production deployments, the recommended approach is `Gunicorn with Uvicorn workers`



### 3. java application

* JDK (Java Development Kit) : Contains everything needed to develop and run Java applications

  - Compiler (javac)

  - Debugger & development tools

  - Full JRE (Java Runtime Environment)

* JRE (Java Runtime Environment) : Contains only what’s needed to run Java applications, not to compile them:

  - JVM (Java Virtual Machine) -> runtime engine that executes Java programs

  - Core libraries
  

* Use JDK only for build 
```dockerfile
FROM maven:3.9.6-eclipse-temurin-17 AS builder`
WORKDIR /app
```

* copy `pom.xml` first 

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline -B
```
* copy source code after download dependencies
```dockerfile
COPY src ./src
RUN mvn clean package -DskipTests
```
* Run the app using lightweight JRE image 
```dockerfile
FROM eclipse-temurin:17-jre`
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```







