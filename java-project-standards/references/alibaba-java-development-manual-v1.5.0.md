# Alibaba Java Development Manual v1.5.0 Condensed Reference

This is a compressed reference distilled from the local PDF source. Use it to recall Alibaba Java Development Manual rule families without loading the full manual text.

Rule severity:
- Mandatory: treat as a blocker unless repository-local rules explicitly override it.
- Recommended: prefer when implementing or refactoring.
- Reference: use as design guidance.

## Programming Conventions

### Naming

Mandatory:
- Identifiers must not start or end with `_` or `$`.
- Do not mix pinyin and English; do not use Chinese identifiers.
- Class names use `UpperCamelCase`, except accepted suffix acronyms such as `DO`, `BO`, `DTO`, `VO`, `AO`, `PO`, `UID`.
- Methods, parameters, fields, and local variables use `lowerCamelCase`.
- Constants use uppercase words separated by underscores.
- Abstract classes start with `Abstract` or `Base`; exception classes end with `Exception`; test classes end with `Test`.
- Arrays use `String[] args`, not `String args[]`.
- POJO Boolean fields must not use an `is` prefix. Database `is_xxx` columns should map to Java `xxx` fields.
- Package names are lowercase, singular, and semantically meaningful.
- Avoid duplicate names between parent/child fields or local variables in nested blocks.
- Avoid unclear abbreviations.
- Service and DAO exposed contracts must be interfaces; implementation classes use `Impl` suffix.

Recommended:
- Prefer complete business words over cryptic abbreviations.
- Include design pattern names in class or method names when a pattern is intentionally used.
- Interface methods and fields should not explicitly declare `public`; keep valid Javadoc.
- Service/DAO method prefixes: `get` single object, `list` multiple objects, `count` statistics, `save/insert`, `remove/delete`, `update`.
- Domain model suffixes: `xxxDO`, `xxxDTO`, `xxxVO`; do not name classes `xxxPOJO`.

### Constants

Mandatory:
- Do not use magic values directly in code.
- Use uppercase `L` for `long` literals, not lowercase `l`.

Recommended:
- Do not keep all constants in one giant constants class; classify by domain or function.
- Constant reuse levels: cross-application, application, submodule, package, class.
- Use enum types when values are limited to a fixed range and may need attached attributes.

### Formatting

Mandatory:
- Non-empty code blocks use normal Java brace style: left brace before newline; right brace on its own line except before `else`/similar continuation.
- No spaces just inside parentheses; one space before `{`.
- Add a space between `if/for/while/switch/do` and `(`.
- Add spaces around binary and ternary operators.
- Use 4 spaces for indentation; avoid tab characters.
- `//` comments have exactly one space after `//`.
- Cast syntax has no space after the closing parenthesis: `(int)value`.
- Keep lines within 120 characters; wrap after operators, dots, and commas, not before them.
- Method parameters have a space after each comma.
- File encoding is UTF-8 and line endings are Unix style.

Recommended:
- Keep each method within roughly 80 lines.
- Do not align assignment operators with extra spaces.
- Separate different logic blocks with one blank line, not many.

### OOP

Mandatory:
- Access static members by class name, not by object reference.
- All override methods must use `@Override`.
- Do not use deprecated classes or methods.
- `equals` comparisons between constants and variables should call `equals` on the constant or known non-null object.
- Objects with custom `equals` must also override `hashCode`.
- When using `Object.equals`, make sure the left side is non-null.
- Value comparisons for boxed primitives use `equals`; avoid `==` except for cached constant ranges.
- Do not modify collection elements inside `foreach`.
- Constructor logic must not call methods that can be overridden.
- Class member and method visibility should be as small as possible.

Recommended:
- Avoid variable-length parameters when a method with the same name already exists.
- Avoid building POJOs with excessive default values that obscure real input.
- Use `StringBuilder` for string concatenation inside loops.
- Use `final` where it clarifies immutability and thread safety.

### Collections

Mandatory:
- Do not use `subList` results after structurally modifying the original list.
- In `subList`, `toIndex` is exclusive.
- Use `toArray(new T[0])` or a correctly sized array for collection-to-array conversion.
- Do not use `Arrays.asList` result for add/remove/clear.
- Generic wildcard `<? extends T>` is read-oriented and must not be used for writes except `null`.
- Do not remove/add collection elements in foreach; use `Iterator.remove()` when removing during iteration.
- `Map` iteration should use `entrySet` when both key and value are needed.

Recommended:
- Set initial collection capacity when size is predictable.
- Use `Collection.isEmpty()` instead of `size() == 0`.
- Use `java.util.Objects.equals` for null-safe equality.
- Know collection null support and thread-safety traits before choosing an implementation.
- Use `Set` uniqueness instead of repeated `List.contains` scans for de-duplication.

