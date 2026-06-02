---
name: java-project-standards
description: Enforce Alibaba-style Java project standards and layered architecture constraints. Use whenever Codex creates, reviews, refactors, debugs, or modifies Java, Spring, Maven, Gradle, MyBatis, JPA, or JVM backend project code; when a repository contains .java files; when checking Controller/Service/Manager/Repository/DAO/Mapper boundaries; or when evaluating naming, dependency, database, logging, testing, exception, and security conventions.
---

# Java Project Standards

## Purpose

Apply Java project governance based on the Alibaba Java Development Manual v1.5.0 and the referenced Aliyun layered architecture article. Treat this skill as an always-on guardrail for Java backend work: preserve existing project conventions, but prevent new code from violating core Java coding, layering, dependency, database, logging, test, and security standards.

Primary references:
- `references/java-standards-checklist.md` for the detailed checklist and rule severity.
- `references/alibaba-java-development-manual-v1.5.0.md` for the converted Alibaba Java Development Manual source material.

## Workflow

1. Detect the project shape before changing Java code.
   - Inspect `pom.xml`, `build.gradle`, `settings.gradle`, `.java` files, package layout, test layout, and existing docs such as `AGENTS.md`, `CLAUDE.md`, `README.md`, or project-specific style guides.
   - Identify Java version, build tool, Spring usage, persistence framework, test framework, and whether the architecture is classic MVC, DDD, or microservice-oriented.

2. Load the detailed checklist before implementation or review.
   - Read `references/java-standards-checklist.md` whenever the task touches Java source, Maven/Gradle dependencies, SQL/ORM mappings, database access, exception handling, logging, tests, or layer boundaries.
   - Use the checklist as a constraint system, not as decorative guidance.

3. Apply project-specific precedence.
   - Follow explicit user instructions and repository-local rules first.
   - When local rules conflict with this skill, name the conflict and choose the safer/local convention unless the user asks otherwise.
   - Do not introduce a new architecture style if the project already has a coherent one.

4. Enforce layer boundaries during design and edits.
   - Controller handles protocol adaptation, parameter binding, basic validation, authentication context, and response conversion.
   - Service owns business orchestration, business validation, and transaction boundaries. In Spring-style projects, define Service as an interface in the `service` package and place its implementation in `service.impl` with an `Impl` suffix.
   - Manager is optional; use it for reusable atomic capabilities, third-party integration isolation, cache/lock/idempotency helpers, and complex data aggregation shared by multiple Services.
   - Repository represents DDD aggregate persistence; DAO/Mapper represents table-oriented data access. Do not blur these names.
   - Object models live under `model`, separated by type: `model.entity`, `model.dto`, `model.vo`, and `model.pojo`; enum classes live under `enums`.
   - Prevent cross-layer shortcuts, especially Controller -> Mapper/DAO and Manager -> Service.

5. Make implementation choices that reduce future violations.
   - Prefer small cohesive classes and methods over broad utility dumping.
   - Depend on Service interfaces from Controller and other callers; do not inject or expose concrete `*ServiceImpl` classes outside `service.impl`.
   - Use DTO/VO/Query/Command/DO/BO names consistently with local patterns.
   - Place model classes in the correct `model.*` package instead of scattering them through Controller, Service, DAO, or Mapper packages.
   - Place enum classes in the `enums` package, name them with the `Enum` suffix when they represent business states or option sets, and keep enum constants uppercase with underscores.
   - Keep SQL, ORM mappings, and persistence-specific objects out of web/controller APIs and domain-facing interfaces.
   - Put transaction annotations at Service/application-service boundaries unless the local architecture clearly differs.

6. Verify before completion.
   - Run the narrowest meaningful checks available: `mvn test`, `mvn -q test`, `./mvnw test`, `gradle test`, `./gradlew test`, Checkstyle, SpotBugs, PMD, formatter, or Alibaba P3C if configured.
   - If verification cannot run, report exactly what was skipped and why.

## Review Output

When asked to review Java code, lead with findings ordered by severity:
- `P0/P1`: correctness, security, data consistency, broken transactions, severe layer violations, or production-impacting dependency problems.
- `P2`: maintainability, naming, logging, test gaps, excessive class size, missing validation, risky SQL/ORM usage.
- `P3`: minor style, readability, or non-blocking recommendations.

For each finding, include file and line references, the violated rule family, why it matters, and the smallest practical fix.

## Implementation Output

When editing Java code, summarize:
- Which layer or rule family was affected.
- Which verification command ran.
- Any remaining standard gap that was intentionally left unchanged because it was outside the request.
