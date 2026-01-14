# Build Tools & Dependency Management

Understanding build tools is essential for Java development. This guide covers Maven and Gradle—the two dominant build tools in the Java ecosystem.

---

## 1) Build Tools Overview

### Why Build Tools?

- **Dependency management**: Automatically download and manage libraries
- **Compilation**: Compile source code consistently
- **Testing**: Run tests as part of the build
- **Packaging**: Create JARs, WARs, and other artifacts
- **Reproducibility**: Same build process everywhere
- **CI/CD integration**: Automate builds in pipelines

### Maven vs Gradle

| Aspect | Maven | Gradle |
|--------|-------|--------|
| Configuration | XML (`pom.xml`) | Groovy/Kotlin DSL |
| Philosophy | Convention over configuration | Flexibility |
| Performance | Slower | Faster (incremental builds) |
| Learning curve | Gentler | Steeper |
| Plugin ecosystem | Mature | Growing |
| Android | Not used | Default |
| Enterprise | Very common | Increasingly common |

---

## 2) Maven Fundamentals

### Project Structure (Convention)

```
my-project/
├── pom.xml                    # Project Object Model
├── src/
│   ├── main/
│   │   ├── java/              # Application source code
│   │   └── resources/         # Config files, properties
│   └── test/
│       ├── java/              # Test source code
│       └── resources/         # Test resources
└── target/                    # Build output (generated)
```

### Basic pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Project coordinates (GAV) -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>  <!-- jar, war, pom, ear -->

    <name>My Application</name>
    <description>A sample application</description>

    <!-- Properties -->
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.1.0</version>
        </dependency>
        
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Maven Coordinates (GAV)

Every Maven artifact is identified by:

| Element | Description | Example |
|---------|-------------|---------|
| **groupId** | Organization/project | `org.springframework` |
| **artifactId** | Project name | `spring-core` |
| **version** | Version number | `5.3.20` |
| **packaging** | Artifact type | `jar`, `war`, `pom` |
| **classifier** | Variant | `sources`, `javadoc` |

```xml
<!-- Full dependency declaration -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
    <scope>compile</scope>
    <type>jar</type>
    <classifier>sources</classifier>
    <optional>false</optional>
</dependency>
```

---

## 3) Maven Dependency Scopes

| Scope | Compile | Test | Runtime | Package | Transitive |
|-------|---------|------|---------|---------|------------|
| **compile** (default) | ✓ | ✓ | ✓ | ✓ | Yes |
| **provided** | ✓ | ✓ | ✗ | ✗ | No |
| **runtime** | ✗ | ✓ | ✓ | ✓ | Yes |
| **test** | ✗ | ✓ | ✗ | ✗ | No |
| **system** | ✓ | ✓ | ✗ | ✗ | No |
| **import** | N/A | N/A | N/A | N/A | BOM only |

### Scope Examples

```xml
<!-- compile (default) - needed for compilation and runtime -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
    <!-- scope is compile by default -->
</dependency>

<!-- provided - container provides at runtime (e.g., servlet API) -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>

<!-- runtime - not needed for compilation, needed at runtime -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
    <scope>runtime</scope>
</dependency>

<!-- test - only for tests -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>
```

---

## 4) Maven Build Lifecycle

### Default Lifecycle Phases

```
validate → compile → test → package → verify → install → deploy
```

| Phase | Description |
|-------|-------------|
| `validate` | Validate project is correct |
| `compile` | Compile source code |
| `test` | Run unit tests |
| `package` | Create JAR/WAR |
| `verify` | Run integration tests |
| `install` | Install to local repo (~/.m2) |
| `deploy` | Deploy to remote repository |

### Common Commands

