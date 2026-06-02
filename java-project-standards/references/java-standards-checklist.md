# Java Standards Checklist

Use this reference when working on Java projects. The rules are summarized from Alibaba Java Development Manual v1.5.0 and the Aliyun article on Java layered architecture. Treat `Mandatory` rules as blockers unless the repository already has an explicit incompatible convention.

## Source Notes

- Alibaba Java Development Manual v1.5.0 华山版 covers programming conventions, exception/logging, unit tests, security, MySQL, engineering structure, and design rules. It classifies rules as mandatory, recommended, and reference.
- The Aliyun article dated 2026-03-27 describes Java enterprise layering with Controller, Service, Manager, Repository, DAO, and Mapper. Its standard request path is client -> Controller -> Service -> Manager -> Repository/DAO/Mapper -> database or third-party systems.

## Architecture And Layers

### Standard Project Structure

Use this structure for Spring-style Java projects unless the repository already has a clear local convention:

```text
src/main/java/{basePackage}/
├── controller/              # request entry, parameter binding, basic validation, response wrapping
├── service/                 # Service interfaces only
│   └── impl/                # Service implementations, e.g. UserServiceImpl
├── manager/                 # reusable atomic capabilities and infrastructure orchestration helpers
├── repository/              # DDD repository interfaces/abstractions when used
├── dao/                     # table/data-source oriented DAO interfaces when used
├── mapper/                  # MyBatis Mapper interfaces
├── model/
│   ├── entity/              # persistence/domain entity objects
│   ├── dto/                 # Service/application transfer objects
│   ├── vo/                  # API/view response objects
│   └── pojo/                # ordinary helper model objects
├── enums/                   # enum classes, e.g. OrderStatusEnum
├── config/                  # framework and application configuration
├── exception/               # business and application exceptions
├── common/                  # shared constants/types for this application
└── util/                    # small stateless utilities only

src/main/resources/
├── mapper/                  # MyBatis XML files when XML mapping is used
└── application*.yml         # environment-aware application configuration

src/test/java/{basePackage}/ # tests mirror production package structure
```

Mandatory:
- Keep Service interfaces in `service` and implementations in `service.impl`.
- Keep object models under `model.entity`, `model.dto`, `model.vo`, or `model.pojo` according to their role.
- Keep enum classes under `enums`; do not scatter enums inside model, service, controller, or mapper packages.
- Keep Mapper XML files under `src/main/resources/mapper` when XML mappings are used.

### Controller

Mandatory:
- Keep Controller thin. It must not contain business workflow or persistence logic.
- Do not call DAO, Mapper, Repository, or third-party clients directly from Controller.
- Depend on Service/application-service interfaces only.
- Perform request binding, basic format validation, auth/context extraction, protocol conversion, and response wrapping.
- Do not leak raw exceptions or stack traces to clients.

Recommended:
- Keep Controller classes small enough to scan quickly. If a Controller grows beyond roughly 300 lines, look for endpoint grouping or workflow leakage.
- Use request DTOs and response VO/API response objects rather than exposing DO/entity/persistence objects.

### Service

Mandatory:
- Place business orchestration, business rule validation, and transaction boundaries in Service or application-service.
- Define Service contracts as interfaces in the `service` package, for example `UserService`.
- Place Service implementation classes under the `service.impl` package and name them with the `Impl` suffix, for example `UserServiceImpl`.
- Annotate/register the implementation class as the Spring bean, while Controllers and other callers depend on the Service interface.
- Do not inject or expose `*ServiceImpl` concrete classes outside `service.impl`.
- Do not depend on Controller or web-layer objects.
- Avoid large transactions. Do not put remote calls, slow I/O, cache writes, or message publishing inside a database transaction unless a deliberate consistency pattern exists.
- Do not duplicate reusable atomic logic across multiple Services; extract to Manager or a local reusable collaborator.

Recommended:
- A Service should map to one cohesive business capability or domain area. Avoid universal services.
- Keep Service classes below roughly 500 lines unless the existing project has a different accepted threshold.
- Throw business exceptions for rule failures and let the outer layer translate them to API responses.

### Manager

Mandatory:
- Use Manager only for reusable atomic capabilities below Service and above data access or infrastructure clients.
- Do not orchestrate business workflows in Manager.
- Do not call Service from Manager.
- Do not own transaction boundaries; Manager should participate in Service transactions when needed.

Recommended:
- Use Manager for cache access, distributed locks, idempotency, third-party client isolation, common queries, batch operations, and data aggregation reused by multiple Services.
- Omit Manager in small projects when Service and DAO/Mapper remain simple and cohesive.

