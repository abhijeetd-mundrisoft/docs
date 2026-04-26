# Technical Context Document - `course-forge-backend`

## A. Project Structure

- **Repository Type:** Single-module backend service.
- **Architecture Style:** Layered Spring Boot monolith with feature-oriented packages.
- **Build Tool:** Maven (`pom.xml`), Spring Boot parent `3.2.0`.
- **Java Version:** `17` (`java.version`, compiler source/target all set to `17`).
- **Primary Package Root:** `com.mundrisoft.courseforge`.

- **Top-Level Technical Layers:**
  - `config` - framework/security/openapi/web config
  - `controller` - REST entrypoints
  - `service` - orchestration + transaction boundaries
  - `repository` - Spring Data JPA interfaces
  - `entity` - JPA persistence models
  - `dto` - request/response transfer models
  - `mapper` - DTO/entity conversion (MapStruct + manual mapping)
  - `exception` - centralized error handling
  - `util` - shared helper utilities
  - `interceptor` - JWT request interception

- **Feature Sub-Packages (within same deployable):**
  - `scorm.*`
  - `scorm.editor.*`
  - `libreoffice.*`
  - `export.*`

## B. Architecture Summary

- **Request handling:** `@RestController` classes receive HTTP input and call services directly.
- **Business orchestration:** `@Service` classes coordinate repositories, file storage, and external integrations.
- **Data access:** `JpaRepository` interfaces with derived methods + `@Query` (JPQL + selective native SQL).
- **Persistence model:** Rich JPA entity model under `entity` plus feature-specific entities (example: `export.entity.CourseExport`).
- **Mapping pattern:** DTO-centric API contracts; mappers convert request DTO -> entity and entity -> response DTO.
- **Transaction model:** Class-level `@Transactional` on services with read-only overrides for query methods.
- **Security flow:** HTTP security config is permissive (`permitAll`), while JWT enforcement is implemented through MVC interceptor and controller/service checks.

## C. Technical Flow Mapping

### 1) Create Course
- **Endpoint:** `POST /courses` (`CourseController.createCourse`)
- **Flow:** Controller -> `CourseService.createCourse` -> `CourseRepository.save`
- **Storage adjunct:** File upload path uses `FileService.storeFile`/`updateFileMetadata` -> `FileMetadataRepository.save` + filesystem I/O.
- **Transformations:** multipart/form -> `CourseRequestDto` -> `Course` -> `CourseResponseDto`.
- **Technical checks:** Manual token extraction and workspace-access validation.

### 2) Get Course Details (Hierarchy)
- **Endpoint:** `GET /courses/{id}/details` (`CourseController.getCourseDetails`)
- **Flow:** Controller -> `CourseService.getCourseDetails` -> `CourseRepository` + `ModuleRepository` + `LessonRepository` + `BlockRepository`.
- **Transformations:** `Course` -> `CourseDetailsResponseDto`; nested `Module/Lesson/Block` -> hierarchy DTOs.
- **Transaction mode:** `@Transactional(readOnly = true)` in service path.

### 3) Create Module
- **Endpoint:** `POST /modules` (`ModuleController.createModule`)
- **Flow:** Controller -> `CourseService.getCourseById` (parent validation) -> `ModuleService.createModule` -> `ModuleRepository.findMaxOrderIndexByCourseId` + `save`.
- **Transformations:** `ModuleRequestDto` -> `Module` -> `ModuleResponseDto`.
- **Technical checks:** Controller injects audit fields (`createdBy`, `updatedBy`) from JWT context.

### 4) Update Module
- **Endpoint:** `PUT /modules/{id}` (`ModuleController.updateModule`)
- **Flow:** Controller -> `ModuleService.getModuleById` -> mapper merge -> `ModuleService.updateModule` -> `ModuleRepository.save`.
- **Transformations:** `ModuleRequestDto` + existing `Module` -> updated `Module` -> `ModuleResponseDto`.
- **Pattern:** Read-modify-write in service layer; mapper-based partial merge.

### 5) Create Lesson
- **Endpoint:** `POST /lessons` (`LessonController.createLesson`)
- **Flow:** Controller -> `ModuleService.getModuleById` -> `LessonService.createLesson` -> `LessonRepository.findMaxOrderIndexByModuleId` + `save`.
- **Transformations:** `LessonRequestDto` -> `Lesson` -> `LessonResponseDto`.
- **Validation:** `@Valid` request DTO + Jakarta validation constraints.

## D. Code Patterns

- **Controller design**
  - Annotation-based REST controllers; many endpoints include explicit header/token handling.
  - API responses generally standardized through `ApiResponseDto`.
  - Swagger/OpenAPI annotations are used for endpoint metadata.

- **Service design**
  - Services own orchestration and transaction semantics.
  - Cross-service collaboration exists (for example, course + file + workspace access flows).
  - Some services are large and contain mixed concerns.

- **Repository pattern**
  - Spring Data repository interfaces extend `JpaRepository`.
  - Query styles include derived method names, JPQL, constructor projections, and some native SQL.

- **DTO vs Entity usage**
  - Entity objects are persistence models.
  - DTOs are used at API boundaries.
  - Mappers (`mapper` package) enforce transformations between transport and persistence models.