```bash
# Compile
mvn compile

# Run tests
mvn test

# Package (creates JAR/WAR)
mvn package

# Install to local repository
mvn install

# Deploy to remote repository
mvn deploy

# Clean (delete target directory)
mvn clean

# Clean and package
mvn clean package

# Skip tests
mvn package -DskipTests        # Compiles tests, doesn't run
mvn package -Dmaven.test.skip  # Skips compilation and execution

# Run specific test
mvn test -Dtest=UserServiceTest
mvn test -Dtest=UserServiceTest#testCreate

# With profiles
mvn package -Pproduction

# Show dependency tree
mvn dependency:tree

# Check for dependency updates
mvn versions:display-dependency-updates

# Effective POM (resolved with inheritance)
mvn help:effective-pom
```

---

## 5) Maven Dependency Management

### Transitive Dependencies

Maven automatically includes dependencies of your dependencies:

```
Your App → Spring Boot → Spring Core → Commons Logging
```

### Dependency Conflicts

When multiple versions of same library exist:

**Maven uses "nearest wins" strategy:**
- Direct dependency wins over transitive
- First declaration wins at same depth

### Excluding Transitive Dependencies

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.20</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Dependency Management Section

Define versions centrally (doesn't add dependency, just manages version):

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.20</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.3.20</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Now just reference without version -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <!-- Version inherited from dependencyManagement -->
    </dependency>
</dependencies>
```

### Bill of Materials (BOM)

Import a BOM to manage versions of related dependencies:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Now all Spring Boot dependencies use compatible versions -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- No version needed - BOM provides it -->
    </dependency>
</dependencies>
```

---

## 6) Maven Plugins

### Common Plugins

```xml
<build>
    <plugins>
        <!-- Compiler Plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>

        <!-- Surefire Plugin (unit tests) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
        </plugin>

        <!-- Failsafe Plugin (integration tests) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>3.1.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- JAR Plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.3.0</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.Main</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>

        <!-- Shade Plugin (uber-jar with dependencies) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.5.0</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- Spring Boot Plugin -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.1.0</version>
        </plugin>
    </plugins>
</build>
```

---

## 7) Maven Profiles

```xml
<profiles>
    <!-- Development profile (default) -->
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>

    <!-- Production profile -->
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
        <dependencies>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>prod-monitoring</artifactId>
                <version>1.0.0</version>
            </dependency>
        </dependencies>
    </profile>

    <!-- Profile activated by JDK version -->
    <profile>
        <id>jdk17</id>
        <activation>
            <jdk>17</jdk>
        </activation>
    </profile>

    <!-- Profile activated by OS -->
    <profile>
        <id>windows</id>
        <activation>
            <os>
                <family>windows</family>
            </os>
        </activation>
    </profile>

    <!-- Profile activated by property -->
    <profile>
        <id>ci</id>
        <activation>
            <property>
                <name>env.CI</name>
                <value>true</value>
            </property>
        </activation>
    </profile>
</profiles>
```

```bash
# Activate profile
mvn package -Pprod

# Multiple profiles
mvn package -Pprod,ci
```

---

## 8) Maven Multi-Module Projects

### Parent POM

```xml
<!-- parent/pom.xml -->
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>core</module>
        <module>api</module>
        <module>web</module>
    </modules>

    <dependencyManagement>
        <!-- Shared dependency versions -->
    </dependencyManagement>

    <build>
        <pluginManagement>
            <!-- Shared plugin configuration -->
        </pluginManagement>
    </build>
</project>
```

### Child POM

```xml
<!-- core/pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>core</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <!-- Module-specific dependencies -->
    </dependencies>
</project>
```

### Project Structure

```
parent/
├── pom.xml
├── core/
│   ├── pom.xml
│   └── src/
├── api/
│   ├── pom.xml
│   └── src/
└── web/
    ├── pom.xml
    └── src/
```

---

## 9) Gradle Fundamentals

### Project Structure

```
my-project/
├── build.gradle(.kts)      # Build script
├── settings.gradle(.kts)   # Project settings
├── gradle/
│   └── wrapper/            # Gradle wrapper
├── gradlew                 # Unix wrapper script
├── gradlew.bat             # Windows wrapper script
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── build/                  # Build output
```

### Basic build.gradle (Groovy DSL)

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    runtimeOnly 'mysql:mysql-connector-java'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### build.gradle.kts (Kotlin DSL)

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.1.0"
    id("io.spring.dependency-management") version "1.1.0"
}

