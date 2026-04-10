# Apache Maven — Build Automation for Java Projects

> A complete reference guide covering core concepts, lifecycle, repository management, and DevOps integration.

---

## Table of Contents

- [What is Maven?](#what-is-maven)
- [How Maven Works](#how-maven-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Project Structure](#project-structure)
- [pom.xml — Project Object Model](#pomxml--project-object-model)
- [Maven Repository System](#maven-repository-system)
- [Build Lifecycle & Commands](#build-lifecycle--commands)
- [Dependency Management](#dependency-management)
- [Offline / Air-Gapped Environments](#offline--air-gapped-environments)
- [Comparison with Other Build Tools](#comparison-with-other-build-tools)
- [Interview Quick Reference](#interview-quick-reference)

---

## What is Maven?

**Apache Maven** is a build automation and dependency management tool primarily used for Java projects. It standardizes how projects are built, tested, and deployed by using a declarative XML configuration file called `pom.xml`.

### Key responsibilities

- Declaring and resolving project dependencies (e.g. MySQL connector, AWS SDK, JDBC)
- Compiling source code into bytecode (`.class` files)
- Running unit and integration tests
- Packaging the compiled code into a distributable format (`.jar`, `.war`, `.ear`)
- Deploying artifacts to remote servers or repositories

---

## How Maven Works

```
MVN Central Repo
(mysql, awscli, jdbc, ...)
        |
        ▼
   pom.xml (your dependency declarations)
        |
        ▼
   Source Java Code
        |
        ▼
      Build
     /    \
Compile   Test
(class)  (JUnit)
        |
        ▼
      Package
   (.jar / .war)
        |
        ▼
      Deploy
        |
        ▼
  Server (e.g. EC2)
        |
        ▼
      Users
```

---

## Prerequisites

| Requirement | Version |
|-------------|---------|
| Java (JDK)  | 8 or higher |
| Maven       | 3.6 or higher |
| Internet    | Required for first-time dependency downloads (see [Offline Setup](#offline--air-gapped-environments)) |

Verify installation:

```bash
java -version
mvn -version
```

---

## Installation

**Linux / EC2:**

```bash
sudo apt update
sudo apt install maven -y
mvn -version
```

**macOS:**

```bash
brew install maven
```

**Windows:** Download from [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi) and add `bin/` to your `PATH`.

> After installation, Maven creates a `.m2/` directory in your home folder (`~/.m2/repository`). This is your **local repository** — all downloaded dependencies are cached here.

---

## Project Structure

A standard Maven project follows this layout:

```
my-project/
├── pom.xml                    ← Project configuration (dependencies, plugins, build settings)
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/example/   ← Your application source code
│   └── test/
│       └── java/
│           └── com/example/   ← Your test code (JUnit, TestNG)
└── target/                    ← Generated output (compiled classes, JAR/WAR) — do not commit
```

---

## pom.xml — Project Object Model

The `pom.xml` is the heart of every Maven project. It defines the project's identity, dependencies, and build behaviour.

### Minimal pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <!-- GAV Coordinates — uniquely identify this project in any Maven repository -->
  <groupId>com.mycompany</groupId>       <!-- Organisation or team namespace -->
  <artifactId>user-service</artifactId>  <!-- Module / project name -->
  <version>1.0.0</version>               <!-- Release version -->
  <packaging>jar</packaging>             <!-- Output format: jar | war | ear | pom -->

  <properties>
    <java.version>11</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
  </properties>

  <dependencies>

    <!-- MySQL JDBC Connector -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.33</version>
      <scope>runtime</scope>
    </dependency>

    <!-- AWS SDK -->
    <dependency>
      <groupId>software.amazon.awssdk</groupId>
      <artifactId>s3</artifactId>
      <version>2.20.0</version>
    </dependency>

    <!-- JUnit 5 — test scope only, not packaged in final JAR -->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.0</version>
      <scope>test</scope>
    </dependency>

  </dependencies>

</project>
```

### Dependency scopes

| Scope | Available at compile | Available at runtime | Packaged in JAR | Use case |
|-------|---------------------|---------------------|-----------------|----------|
| `compile` (default) | ✅ | ✅ | ✅ | Standard libraries |
| `provided` | ✅ | ❌ | ❌ | `servlet-api` (Tomcat provides it) |
| `runtime` | ❌ | ✅ | ✅ | JDBC drivers, MySQL connector |
| `test` | ✅ (test only) | ✅ (test only) | ❌ | JUnit, Mockito |
| `system` | ✅ | ✅ | ❌ | Local JAR path (avoid) |
| `import` | N/A | N/A | N/A | BOM imports only |

---

## Maven Repository System

Maven resolves dependencies by searching repositories in the following order:

```
1. Local Repository     →  ~/.m2/repository        (your machine)
         ↓ not found
2. Private Repository   →  Nexus / Artifactory      (your organisation)
         ↓ not found
3. Central Repository   →  repo.maven.apache.org    (public internet)
```

### Local repository — `.m2`

When Maven downloads a dependency for the first time, it saves it to `~/.m2/repository`. On subsequent builds, it reads from there — **no internet required** for cached dependencies.

```bash
# Location on Linux / EC2
~/.m2/repository/

# Example path for mysql-connector-java 8.0.33
~/.m2/repository/mysql/mysql-connector-java/8.0.33/mysql-connector-java-8.0.33.jar
```

### Private repository — Nexus / Artifactory

Organisations use private repository managers to:

- Host internal proprietary artifacts
- Cache and proxy Maven Central (air-gapped environments)
- Enforce security scanning on all dependencies
- Control who can publish and consume artifacts

Configure a private repo in `~/.m2/settings.xml`:

```xml
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>https://nexus.mycompany.com/repository/maven-public/</url>
    </mirror>
  </mirrors>

  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
```

> ⚠️ **Never store credentials in `pom.xml`** — it is committed to version control. Always use `settings.xml` or environment variables.

---

## Build Lifecycle & Commands

Maven has three built-in lifecycles. Each lifecycle is made up of sequential phases — running a phase automatically runs all preceding phases.

### Default lifecycle (most commonly used)

```
validate → compile → test → package → verify → install → deploy
```

| Phase | What it does |
|-------|--------------|
| `validate` | Checks pom.xml is valid and complete |
| `compile` | Compiles `.java` → `.class` files into `target/` |
| `test` | Runs unit tests via Surefire plugin |
| `package` | Bundles compiled code into a `.jar` or `.war` |
| `verify` | Runs integration tests via Failsafe plugin |
| `install` | Copies artifact to your local `.m2` repository |
| `deploy` | Pushes artifact to remote repository (Nexus/Artifactory) |

### Common commands

```bash
# Compile source code
mvn compile

# Run tests only
mvn test

# Create the JAR/WAR package
mvn package

# Install to local .m2 (useful for multi-module projects)
mvn install

# Push to remote repo (used in CI/CD pipelines)
mvn deploy

# Remove the target/ directory (clean slate)
mvn clean

# Clean then package — most common full build command
mvn clean package

# Skip tests (use only for local debugging — never in CI)
mvn clean package -DskipTests=true

# Run in offline mode — uses only what is in .m2 (no network calls)
mvn clean package -o
```

---

## Dependency Management

### Transitive dependencies

If your project depends on library A, and A depends on library B, Maven automatically resolves and downloads B as well. This is called a **transitive dependency**.

```
your-project
  └── library-A (direct)
        └── library-B (transitive — pulled in automatically)
```

### Excluding an unwanted transitive dependency

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>library-a</artifactId>
  <version>2.0.0</version>
  <exclusions>
    <exclusion>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### SNAPSHOT vs RELEASE versions

| | SNAPSHOT (e.g. `1.0-SNAPSHOT`) | RELEASE (e.g. `1.0.0`) |
|--|-------------------------------|------------------------|
| Meaning | Work in progress | Stable, finalised |
| Re-downloaded every build? | Yes (by default) | No — immutable |
| Used in | Feature branches, development | Production deployments |
| Best practice | Never deploy SNAPSHOTs to production | Always release from a tagged commit |

---

## Offline / Air-Gapped Environments

In environments without internet access (e.g. private VPCs without a NAT Gateway), use one of these approaches:

### Option 1 — Private Nexus/Artifactory as a proxy (recommended)

Set up Nexus inside your VPC. Pre-populate it by mirroring Maven Central. Point all servers to it via `settings.xml` mirror configuration. No server needs direct internet access.

### Option 2 — S3 bucket as a Maven repository

Use the `wagon-s3` Maven extension to serve artifacts from an S3 bucket. Useful when Nexus infrastructure is not available.

```xml
<!-- In pom.xml -->
<repositories>
  <repository>
    <id>s3-repo</id>
    <url>s3://my-bucket/maven-repo</url>
  </repository>
</repositories>
```

### Option 3 — Seed the .m2 cache and use offline mode

Run a full build on an internet-connected machine. Copy the populated `~/.m2/repository` to the air-gapped server. Then always run Maven with `-o` (offline flag).

```bash
mvn clean package -o
```

---

## Comparison with Other Build Tools

| Language | Package Manager | Dependency File | Central Repository |
|----------|-----------------|-----------------|--------------------|
| Java     | Maven / Gradle / ANT | `pom.xml` / `build.gradle` | [repo.maven.apache.org](https://repo.maven.apache.org) |
| Python   | pip             | `requirements.txt` | [pypi.org](https://pypi.org) |
| Node.js  | npm / yarn      | `package.json`  | [npmjs.com](https://registry.npmjs.org) |

### Maven vs Gradle vs ANT

| Feature | Maven | Gradle | ANT |
|---------|-------|--------|-----|
| Configuration style | Declarative XML | Groovy / Kotlin DSL | Imperative XML |
| Convention over config | ✅ Strong | ✅ Flexible | ❌ Manual |
| Incremental builds | Limited | ✅ Excellent | ❌ |
| Build speed | Moderate | Fast (caching) | Varies |
| Learning curve | Low | Medium | Low |
| Industry usage | Very high (enterprise Java) | High (Android, modern Java) | Legacy |

---

## Interview Quick Reference

| Question | One-line answer |
|----------|-----------------|
| What is `.m2`? | Local dependency cache created in home directory on first Maven install |
| What is GAV? | GroupId + ArtifactId + Version — uniquely identifies any artifact |
| `mvn install` vs `mvn deploy`? | `install` → local `.m2`; `deploy` → remote Nexus/Artifactory |
| What is `pom.xml`? | Project Object Model — declares dependencies, plugins, and build config |
| What is a SNAPSHOT? | A mutable, in-progress version — re-downloaded on every build |
| What is a transitive dependency? | A dependency pulled in automatically by one of your direct dependencies |
| What is Nexus/Artifactory? | Private artifact repository manager used inside organisations |
| What is Maven Central? | Public global repository at `repo.maven.apache.org` hosting open-source JARs |
| How does Maven resolve dependency conflicts? | Nearest-wins — the dependency closest to your project in the tree wins |
| What is `dependencyManagement`? | Declares versions centrally in a parent POM without forcing inclusion |

---

## References

- [Apache Maven Official Documentation](https://maven.apache.org/guides/)
- [Maven Central Repository](https://search.maven.org/)
- [Maven Repository (mvnrepository.com)](https://mvnrepository.com/)
- [Sonatype Nexus Repository](https://www.sonatype.com/products/nexus-repository)
- [JFrog Artifactory](https://jfrog.com/artifactory/)

---

*Notes compiled from classroom session — MAVEN_20260410*
