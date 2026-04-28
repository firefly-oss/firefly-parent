# Firefly Parent POM

Parent POM for all **Firefly OpenCore Banking Platform** modules. It extends `org.fireflyframework:fireflyframework-parent` (which already manages Spring Boot, Spring Cloud, Lombok, MapStruct, and all other third-party dependency and plugin versions) and adds the Firefly-specific configuration on top.

## What It Provides

| Concern | Description |
|---------|-------------|
| **Firefly Framework BOM** | Imports `org.fireflyframework:fireflyframework-bom` so that all `org.fireflyframework` artifact versions are managed centrally |
| **Framework version** | Single source of truth for `fireflyframework.bom.version` (currently **26.04.03**) |
| **Project conventions** | Shared OpenAPI code-gen properties (`base.package`, model/api package layout) |

All third-party dependency versions (Spring Boot, Lombok, MapStruct, database drivers, testing frameworks, etc.) and plugin management are inherited from `fireflyframework-parent`. There is no need to redeclare them here.

## Inheritance Chain

```
org.fireflyframework:fireflyframework-parent  (Spring Boot, Lombok, plugins, …)
  └── com.firefly:firefly-parent              (fireflyframework BOM + project conventions)
        └── your module                       (inherits everything)
```

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

Your module then inherits:
- All `org.fireflyframework` dependency versions (via the BOM at **26.04.03**)
- All third-party dependency and plugin versions from `fireflyframework-parent`
- Firefly OpenAPI code-generation conventions

No need to declare `fireflyframework.bom.version` or import the `fireflyframework-bom` in your module — the parent handles it.

## Properties

| Property | Value | Purpose |
|----------|-------|---------|
| `fireflyframework.bom.version` | `26.04.03` | Version of the Firefly Framework BOM |
| `base.package` | `com.firefly` | Root Java package for all modules |
| `openapi.base.package` | `${base.package}.${project.artifactId}` | OpenAPI code-gen base package |
| `openapi.model.package` | `…interfaces.dto` | Generated DTO package |
| `openapi.api.package` | `…interfaces.api` | Generated API package |
| `openapi.gen.skip` | `true` | Controls OpenAPI spec generation (set to `false` in `-web` modules to enable) |

## Automated OpenAPI Generation

This parent POM includes a shared Maven profile (`generate-openapi`) that automates OpenAPI spec generation and SDK creation at build time. The profile is always active but skipped by default — each `-web` module opts in by setting `openapi.gen.skip=false`.

### How It Works

During `mvn install`, the profile:

1. **`pre-integration-test`** — Starts a lightweight Spring Boot app (`OpenApiGenApplication`) that loads only the controllers and Springdoc
2. **`integration-test`** — Fetches the OpenAPI spec from the running app and writes it to `target/openapi/openapi.yml`
3. **`post-integration-test`** — Stops the app

The `-sdk` module then uses `openapi-generator-maven-plugin` to generate typed API clients from the spec.

### Enabling in a Microservice

**Step 1:** Add two properties to the `-web` module's `pom.xml`:

```xml
<properties>
    <openapi.gen.skip>false</openapi.gen.skip>
    <openapi.gen.mainClass>com.firefly.your.module.web.openapi.OpenApiGenApplication</openapi.gen.mainClass>
</properties>
```

**Step 2:** Create `OpenApiGenApplication` in `src/test/java` (never packaged into production):

```java
@EnableOpenApiGen
@ComponentScan(basePackages = "com.firefly.your.module.web.controllers")
public class OpenApiGenApplication {
    public static void main(String[] args) {
        SpringApplication.run(OpenApiGenApplication.class, args);
    }
}
```

`@EnableOpenApiGen` and `AutoMockMissingBeansConfig` are provided by `fireflyframework-web`. See the [fireflyframework-web README](https://github.com/fireflyframework/fireflyframework-web) for details on those components.

**Step 3:** Run the build:

```bash
mvn clean install
```

The generated spec will be at `your-module-web/target/openapi/openapi.yml`. To skip generation temporarily, pass `-DskipOpenApiGen`.

### Profile Properties

| Property | Default | Purpose |
|----------|---------|---------|
| `openapi.gen.skip` | `true` | Set to `false` in `-web` modules to enable generation |
| `openapi.gen.mainClass` | _(none)_ | Fully qualified class name of the `OpenApiGenApplication` |

## Related

- [fireflyframework-parent](https://github.com/fireflyframework/fireflyframework-parent) - The upstream framework parent (Spring Boot, plugins, etc.)
- [firefly-bom](https://github.com/firefly-oss/firefly-bom) - Bill of Materials for all `com.firefly` artifacts