group = "com.example"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.apache.commons:commons-lang3:3.12.0")
    
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

---

## 10) Gradle Dependency Configurations

| Configuration | Maven Equivalent | Description |
|---------------|------------------|-------------|
| `implementation` | compile | Compile + runtime, not exposed to consumers |
| `api` | compile | Compile + runtime, exposed to consumers |
| `compileOnly` | provided | Compile only |
| `runtimeOnly` | runtime | Runtime only |
| `testImplementation` | test | Test compile + runtime |
| `testRuntimeOnly` | test | Test runtime only |
| `annotationProcessor` | - | Annotation processing |

### implementation vs api

```groovy
// library/build.gradle
dependencies {
    // Internal use - consumers won't see Gson on their classpath
    implementation 'com.google.gson:gson:2.10'
    
    // Exposed to consumers - they can use Spring in their code
    api 'org.springframework:spring-core:5.3.20'
}

// app/build.gradle (depends on library)
dependencies {
    implementation project(':library')
    // Can use Spring (api) - cannot use Gson (implementation)
}
```

---

## 11) Gradle Tasks

### Common Commands

```bash
# Build
./gradlew build

# Clean
./gradlew clean

# Run tests
./gradlew test

# Package without tests
./gradlew build -x test

# List tasks
./gradlew tasks

# Show dependencies
./gradlew dependencies

# Specific configuration dependencies
./gradlew dependencies --configuration runtimeClasspath

# Run application
./gradlew bootRun

# Build parallel
./gradlew build --parallel

# Build with cache
./gradlew build --build-cache
```

### Custom Tasks

```groovy
tasks.register('hello') {
    doLast {
        println 'Hello, Gradle!'
    }
}

tasks.register('copyDocs', Copy) {
    from 'docs'
    into 'build/docs'
}

// Task dependencies
tasks.register('fullBuild') {
    dependsOn 'clean', 'build', 'test'
}

// Configure existing task
tasks.named('test') {
    useJUnitPlatform()
    maxParallelForks = 4
    testLogging {
        events 'passed', 'skipped', 'failed'
    }
}
```

---

## 12) Gradle Multi-Project

### settings.gradle

```groovy
rootProject.name = 'my-app'

include 'core'
include 'api'
include 'web'
```

### Root build.gradle

```groovy
plugins {
    id 'java'
}

// Configuration for all projects
allprojects {
    group = 'com.example'
    version = '1.0.0'
    
    repositories {
        mavenCentral()
    }
}

// Configuration for subprojects only
subprojects {
    apply plugin: 'java'
    
    java {
        sourceCompatibility = '17'
    }
    
    dependencies {
        testImplementation 'org.junit.jupiter:junit-jupiter:5.9.3'
    }
    
    test {
        useJUnitPlatform()
    }
}
```

### Subproject Dependencies

```groovy
// api/build.gradle
dependencies {
    implementation project(':core')
}

// web/build.gradle
dependencies {
    implementation project(':core')
    implementation project(':api')
}
```

---

## 13) Dependency Locking

### Maven

```xml
<!-- Use versions plugin to lock versions -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>versions-maven-plugin</artifactId>
    <version>2.15.0</version>
</plugin>

<!-- Generate: mvn versions:lock-snapshots -->
```

### Gradle

```groovy
// Enable dependency locking
dependencyLocking {
    lockAllConfigurations()
}

// Generate lock file
// ./gradlew dependencies --write-locks

// Lock file: gradle.lockfile
```

---

## 14) Repository Configuration

### Maven