- **Utilities/helpers**
  - Shared technical helpers in `util` (e.g., JWT helpers, email provider factory, common parsers/helpers).
  - Feature-level helpers in specialized packages (e.g., SCORM parser utilities).

## E. Frameworks Used

- **Core Platform**
  - Spring Boot
  - Spring MVC (`spring-boot-starter-web`)
  - Spring Data JPA (`spring-boot-starter-data-jpa`)
  - Spring Validation (`spring-boot-starter-validation`)
  - Spring Security (`spring-boot-starter-security`)
  - Spring AOP (`spring-boot-starter-aop`)
  - Spring Actuator (`spring-boot-starter-actuator`)
  - Spring WebFlux client support (`spring-boot-starter-webflux` for `WebClient`)

- **Persistence/ORM**
  - Hibernate/JPA via Spring Data JPA
  - MySQL driver
  - H2 for test scope

- **API/Serialization/Mapping**
  - Jackson
  - MapStruct
  - springdoc OpenAPI

- **Security/Identity**
  - JJWT (`jjwt-api`, `jjwt-impl`, `jjwt-jackson`)
  - BCrypt password encoder

- **Other Technical Libraries Present**
  - JODConverter, PDFBox, Apache POI, docx4j, flexmark, Jsoup, Apache Tika
  - Stripe SDK
  - AWS SES SDK + Jakarta Mail

- **Not detected in current backend**
  - Redis/Spring Cache integration
  - Kafka/Rabbit/JMS messaging stack

## F. Database Design

- **Entity Footprint**
  - Core entities include user/workspace/access, content hierarchy, AI config/usage, billing/payments, templates, and export metadata.

- **Key Relationship Patterns (examples)**
  - `Course` `@OneToMany` `Module`
  - `Module` `@ManyToOne` `Course`
  - `Lesson` `@ManyToOne` `Module`
  - `UserWorkspace` `@ManyToOne` (`User`, `Workspace`, `Role`)
  - `User` `@OneToMany` `UserWorkspace`
  - `User` `@OneToOne` `FileMetadata` (profile media)

- **Query Patterns**
  - Derived queries (`findBy...`, `countBy...`)
  - JPQL via `@Query`
  - Constructor projection queries
  - Selective native SQL where needed

- **Transaction Management**
  - Service-layer `@Transactional` as default write boundary.
  - Read paths frequently marked `readOnly = true`.
  - Global exception handling includes transaction exception unwrapping for constraint errors.

## G. Risk Areas

- **Large classes / methods**
  - Very large service/controller/parser classes (notably `UserService`, `ScormEditorController`, `RiseParser`) increase regression risk and change coupling.

- **Tight coupling**
  - Some controllers combine auth parsing, permission checks, mapping, and orchestration logic in single methods.
  - Repeated direct security-token handling across endpoints rather than fully centralized method security.

- **Potential performance pressure**
  - Hierarchical loading patterns may trigger repetitive repository calls (`Course -> Module -> Lesson -> Block` loops).
  - Enrichment loops that perform per-item repository/service lookups can cause N+1-like behavior.
  - Some repository interfaces contain many query methods, increasing maintenance and optimization complexity.

- **Repeated logic**
  - Repeated JWT extraction/access validation and repeated response assembly patterns across controllers.
  - Similar error-handling and validation behaviors appear across multiple paths, partly centralized but still duplicated locally in places.

- **Security consistency risk**
  - Security chain defaults to `permitAll`; authorization relies heavily on interceptor/controller checks, so inconsistent enforcement in new endpoints is a real risk.

## H. STRICT TECHNICAL RULES

1. **Do not change existing API contracts**
   - Preserve endpoint paths, HTTP methods, request/response DTO schemas, and response envelope conventions.

2. **Follow package structure strictly**
   - New code must be placed in existing layer/feature package conventions (`controller`, `service`, `repository`, `entity`, `dto`, `mapper`, etc.).

3. **Reuse existing services and repositories**
   - Extend current service flows before creating parallel logic paths.
   - Avoid duplicating validation, access checks, and mapping logic.

4. **Maintain backward compatibility**
   - Preserve DB column semantics, entity relationships, and query behavior unless migration-safe changes are explicitly planned.
   - Keep old clients functional for all existing endpoints.

5. **Follow existing logging and exception patterns**
   - Use structured SLF4J logging style and centralized exception handling via existing advice classes.
   - Return standardized `ApiResponseDto` error structures.

6. **Avoid introducing new frameworks**
   - Do not add new infrastructure stacks (messaging/cache/security frameworks) unless required by architectural decision records and compatibility impact analysis.

7. **Enforce thread safety and transaction safety**
   - Keep mutable state request-scoped or method-local unless explicitly synchronized.
   - Keep write flows inside service transactional boundaries.
   - Avoid non-atomic multi-repository writes outside `@Transactional`.

8. **Respect DTO/entity boundaries**
   - Do not expose JPA entities directly in external API contracts.
   - Keep mapper layer authoritative for DTO <-> entity transformations.

9. **Keep authorization checks technically consistent**
   - Every new controller endpoint must pass through existing JWT/interceptor + workspace/user access validation pattern.
   - Do not rely on `permitAll` as authorization.

10. **Preserve query performance characteristics**
    - Avoid adding iterative repository calls in loops when batch/fetch alternatives exist.
    - Validate large-list endpoints for N+1 and projection opportunities before merge.