### Concurrency

Mandatory:
- Singleton objects and their methods must be thread-safe.
- Name threads and thread pools meaningfully.
- Thread resources must come from thread pools, not ad hoc raw thread creation.
- Prefer `ThreadPoolExecutor` over `Executors` factories to avoid unbounded queues or thread counts.
- `SimpleDateFormat` is not thread-safe; use per-call instances, synchronization, `ThreadLocal`, or Java 8 date-time APIs.
- Remove `ThreadLocal` values after use, especially in thread pools.
- Lock acquisition and release must be paired, usually via `try/finally`.
- Avoid lock calls inside loops when possible.
- Use double-check locking with `volatile` if lazy initialization requires it.
- Use `CountDownLatch`, `Semaphore`, or similar primitives instead of busy loops where appropriate.
- Random in concurrent scenarios should use `ThreadLocalRandom`.

Recommended:
- Reduce lock scope.
- Prefer immutable objects for shared state.
- Use `Atomic*` or `LongAdder` for counters where suitable.
- Be careful with `HashMap` under concurrency; it is not thread-safe.

### Control Statements

Mandatory:
- Every `switch` case must end with `break`, `continue`, `return`, or a comment explaining fall-through.
- Every `switch` must include `default`, placed last.
- For `switch` on external `String` input, null-check before switching.
- Always use braces for `if/else/for/while/do`, even for one-line bodies.
- In high-concurrency scenarios, avoid equality checks as stop conditions when counters may skip or overshoot values.

Recommended:
- Prefer guard clauses for abnormal branches; keep `if/else` nesting shallow.
- Avoid assignments inside conditions.
- Avoid overly complex boolean expressions; assign meaningful boolean variables.
- Method return codes may use negative for error, positive for success, zero for neutral when that convention is used.

### Comments

Mandatory:
- Class, field, method, parameter, and return comments must be meaningful for externally exposed APIs.
- Abstract methods must document parameters, return values, exceptions, and template behavior.
- Enum fields need comments explaining each value.
- Comments must be updated when behavior changes.
- Special code with non-obvious intent must be documented.

Recommended:
- Explain design intent and business meaning, not obvious line-by-line behavior.
- Use TODO comments with owner/time/context when needed.

## Exceptions And Logging

### Exceptions

Mandatory:
- Do not catch exceptions without handling or logging.
- Do not use exceptions for normal control flow.
- Do not catch broad `Exception` or `Throwable` unless boundary handling is intentional.
- Do not ignore return values from methods that signal success/failure.
- Do not expose internal exception details to users.
- Use business exceptions for business rule failures and framework exceptions for technical failures.
- Cleanup resources in `finally` or try-with-resources.

Recommended:
- Translate exceptions at architectural boundaries.
- Avoid duplicate log-and-throw noise.
- Preserve the original exception cause when wrapping.

### Logging

Mandatory:
- Do not use `System.out`, `System.err`, or `printStackTrace`.
- Use parameterized logging, not string concatenation.
- Do not log secrets, credentials, tokens, private data, or sensitive request bodies.
- Avoid logging the same exception repeatedly at multiple layers.
- Production logs should be meaningful for diagnosis and not rely on local debugging prints.

Recommended:
- Use stable error codes where APIs need predictable error responses.
- Log enough business context to identify the operation and key identifiers.
- Choose log levels deliberately: error for failures needing attention, warn for risky but recoverable states, info for key lifecycle events, debug/trace for diagnostics.

## Unit Tests

Mandatory:
- New behavior and bug fixes must have tests where feasible.
- Unit tests must be deterministic and independent; do not rely on execution order.
- Unit tests should verify with assertions, not manual `System.out` inspection.
- Test data should not pollute shared external systems unless isolated as integration tests.

Recommended:
- Cover normal, boundary, exceptional, and concurrency-sensitive paths.
- Use meaningful test names that describe behavior.
- Keep tests fast; separate slow integration tests.
- For database or ORM behavior, include mapping/query tests when SQL is non-trivial.

## Security

Mandatory:
- Validate untrusted input.
- Enforce authorization in service/domain logic, not only at the web layer.
- Prevent SQL injection with parameter binding; never concatenate untrusted input into SQL.
- Do not hard-code credentials, private keys, passwords, or tokens.
- Do not expose stack traces, internal paths, dependency versions, or sensitive IDs in API responses.
- Mask sensitive information in logs and responses.
- Apply allowlists and strict constraints for file paths, reflection, serialization, expression evaluation, and remote calls.

