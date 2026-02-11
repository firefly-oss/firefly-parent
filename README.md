# Firefly Parent POM

Parent POM for all **Firefly OpenCore Banking Platform** modules. It extends `org.fireflyframework:fireflyframework-parent` (which already manages Spring Boot, Spring Cloud, Lombok, MapStruct, and all other third-party dependency and plugin versions) and adds the Firefly-specific configuration on top.

## What It Provides

| Concern | Description |
|---------|-------------|
| **Firefly Framework BOM** | Imports `org.fireflyframework:fireflyframework-bom` so that all `org.fireflyframework` artifact versions are managed centrally |
| **Framework version** | Single source of truth for `fireflyframework.bom.version` (currently **26.02.01**) |
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
- All `org.fireflyframework` dependency versions (via the BOM at **26.02.01**)
- All third-party dependency and plugin versions from `fireflyframework-parent`
- Firefly OpenAPI code-generation conventions

No need to declare `fireflyframework.bom.version` or import the `fireflyframework-bom` in your module — the parent handles it.

## Properties

| Property | Value | Purpose |
|----------|-------|---------|
| `fireflyframework.bom.version` | `26.02.01` | Version of the Firefly Framework BOM |
| `base.package` | `com.firefly` | Root Java package for all modules |
| `openapi.base.package` | `${base.package}.${project.artifactId}` | OpenAPI code-gen base package |
| `openapi.model.package` | `…interfaces.dto` | Generated DTO package |
| `openapi.api.package` | `…interfaces.api` | Generated API package |

## Related

- [fireflyframework-parent](https://github.com/fireflyframework/fireflyframework-parent) - The upstream framework parent (Spring Boot, plugins, etc.)
- [firefly-bom](https://github.com/firefly-oss/firefly-bom) - Bill of Materials for all `com.firefly` artifacts