### Repository, DAO, And Mapper

Mandatory:
- Do not write business rules in Repository, DAO, or Mapper.
- DAO and Mapper are table/data-source oriented. Keep them close to single-table CRUD or simple query responsibilities unless the project explicitly centralizes queries there.
- Repository is domain/aggregate oriented. In DDD code, a Repository should correspond to an aggregate root, not a random table.
- Do not expose persistence entities or DOs upward when the upper layer expects domain objects, DTOs, or VOs.
- Keep data access dependencies pointing downward only; data access code must not depend on Service or Controller.

Recommended:
- Use Repository interfaces in the domain layer and implementations in infrastructure for DDD projects.
- Use Mapper XML or annotations consistently with the existing MyBatis style.
- Avoid excessive join-heavy SQL, nested mappings, and N+1 query risks. Prefer explicit query design with tests or measured performance expectations.

### Request And Data Flow

Mandatory:
- Preserve a single-direction call chain. Classic MVC: Controller -> Service -> DAO/Mapper. With Manager: Controller -> Service -> Manager -> DAO/Mapper/client. DDD: interface -> application service -> domain service/entity -> repository interface -> infrastructure implementation.
- Validate at the proper layer: basic shape in Controller, business rules in Service/domain, persistence constraints in data access/database.
- Return data along the same path and convert objects at boundaries.

## Naming And Code Style

Mandatory:
- Do not start or end identifiers with underscore or dollar signs.
- Do not mix pinyin and English in names; do not use Chinese identifiers. Widely recognized names such as `taobao` or `hangzhou` may be treated as proper nouns.
- Use `UpperCamelCase` for class names, except accepted suffix acronyms such as `DO`, `DTO`, `BO`, `VO`, `AO`, `PO`, and `UID`.
- Use `lowerCamelCase` for method names, parameters, member variables, and local variables.
- Use uppercase words separated by underscores for constants.
- Name abstract classes with `Abstract` or `Base` prefix, exception classes with `Exception` suffix, and test classes with the tested class name plus `Test`.
- Name business enum classes with the `Enum` suffix, for example `OrderStatusEnum`; enum constants use uppercase words separated by underscores.
- Write arrays as `String[] args`, not `String args[]`.
- Do not prefix POJO Boolean fields with `is`. Map database `is_xxx` columns to Java `xxx` fields explicitly.
- Package names are lowercase, singular, dot-separated, and semantically meaningful.
- Avoid same names between parent/child fields and between local variables in different blocks of the same method.
- Avoid unclear abbreviations.

Recommended:
- Use names that reveal business intent over implementation details.
- Prefer explicit suffixes for boundary objects: `CreateUserRequest`, `UserDTO`, `UserVO`, `UserQuery`, `UserDO`, `UserBO`.

## Objects And Boundaries

Mandatory:
- Create a `model` package for project object models.
- Place entity objects under `model.entity`.
- Place DTO objects under `model.dto`.
- Place VO objects under `model.vo`.
- Place ordinary POJO/helper model objects under `model.pojo`.
- Place enum classes under `enums`.
- Do not scatter request/response/data objects directly under Controller, Service, DAO, Mapper, or unrelated feature packages unless the repository already has a clear local convention.
- Keep DO/entity objects aligned with persistence concerns and do not expose them directly in APIs.
- Use DTO for service/application transfer, VO for view/API responses, Query/Command/Request for input intent, BO/domain objects for business concepts.
- If a query has more than two parameters, encapsulate it in a query object instead of passing a `Map`.
- Do not use `Map<String, Object>` as a vague cross-layer contract.

Recommended:
- Centralize object conversion with explicit assemblers/converters/mappers when repeated conversions appear.
- Keep conversion code out of Mapper XML and away from Controller business flow.

## Dependencies And Build

Mandatory:
- Maven coordinates should be meaningful: `groupId` follows organization/business hierarchy, `artifactId` reflects product/module, and versions follow semantic major.minor.patch intent.
- Do not depend on `SNAPSHOT` versions in production applications except controlled security patches.
- Keep dependency versions unified with Maven properties or dependency management where appropriate.
- Do not declare the same `groupId` and `artifactId` with multiple versions across submodules.
- When upgrading dependencies, check transitive dependency changes and exclude unintended conflicts.
- Public library interfaces should not return enum types or POJOs containing enums if consumers may be cross-language or version-sensitive.