```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
    <repository>
        <id>company-nexus</id>
        <url>https://nexus.company.com/repository/maven-public</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>

<!-- For deployment -->
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>https://nexus.company.com/repository/maven-releases</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>https://nexus.company.com/repository/maven-snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

### Gradle

```groovy
repositories {
    mavenCentral()
    google()
    maven {
        url = uri("https://nexus.company.com/repository/maven-public")
        credentials {
            username = findProperty("nexusUsername") ?: ""
            password = findProperty("nexusPassword") ?: ""
        }
    }
}

publishing {
    repositories {
        maven {
            url = uri("https://nexus.company.com/repository/maven-releases")
            credentials {
                username = findProperty("nexusUsername") ?: ""
                password = findProperty("nexusPassword") ?: ""
            }
        }
    }
}
```

---

## 15) Build Reproducibility

### Best Practices

1. **Lock dependency versions**
   - Never use dynamic versions (`1.+`, `LATEST`)
   - Use BOMs for consistent versions

2. **Use wrapper**
   ```bash
   # Maven Wrapper
   mvn -N io.takari:maven:wrapper
   ./mvnw clean package
   
   # Gradle Wrapper
   gradle wrapper
   ./gradlew build
   ```

3. **Pin plugin versions**
   ```xml
   <!-- Maven: Always specify plugin versions -->
   <plugin>
       <artifactId>maven-compiler-plugin</artifactId>
       <version>3.11.0</version>  <!-- Don't rely on defaults -->
   </plugin>
   ```

4. **Use consistent JDK**
   ```xml
   <!-- Enforce JDK version -->
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-enforcer-plugin</artifactId>
       <version>3.3.0</version>
       <executions>
           <execution>
               <goals>
                   <goal>enforce</goal>
               </goals>
               <configuration>
                   <rules>
                       <requireJavaVersion>
                           <version>[17,)</version>
                       </requireJavaVersion>
                   </rules>
               </configuration>
           </execution>
       </executions>
   </plugin>
   ```

---

## 16) Interview Questions

### Maven
1. What is Maven and why use it?
2. Explain the Maven build lifecycle.
3. What is the difference between `compile`, `provided`, and `runtime` scopes?
4. How does Maven resolve dependency conflicts?
5. What is a BOM and when would you use it?
6. What is the difference between `dependencies` and `dependencyManagement`?
7. How do you create an executable JAR with Maven?

### Gradle
8. What are the advantages of Gradle over Maven?
9. Explain `implementation` vs `api` in Gradle.
10. What is the Gradle daemon?
11. How does Gradle achieve faster builds?
12. What is the difference between `buildscript` and `plugins` blocks?

### General
13. How do you handle transitive dependencies?
14. What is dependency locking and why is it important?
15. How do you ensure build reproducibility?
16. What is a multi-module project? When would you use one?
17. How do you configure different environments (dev, prod)?
18. What is a fat JAR / uber JAR?

---

## 17) Quick Reference

### Maven Commands

```bash
mvn clean                    # Delete target/
mvn compile                  # Compile sources
mvn test                     # Run tests
mvn package                  # Create JAR/WAR
mvn install                  # Install to local repo
mvn deploy                   # Deploy to remote repo
mvn dependency:tree          # Show dependencies
mvn versions:display-dependency-updates  # Check updates
mvn help:effective-pom       # Show resolved POM
```

### Gradle Commands

```bash
./gradlew build             # Full build
./gradlew clean             # Clean build/
./gradlew test              # Run tests
./gradlew dependencies      # Show dependencies
./gradlew tasks             # List tasks
./gradlew bootRun           # Run Spring Boot app
./gradlew --refresh-dependencies  # Force refresh
```

### Maven Scopes vs Gradle Configurations

| Maven | Gradle |
|-------|--------|
| compile | implementation / api |
| provided | compileOnly |
| runtime | runtimeOnly |
| test | testImplementation |
| - | annotationProcessor |

