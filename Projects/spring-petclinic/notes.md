# Spring Petclinic - Important Notes from pom.xml

## Project Information

- **Group ID**: `org.springframework.samples`
- **Artifact ID**: `spring-petclinic`
- **Version**: `4.0.0-SNAPSHOT`
- **Name**: `petclinic`
- **License**: Apache License, Version 2.0

## Parent Configuration

- **Parent**: Spring Boot Starter Parent
- **Version**: `4.0.1`

## Key Properties

| Property        | Value                | Notes                             |
| --------------- | -------------------- | --------------------------------- |
| Java Version    | 17                   | Minimum required JDK version      |
| Source Encoding | UTF-8                | Project-wide encoding             |
| Build Timestamp | 2024-11-28T14:37:52Z | Important for reproducible builds |

### Web Dependencies Versions

- **WebJars Locator**: 1.1.2
- **Bootstrap**: 5.3.8
- **Font Awesome**: 4.7.0

### Tool Versions

- **Checkstyle**: 12.1.2
- **JaCoCo**: 0.8.14
- **LibSass**: 0.3.4
- **Maven Checkstyle Plugin**: 3.6.0
- **Spring Format**: 0.0.47

## Core Dependencies

### Spring Boot Starters

- `spring-boot-starter-actuator` - Production monitoring and management
- `spring-boot-starter-cache` - Caching support
- `spring-boot-starter-data-jpa` - JPA/Hibernate for database access
- `spring-boot-starter-thymeleaf` - Thymeleaf template engine
- `spring-boot-starter-validation` - Bean validation
- `spring-boot-starter-webmvc` - Web MVC framework

### Database Support

- **H2** - In-memory database (runtime)
- **MySQL** (`mysql-connector-j`) - MySQL database (runtime)
- **PostgreSQL** - PostgreSQL database (runtime)

### Caching

- **Caffeine** - High-performance caching library

### Development Tools

- `spring-boot-devtools` - Development-time features (optional)

### Testing Dependencies

- `spring-boot-starter-data-jpa-test`
- `spring-boot-starter-restclient-test`
- `spring-boot-starter-webmvc-test`
- `spring-boot-testcontainers`
- `spring-boot-docker-compose`
- `testcontainers-junit-jupiter`
- `testcontainers-mysql`

## Important Build Plugins

### Maven Enforcer Plugin

> **Note**: Enforces Java version requirement. Build fails if JDK < 17.

### Spring Java Format Plugin

- Validates code formatting during the `validate` phase
- Version: 0.0.47

### Maven Checkstyle Plugin

- Includes nohttp-checkstyle validation
- Config location: `src/checkstyle/nohttp-checkstyle.xml`
- Excludes: `.git/`, `.idea/`, `target/`, `.flattened-pom.xml`, `.class` files

### Spring Boot Maven Plugin

> **Note**: Generates `META-INF/build-info.properties` for Spring Boot Actuator to display build information.

### JaCoCo Plugin

- Code coverage reporting
- Report generated during `prepare-package` phase

### Git Commit ID Plugin

> **Note**: Generates `git.properties` for Actuator to display Git information.
>
> - `failOnNoGitDirectory: false`
> - `failOnUnableToExtractRepoInfo: false`

### CycloneDX Maven Plugin

> **Note**: Generates SBOM (Software Bill of Materials) for Actuator.

## Maven Profiles

### `css` Profile

- Unpacks Bootstrap WebJar
- Compiles SCSS to CSS using LibSass
- Input: `src/main/scss/`
- Output: `src/main/resources/static/resources/css/`

### `m2e` Profile

- Activated when running in Eclipse
- Configures lifecycle mapping for Eclipse m2e
- Ignores certain plugin executions that don't affect builds

## Tips for Version Updates

Use the following command to update project version for reproducible builds:

```bash
./mvnw versions:set -DnewVersion=<new-version>
```