Recommended:
- Prefer centralized auth and validation helpers.
- Use idempotency/replay protection for write APIs that may be retried.
- Apply rate limiting for externally exposed high-risk endpoints.

## MySQL

### Table Design

Mandatory:
- Table and column naming should be lowercase with underscores when using MySQL conventions.
- Boolean columns use `is_xxx`, but Java properties map to `xxx`.
- Table names should not use plural forms.
- Primary key column is usually `id`; create time and modified time columns are commonly `gmt_create` and `gmt_modified`.
- Avoid reserved words in table and column names.
- Use appropriate numeric and varchar lengths; do not over-allocate blindly.
- Text-like large fields should be isolated when they are rarely accessed.

Recommended:
- Add comments to tables and columns.
- Choose `decimal` for precise monetary values.
- Choose `unsigned` only when negative values are impossible.

### Indexes

Mandatory:
- Business-unique fields must have unique indexes, even if application logic validates uniqueness.
- More than three-table joins are prohibited.
- Joined columns must have identical types and suitable indexes.
- Index varchar columns with a specified prefix length when full length is unnecessary.
- Avoid left-fuzzy or full-fuzzy search with normal B-tree indexes.

Recommended:
- Use covering indexes where practical.
- Use delayed association/subquery strategies for very deep pagination.
- Aim for at least `range`, preferably `ref`, ideally `const` in query plans.
- Put high-selectivity equality columns early in composite indexes.
- Avoid implicit type conversion that invalidates indexes.

### SQL

Mandatory:
- Use `count(*)` for row counts.
- Be careful with `count(distinct col1, col2)` when columns may be null.
- Use `IFNULL(SUM(col), 0)` or equivalent when `SUM` may return null.
- Use `ISNULL()` or proper null checks instead of direct `= NULL`.
- If pagination total count is zero, return early before page query.
- Do not use foreign keys or cascading updates in high-concurrency distributed systems; enforce relationships in application logic.
- Do not use stored procedures for application business logic.
- For data correction, select and confirm before update/delete.

Recommended:
- Avoid large `IN` lists; keep them under roughly 1000 values when unavoidable.
- Use `utf8mb4` if storing emoji.
- Avoid `TRUNCATE` in application code.

### ORM Mapping

Mandatory:
- Do not use `SELECT *`; list required columns.
- Map database `is_xxx` columns to Java `xxx` Boolean properties.
- Define result mappings instead of relying on fragile result-class conventions.
- Use `#{}` parameter binding; do not use `${}` with untrusted input.
- Do not return raw `HashMap`/`Hashtable` for result sets.
- Update `gmt_modified` when updating rows.

Recommended:
- Avoid large all-purpose update APIs.
- Do not abuse `@Transactional`; consider QPS, rollback, cache, search, message compensation, and statistics correction.

## Engineering Structure

### Layered Models

Common object names:
- `DO`: data object aligned to database table.
- `DTO`: transfer object used across service/application boundaries.
- `BO`: business object carrying business semantics.
- `AO`: application object between web and service in some architectures.
- `VO`: view/API response object.
- `Query`: query request object; use it when query parameters exceed two.

Mandatory:
- Do not use `Map` as a vague multi-parameter or cross-layer transport object.
- Keep object conversions at architectural boundaries.

### Dependencies

Mandatory:
- Maven coordinates should be meaningful and stable.
- Versions follow major.minor.patch intent.
- Production apps should not depend on `SNAPSHOT` versions except controlled security patches.
- Dependency upgrades must inspect transitive resolution changes.
- A set of related dependencies should use a unified version property.
- Do not declare the same `groupId`/`artifactId` with different versions across submodules.

Recommended:
- Put version arbitration in `dependencyManagement`; submodules explicitly declare actual dependencies.
- Be conservative when introducing third-party dependencies into low-level or core infrastructure modules.

### Servers

Mandatory:
- High-concurrency servers should configure timeouts, thread counts, connection pools, and memory consciously.
- Application logs, temp files, and generated files must not fill system disks.
- Sensitive server configuration must not be committed.

Recommended:
- Keep deployment, environment, and runtime conventions documented for repeatable operations.

## Design Conventions

Mandatory:
- Design APIs and modules around clear responsibilities and stable contracts.
- Do not expose internal persistence details in external APIs.
- Keep call direction one-way across layers.
- Use interfaces for externally exposed capabilities and implementation classes for internal details.
- Preserve transaction boundaries around business operations rather than around low-level helpers.

Recommended:
- Prefer high cohesion and low coupling.
- Use domain models and domain services when business rules become complex.
- Avoid over-abstracting simple CRUD, but prevent cross-layer shortcuts that block future extension.
- Document deliberate deviations from the standard when legacy constraints require them.
