# Maven - Java Build Tool

Build automation and dependency management for Java projects.

## Maven Lifecycle

### Clean Lifecycle
```bash
mvn clean  # Delete target/ directory
```

### Build Lifecycle

| Phase | Description | Command |
|-------|-------------|---------|
| **validate** | Validate project structure | `mvn validate` |
| **compile** | Compile source code → `target/classes/` | `mvn compile` |
| **test** | Run unit tests | `mvn test` |
| **package** | Create JAR/WAR → `target/*.jar` | `mvn package` |
| **verify** | Run integration tests | `mvn verify` |
| **install** | Install to local repo `~/.m2/repository` | `mvn install` |
| **deploy** | Deploy to remote repository | `mvn deploy` |

---

## Files

### `pom.xml` (Project Object Model)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <!-- 1. Project Info -->
    <groupId>com.example</groupId>      <!-- Company/domain -->
    <artifactId>my-app</artifactId>     <!-- Project name -->
    <version>1.0.0</version>            <!-- Release version -->
    <packaging>jar</packaging>          <!-- Output type: jar/war -->

    <!-- 2. Properties -->
    <properties>
        <java.version>17</java.version>
        <spring.version>3.0.0</spring.version>
    </properties>

    <!-- 3. Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>

    <!-- 4. Build & Plugins -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!-- 5. Profiles (Environment-specific) -->
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <env>development</env>
            </properties>
        </profile>
    </profiles>
</project>
```

### Maven Wrapper (`mvnw` / `mvnw.cmd`)
Run Maven without requiring global installation:
```bash
./mvnw clean package
```

---

## Directory Structure

```
my-app/
├── pom.xml
├── mvnw / mvnw.cmd          # Maven wrapper scripts
├── .mvn/                    # Wrapper JAR & config
├── src/
│   ├── main/
│   │   ├── java/            # Application source code
│   │   ├── resources/       # Config files (application.properties)
│   │   └── webapp/          # Web resources (WAR projects)
│   └── test/
│       └── java/            # Test source code
└── target/                  # Build output (generated)
    ├── classes/             # Compiled .class files
    └── app.jar              # Final artifact
```

---

## CLI Commands

### Common Commands
```bash
mvn clean                    # Remove target/
mvn compile                  # Compile source code
mvn test                     # Run unit tests
mvn package                  # Create JAR/WAR
mvn install                  # Install to ~/.m2/repository
mvn spring-boot:run          # Run Spring Boot app (with plugin)
```

### CI/CD Commands
```bash
mvn dependency:go-offline -B # Download all dependencies (cache)
mvn clean package -DskipTests # Build without running tests
```

### Local Repository
Dependencies cached at: `~/.m2/repository/`

---

## Gradle (Alternative)

Faster, more flexible build tool using Groovy/Kotlin DSL.

```bash
./gradlew build              # Build project
./gradlew test               # Run tests
./gradlew bootRun            # Run Spring Boot app
```