Recommended:
- Put dependency declarations in `dependencies` and version arbitration in `dependencyManagement`.
- Be conservative when introducing third-party libraries into low-level frameworks, core data platforms, or systems close to infrastructure.
- Prefer local project utilities only when they are cohesive; do not create broad "common" modules for unrelated code.

## Exceptions And Logging

Mandatory:
- Do not swallow exceptions silently.
- Do not catch `Exception` broadly without recovery, translation, or contextual logging.
- Do not use exceptions for normal control flow.
- Do not log and rethrow the same exception at multiple layers unless each log adds necessary context.
- Use parameterized logging instead of string concatenation.
- Do not print logs with `System.out`, `System.err`, or `e.printStackTrace()`.
- Do not log secrets, tokens, credentials, raw personal data, or sensitive request bodies.

Recommended:
- Define business exceptions with stable error codes/messages where APIs need predictable responses.
- Log enough context to diagnose the operation, but avoid excessive full object serialization.
- Let the boundary layer translate exceptions to user-facing responses.

## Unit Tests

Mandatory:
- Add or update tests for new behavior, bug fixes, business rules, transaction-sensitive paths, data access queries, and security-sensitive code.
- Do not rely on test execution order.
- Avoid tests that require external mutable services unless marked or isolated as integration tests.

Recommended:
- Keep unit tests fast and deterministic.
- Cover positive, negative, boundary, and exception paths for business logic.
- Use integration tests for SQL/ORM mappings when query behavior is non-trivial.
- Name tests after the behavior under test, not only the implementation method.

## Security

Mandatory:
- Validate untrusted input at the boundary and re-check business authorization in the Service/domain layer.
- Prevent SQL injection by using bound parameters; do not concatenate untrusted input into SQL.
- Do not expose stack traces, internal IDs, filesystem paths, or dependency versions in API errors.
- Do not hard-code credentials, tokens, private keys, or passwords.
- Mask sensitive information in logs and responses.
- For file, network, reflection, serialization, or expression-language use, check allowlists and input constraints.

Recommended:
- Use centralized authentication/authorization helpers already present in the project.
- Prefer explicit field allowlists for updates and queries.
- Consider replay, idempotency, and rate-limit behavior on write APIs.

## MySQL And ORM

Mandatory:
- Boolean database columns may use `is_xxx`, but Java POJO fields should map to `xxx`.
- Avoid `SELECT *`; list required columns.
- Always consider indexes for high-frequency query predicates and joins.
- Do not use unbounded queries for user-facing endpoints.
- Keep transaction isolation, locking, and update order explicit when concurrency matters.
- Do not let ORM lazy loading create hidden N+1 behavior in API serialization.

Recommended:
- Keep table and column names lowercase with underscores when the project follows MySQL conventions.
- Add comments or migration notes for schema changes when the repository has a migration system.
- Prefer batch operations for bulk writes when correctness and driver support allow it.

## Concurrency

Mandatory:
- Do not share mutable non-thread-safe objects across threads without synchronization or confinement.
- Use thread pools instead of creating raw threads in application code.
- Do not ignore interrupted status; restore interruption when appropriate.
- Guard distributed locks with timeouts, unique ownership tokens, and finally-release behavior.

Recommended:
- Keep asynchronous execution out of transaction scopes unless deliberately designed.
- Use idempotency keys for externally visible write operations that may retry.

## Comments And Documentation

Mandatory:
- Public APIs, complex business rules, concurrency logic, and non-obvious SQL must have concise explanatory comments.
- Comments must match behavior; update stale comments during edits.

Recommended:
- Explain why a decision exists, not what a simple line of code does.
- Document layer exceptions explicitly when legacy constraints force a non-standard dependency.

## Review Heuristics

Flag these aggressively:
- Controller directly calls Mapper/DAO/Repository.
- Service class exists without a matching interface in the `service` package.
- `*ServiceImpl` is not under `service.impl`, lacks the `Impl` suffix, or is injected directly by Controller/other callers.
- Model object classes are not placed under the corresponding `model.entity`, `model.dto`, `model.vo`, or `model.pojo` package.
- Enum classes are not under the `enums` package or business enum names do not end with `Enum`.
- Manager calls Service or starts transactions.
- Repository is used as table CRUD in a DDD package without aggregate semantics.
- Service contains large technical helper blocks better suited to Manager/infrastructure.
- Mapper XML contains business branching or too many multi-table joins.
- API returns DO/entity/JPA objects directly.
- `Map<String, Object>` crosses service boundaries.
- Duplicate dependency versions or production `SNAPSHOT`.
- `e.printStackTrace()`, `System.out`, string-concatenated logs, or sensitive logs.
- Missing tests for changed business logic.
