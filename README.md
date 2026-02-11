# Firefly Parent POM

Parent POM for all **Firefly OpenCore Banking Platform** modules. It centralises dependency versions, plugin configuration, and shared build properties so that every child module inherits a consistent, tested baseline.

## What It Provides

| Concern | Description |
|---------|-------------|
| **Dependency management** | Versions for Spring Boot, Spring Cloud, database drivers, OpenAPI tooling, mapping, testing, and logging libraries |
| **Plugin management** | Pre-configured Maven plugins for compilation, deployment, code generation, source/javadoc packaging, and Spring Boot repackaging |
| **Shared properties** | Java version, encoding, OpenAPI code-gen conventions, and library versions used across the platform |

## Key Managed Versions

| Dependency | Version |
|------------|---------|
| Java | 21 |
| Spring Boot | 3.2.2 |
| Spring Cloud | 2023.0.0 |
| SpringDoc OpenAPI | 2.3.0 |
| PostgreSQL Driver | 42.7.1 |
| R2DBC PostgreSQL | 1.0.4.RELEASE |
| Flyway | 9.22.3 |
| OpenAPI Generator | 7.2.0 |
| MapStruct | 1.5.5.Final |
| Lombok | 1.18.30 |
| JUnit Jupiter | 5.10.1 |
| Testcontainers | 1.19.3 |

## Usage

Reference `firefly-parent` as the parent in your module's `pom.xml`:

```xml
<parent>
    <groupId>com.firefly</groupId>
    <artifactId>firefly-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```

All managed dependency and plugin versions are then available without explicit `<version>` tags.

## Managed Plugins

| Plugin | Purpose |
|--------|---------|
| `maven-compiler-plugin` | Java 21 compilation with Lombok + MapStruct annotation processing |
| `maven-deploy-plugin` | Artifact deployment to remote repositories |
| `openapi-generator-maven-plugin` | Server/client code generation from OpenAPI specs |
| `maven-source-plugin` | Source JAR packaging |
| `maven-javadoc-plugin` | Javadoc JAR packaging |
| `spring-boot-maven-plugin` | Spring Boot fat-JAR repackaging (excludes Lombok) |

## Project Structure

```
firefly-parent/
  pom.xml          # This parent POM
  README.md        # This file
```

## Related

- [firefly-bom](../firefly-bom) - Bill of Materials for all Firefly artifacts
