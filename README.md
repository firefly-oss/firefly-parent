# Firefly Parent POM

Parent POM for all **Firefly OpenPlatform** modules. It extends `org.fireflyframework:fireflyframework-parent` (which already manages Spring Boot, Spring Cloud, Lombok, MapStruct, and all other third-party dependency and plugin versions) and adds the Firefly-specific configuration on top. Banking, lending, and insurance modules share this parent.

## What It Provides

| Concern | Description |
|---------|-------------|
| **Firefly Framework BOM** | Imports `org.fireflyframework:fireflyframework-bom` so that all `org.fireflyframework` artifact versions are managed centrally |
| **Framework version** | Single source of truth for `fireflyframework.bom.version` (currently **26.10.19**, Spring Boot 4.1.1) |
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
- All `org.fireflyframework` dependency versions (via the BOM at **26.10.19**)
- All third-party dependency and plugin versions from `fireflyframework-parent`
- Firefly OpenAPI code-generation conventions

No need to declare `fireflyframework.bom.version` or import the `fireflyframework-bom` in your module — the parent handles it.

**Maven Central** hosts `26.10.19` (`https://repo1.maven.org/maven2/org/fireflyframework/fireflyframework-parent/26.10.19/`). No extra repository is required for OpenPlatform builds that can reach Central. For air-gapped or pre-sync CI, use a local `./mvnw install` of `fireflyframework-jafly`, or activate `-Pjafly-packages` for GitHub Packages (`https://maven.pkg.github.com/fireflyframework/fireflyframework-jafly`) with `read:packages` credentials for Maven server id `firefly-github`.

## Properties

| Property | Value | Purpose |
|----------|-------|---------|
| `fireflyframework.bom.version` | `26.10.19` | Version of the Firefly Framework BOM (Boot 4.1.1 train) |
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

## Consuming jafly 26.10.19 (OpenPlatform)

| Item | Value |
|------|-------|
| Framework parent / BOM | `org.fireflyframework:fireflyframework-parent` / `fireflyframework-bom` **26.10.19** |
| Spring Boot | **4.1.1** |
| Spring Cloud | **2025.1.3** |
| Java | **25** (`JAVA_HOME` must be JDK 25; Maven on JDK 26 will fail toolchain/source checks) |
| Registry | **Maven Central first** — no extra `<repository>` required when Central is reachable |

### Resolve the framework from Central

```xml
<!-- Already done by firefly-parent — do not re-pin in service POMs -->
<parent>
    <groupId>org.fireflyframework</groupId>
    <artifactId>fireflyframework-parent</artifactId>
    <version>26.10.19</version>
    <relativePath/>
</parent>
```

Central coordinates: `https://repo1.maven.org/maven2/org/fireflyframework/fireflyframework-parent/26.10.19/`.

### Dual-repo profile (optional GitHub Packages)

Activate only when Central is unreachable (air-gapped / pre-sync CI):

```bash
mvn -Pjafly-packages …   # server id firefly-github, PAT with read:packages
```

Prefer Central for **26.10.19+**. Leave `-Pjafly-packages` off by default. Alternative: `./mvnw install` from a local `fireflyframework-jafly` checkout.

### Boot 4 / OpenAPI regeneration notes

When bumping a service onto this parent:

1. Install `firefly-parent:1.0.0-SNAPSHOT` (this repo) so the BOM pin is **26.10.19**.
2. In each `-sdk` module: drop hardcoded `openapi-generator.version` / Jackson / swagger pins; inherit plugin + dependency versions from `fireflyframework-parent`; set `useJakartaEe=true` on the webclient generator; keep `javax.annotation-api` only while the generator still emits `javax.annotation.Generated`; add `spring-webflux` for the WebClient SDK runtime.
3. Apply Boot 4 config migrations from the [jafly Boot 4 guide](https://github.com/fireflyframework/fireflyframework-jafly/blob/main/docs/migration/boot4-application-guide.md) where used: `spring.jackson2.*` (not `spring.jackson.*`), OTLP tracing keys, Testcontainers 2 module names, springdoc 3.x.
4. Regenerate OpenAPI via the parent `generate-openapi` profile (`openapi.gen.skip=false` on `-web`); skip with `-DskipOpenApiGen` when iterating compile-only.
5. Verify with JDK 25: `mvn -B -ntp clean test`.

### Roll-out order

Prefer dependency order **core → domain → exp** for the same capability:

1. Shared libraries (`library-*`) and `core-common-*`
2. Capability cores (`core-lending-*`, `core-banking-*`, …)
3. Domain aggregators (`domain-*`)
4. Experience APIs (`exp-*`)

### Roll-out status (2026-10-18/19)

**Train:** jafly **26.10.19** / Spring Boot **4.1.1** / Java **25** — consumed from **Maven Central** (optional `-Pjafly-packages` fallback).

| Layer | Status | Notes |
|-------|--------|-------|
| `firefly-parent` | **Green** | Parent + BOM pin `26.10.19` |
| `library-common-batch` | **Green** | Boot 4 Flyway auto-config fixes |
| `core-common-*` | **Green** | All SDK modules unpinned + verified |
| `core-lending-*` | **Green** | All SDK modules unpinned + verified |
| `core-banking-*` | **Green** | Includes `core-banking-psdx` Boot 4 error/security fixes |
| `domain-*` (banking, lending, common, product, customer, distributor, core, data) | **Green** | SDK recipe + Boot 4 test fixes as needed |
| `exp-*` (accounts, cards, channel-management, distributor, lending, notifications, onboarding, profile, security) | **Green** | SDK recipe; WebFluxTest / `@MockitoBean` / reactive security excludes |
| `library-psps`, `library-rails` | **N/A (deferred)** | Still on `resilience4j-spring-boot3`; no OpenAPI SDK; out of this cutover |
| `app-authenticator`, `app-configurator-masters` | **N/A (deferred)** | Thin apps; bump with consumer cutover |

Verify a service with: `JAVA_HOME=$(/usr/libexec/java_home -v 25) mvn -B -ntp clean test -DskipOpenApiGen`.

## Related

- [fireflyframework-jafly](https://github.com/fireflyframework/fireflyframework-jafly) - Unified Firefly Framework (parent, BOM, libraries)
- [Boot 4 application migration guide](https://github.com/fireflyframework/fireflyframework-jafly/blob/main/docs/migration/boot4-application-guide.md)
- [firefly-bom](https://github.com/firefly-oss/firefly-bom) - Bill of Materials for all `com.firefly` artifacts
