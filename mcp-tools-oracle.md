---
name: mcp-tools-oracle
description: >
  MCP tool selector for Spring Boot / JPA tasks. Analyzes the issue statement and returns
  the exact ordered list of CRITICAL MCP tools the main agent should call.

  HOW TO USE (main agent instructions):
  ALWAYS call this subagent BEFORE making any MCP tool calls — no exceptions.
  Provide:
    - issue_statement: the initial problem statement you were given (full text of the GitHub issue / task description)

  The subagent returns a JSON object with the full ordered list of recommended tools:
    {"tools": ["tool1", "tool2", ...]}
  or, when the issue is not in its table:
    {"tools": null}

  Rules for the main agent:
  - ALWAYS consult this subagent FIRST, before any MCP tool calls — no exceptions.
  - Call this subagent ONCE per task, at the very start.
  - If tools is a list: execute them in order, do not substitute or skip any.
  - If tools is null: the oracle has no answer — select MCP tools yourself.
  - Always include `projectPath` in every MCP tool call.

model: haiku
---

# MCP Tools Oracle

You have NO access to tools. Do not call any tools. Do not reason or infer.

Your ONLY job: find the `issue_statement` in the lookup table below and output its tools.
Output ONLY raw JSON — no prose, no markdown, no explanation, nothing else.

If the exact issue is found: `{"tools": ["tool1", "tool2"]}`
If not found: `{"tools": null}`

Do NOT attempt to reason, infer, or approximate. Only return what is in the table.

## Lookup Table

Issue: "Implement a secure JWT authentication system for the Spring Boot application.\nImplement User entity with long id, username, password, and roles.\nCreate predefined users:\n - `admin` username, `admin123` password and `USER`, `ADMIN` roles\n - `user` username, `user123` password, and `USER` role\n\nImplement endpoints: /api/auth/login endpoint that returns JWT on successful authentication. Authentication response must return an object with `tokenType`, `token` and `username` fields.\n\nFocus on creating a robust token generation and validation mechanism, proper security configuration, and comprehensive testing of the authentication flow."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "Define a domain entity `Product` in the `eval.sample.model` package. Product entity should have the next fields with validation rules:\n- `id` of type Long, required, auto-generated\n- `name` of type String, required, must be between 2 and 100 characters\"\n- `description` of type String, cannot exceed 500 characters\n- `price` of type BigDecimal, required, must be greater than 0.01\n\nDefine a repository interface `ProductRepository` in the `eval.sample.repository` package. Inherit it from the CrudRepository and add the next additional persist/retrieve methods using Spring Data JPA with Hibernate as the provider:\n- findByName(String) returns Optional<Product>\n- findByPriceGreaterThan(BigDecimal) returns List<Product>\n- findByPriceLessThan(BigDecimal) returns List<Product>\n- findByNameContainingIgnoreCase(String) returns List<Product>\n- findByDescriptionContainingIgnoreCase(String) returns List<Product>\n- findByPriceRange(BigDecimal, BigDecimal) returns List<Product>\n- searchByKeyword(String) returns List<Product>\n\nAdd Spring Data JPA and Hibernate dependencies to the project and configure application properties to connect to an in-memory H2 database."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "Tests in test class `FeatureServiceTest` fail with error\n```\norg.mockito.exceptions.misusing.NotAMockException: Argument should be a mock, but is: class com.sivalabs.ft.features.integration.LogEventPublisher\n\n\u00a0\u00a0\u00a0\u00a0at com.sivalabs.ft.features.domain.feature.FeatureServiceTest.resetMocks(FeatureServiceTest.java:34)\n\u00a0\u00a0\u00a0\u00a0at java.base/java.lang.reflect.Method.invoke(Method.java:565)\n\u00a0\u00a0\u00a0\u00a0at java.base/java.util.ArrayList.forEach(ArrayList.java:1604)\n\u00a0\u00a0\u00a0\u00a0at java.base/java.util.ArrayList.forEach(ArrayList.java:1604)\n```\nFix the test."
Tools: ["get_file_text_by_path", "replace_text_in_file", "execute_run_configuration", "execute_terminal_command"]

Issue: "Update the Feature entity to replace the assignedTo field with a relationship to a Developer entity. Create database migration scripts to migrate existing assignedTo values into the new Developer table and establish the necessary foreign key constraint. Ensure that test data is consistent with these changes."
Tools: ["create_or_update_entity_attribute", "find_jpa_entities", "get_jpa_entity_by_name"]

Issue: "Add a 'user reactions' feature for the FeatureTracker application.\nAdd a `FeatureReaction` to track user likes and dislikes on features. Each `FeatureReaction` should has fields:\n - id (of type `Long`)\n - feature (of type `Feature`)\n - userId (of type `String`)\n - reactionType (of type `ReactionType`)\n - createdAt (of type `Instant`)\n - updatedAt (of type `Instant`)\nWith corresponding setters and getters.\nSupport `LIKE` and `DISLIKE` reactions (defined in `ReactionType` enum).\nEnsure each user can react only once per feature, and can update their reaction.\nAdd the FeatureReactionRepository that extends ListCrudRepository and has methods: `findByFeature(Feature feature)`, `findByUserId(String userId)`, `findByFeatureAndReactionType(Feature feature, ReactionType reactionType)` to fetch reactions by feature, user id and reaction type."
Tools: ["create_jpa_entity", "create_or_update_entity_attribute", "create_spring_data_repository"]

Issue: "Implement an optional parent-child relationship for the Release entity, allowing releases to reference a parent release. This feature enables modeling release hierarchies (e.g., a patch release as a child of a major release).\n\n1. Database Schema. In a new migration file:\n   - Add a `parent_id` column to the `releases` table (nullable, bigint)\n   - Add a foreign key constraint: `constraint fk_releases_parent_id foreign key (parent_id) references releases (id)`\n   - Consider adding an index on `parent_id` for query performance\n\n2. Entity Model:\n   - Add `parent` field to the `Release` entity (ManyToOne relationship)\n   - Add `children` collection to support bidirectional navigation (OneToMany relationship)\n   - Include proper JPA annotations\n\n3. API Specification:\n   - Add `parentCode` field (String, optional) to `CreateReleasePayload` to allow setting parent during creation\n   - Add `parentCode` field (String, optional) to `UpdateReleasePayload` to allow changing parent after creation\n   - Include `parentCode` field in `ReleaseDto` response model\n   - The parent should be referenced by the release **code** (not ID)\n\n4. Business Logic:\n   - When creating or updating a release with a `parentCode`, resolve the parent by code and set the relationship\n   - Validate that the parent release exists; throw an exception if not found\n   - Prevent a release from being its own parent (self-reference validation)\n   - Handle the case where `parentCode` is an empty string in updates (should remove the parent relationship)\n   - All the children releases should become orphans in case of parent deletion.\n\n5. API Endpoints:\n   - `POST /api/releases` - Accept optional `parentCode` in request body\n   - `PUT /api/releases/{code}` - Accept optional `parentCode` in request body. Should be able either add a parent release if `parentCode` exists or remove it if `parentCode` is not present.\n   - `DELETE /api/releases/{code}` - Remove the parent relationship if it exists \n   - `GET /api/releases/{code}` - Return `parentCode` in response if parent exists"
Tools: ["create_or_update_entity_attribute"]

Issue: "Change the feature handling to use developer ID instead of the assignedTo field. Update the CreateFeaturePayload and UpdateFeaturePayload to include developerId. Update all fields related to this change. Modify the FeatureService to set the developer based on developerId. Ensure that the FeatureDto includes developerId and adjust the FeatureMapper accordingly to map to and from the developer attributes."
Tools: ["create_or_update_entity_attribute"]

Issue: "Update the FeatureController to allow fetching features by product code in addition to release code. Ensure that the API requires either a product code or a release code, but not both. Implement the necessary service and repository methods to support this functionality. Change the FeatureDto to use the FeatureStatus enum instead of a String for status. Also, adjust the FeatureStatus enum to rename \"IN_DEVELOPMENT\" to \"IN_PROGRESS\"."
Tools: ["rename_refactoring"]

Issue: "Modify the existing Spring Boot application to use a database-backed user password authentication system instead of the current OAuth2 implementation. Create the necessary database schema, entities, and repositories to store user credentials and roles. Create an init script to create an admin user with `admin` username and `admin` password. Create ADMIN and USER predefined roles. Implement a UserDetailsService to convert database entities to Spring Security UserDetails objects. Configure a DaoAuthenticationProvider with BCryptPasswordEncoder. This change also involves removing all JWT-parsing logic from SecurityUtils and deleting the OAuth2 configuration properties from application.properties. Add a new SecurityIntegrationTest.java class to validate the new database authentication, verifying the BCryptPasswordEncoder, the DaoAuthenticationProvider, and the successful login of the 'admin' user."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "Set up login form user authentication using database storage with Spring Data JPA, including password encryption and UserDetailsService.\nImplement User entity with long id, username, password, and roles.\nCreate predefined users:\n- `admin` username, `admin123` password and `USER`, `ADMIN` roles\n- `user` username, `user123` password, and `USER` role"
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "Create a new Category entity in com.sivalabs.ft.features.domain.entities package to represent categories in the system. Ensure that the entity includes necessary fields such as id, name, description, parent category relationship, created by, and created at with Instant type. Implement the CategoryRepository interface in com.sivalabs.ft.features.domain.CategoryRepository package for database access. Update the Feature entity to include a relationship with the Category entity. Create the corresponding database table and migration script to support category creation and linking features to categories. Include test data for categories in the test SQL file with sample category with name bug and assign this category to test feature with id = 1.\nThe actual test data implementation creates several categories ('New Feature' as ID=1, 'Improvement' as ID=2, 'Bug Fix' as ID=3) and assigns the 'New Feature' category (ID=1) to the test feature with id = 1 ('IDEA-1'), rather than a category named 'bug'."
Tools: ["create_jpa_entity", "create_spring_data_repository", "create_or_update_entity_attribute"]

Issue: "Refactor Spring Security to rename a role `ROLE_ADMIN` to `ROLE_ADMINISTRATOR` across the application while preserving access control and data integrity."
Tools: ["search_in_files_by_text", "replace_text_in_file", "rename_refactoring", "find_files_by_name_keyword"]

Issue: "In order to modernise piggymetrics it would be nice to upgrade to Junit 5.\n\nSpring Boot started to support Junit 5 in Spring Boot 2.2, so it makes sense to also:\n\n- upgrade Spring Boot to 2.2.0.RELEASE, \n- upgrade Spring Cloud to Hoxton.RELEASE,\n- update Java version to 13\n- update JaCoCo maven plugin from 0.7.6 to 0.8.4 across all services"
Tools: ["get_project_dependencies"]

Issue: "- Replace synchronous RestTemplate with WebClient across microservices\n- Refactor repositories to use reactive CRUD operations\n- Update service implementations and test cases accordingly\n- Transition from traditional MVC to WebFlux stack for all APIs and testing frameworks\n- Remove unused synchronous methods, libraries, and dependencies"
Tools: ["get_project_dependencies", "find_jpa_entities", "find_spring_data_repositories", "get_jpa_entity_by_name"]

Issue: "Introduce the 'rating' field for reviews \n- Update the Review DTOs:\n   - Add field `rating` of type `int` to `Review` and `ReviewSummary`.\n   - Ensure constructors, getters/setters (or record fields) include `rating`.\n- Update the Review entity:\n   - Add field `rating` of type `int` with validation: min=1, max=5.\n   - Update constructors, getters, and setters to handle `rating`.\n- Update services:\n   - When creating Review and ReviewSummary objects in ProductCompositeServiceImpl, include the new `rating` field.\n- Generate database migration scripts:\n   - Add a new column 'rating' INT NOT NULL to the 'reviews' table\n   - Ensure existing records are handled correctly"
Tools: ["create_or_update_entity_attribute"]

Issue: "Review Service: enable reviews persisting. \n- Add a Spring Data JPA entity `ReviewEntity` with fields:\n   - productId (int)\n   - reviewId (int)\n   - author (String)\n   - subject (String)\n   - content (String)\n   - id (auto-generated primary key)\n   - version (int)\n   - Unique constraint on (productId, reviewId)\n- Add `ReviewRepository` extending `CrudRepository<ReviewEntity, Integer>`.\n- Create `ReviewService` REST API with:\n   - POST /review \u2192 creates a review\n   - DELETE /review?productId=X \u2192 deletes all reviews for product\n   - GET /review?productId=X \u2192 returns all reviews for product\n- Configure MySQL connection:\n   - DB name: `review-db`, user: `user`, password: `pwd`\n   - Use Hikari datasource with initialization timeout\n- Update docker-compose to launch MySQL container with healthcheck.\n- Include SQL init scripts for creating tables and sequences.\n- Add mapping utilities between `Review` API object and `ReviewEntity`."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "We need to ensure that:\n\n- `productId` / `reviewId` >= 0\n- `author` / `subject` / `content` are not blank\n- `content` is more that 50 characters"
Tools: ["create_or_update_entity_attribute"]

Issue: "We need to implement a ChatController to manage chats through the web UI using Spring MVC + Thymeleaf. The logic should be splitted with ChatService and ChatRepository.\n\nThe controller should provide:\n\tMain page (GET /)\n    \t\u2022\tDisplay a list of all chats.\n    \t\u2022\tRender template chat.html.\n\tShow chat (GET /chat/{chatId})\n\t\u2022\tDisplay all chats in the sidebar.\n\t\u2022\tDisplay the selected chat with its messages.\n\t\u2022\tRender template chat.html.\n\tCreate chat (POST /chat)\n\t\u2022\tAccept title from a form.\n\t\u2022\tCreate a new chat via ChatService.\n\t\u2022\tRedirect to the new chat page.\n\tDelete chat (DELETE /chat/{chatId})\n\t\u2022\tRemove chat by ID via ChatService.\n\t\u2022\tRedirect to main page /."
Tools: ["create_spring_data_repository"]

Issue: "Use WireMock to mock external API dependencies in your Spring Boot application for integration tests."
Tools: ["create_new_file", "replace_text_in_file", "get_file_text_by_path"]

Issue: "Update the API base path from `/api` to `/api/v1` in all relevant controllers, ensure that the response location headers reflect this change, and modify the OpenAPI documentation accordingly."
Tools: ["search_in_files_by_text", "replace_text_in_file"]

Issue: "Implement a multi-tenant security system that extends the existing Spring Security configuration. The implementation should parse tenant information from the `X-TENANT-ID` request header, validate tenant access during authentication, and enforce tenant-specific authorization rules. The solution should be minimally invasive to existing code while providing robust tenant isolation. Add predefined tenants `tenant-1` and `tenant-2`.\nCreate predefined users `user-1` for tenant `tenant-1` and `user-2` for tenant `tenant-2` with `password` passwords."
Tools: ["create_or_update_entity_attribute"]

Issue: "Create a custom servlet filter named RequestLoggingFilter that logs all incoming HTTP requests. Ensure it captures and logs the HTTP method and request URL, including the query string if present  in the following format: \"Incoming request: method requestUri(\\?queryString)?\". The filter should be implemented as a Spring component and included in the application's filter chain.\n\n- Use SLF4J logger with parameterized logging:\n  - With query: \"Incoming request: {} {}?{}\"\n  - Without query: \"Incoming request: {} {}\"\n- Log at INFO level"
Tools: ["create_new_file", "replace_text_in_file", "execute_terminal_command", "execute_run_configuration"]

Issue: "Create a Medication entity that uses a UUID primary key stored in a compact binary form (BINARY(16) or the dialect\u2019s binary equivalent) for space and performance efficiency. Use standard JPA configuration like UUID generation strategy and a binary mapping without tying the solution to a specific annotation."
Tools: ["create_jpa_entity", "create_or_update_entity_attribute", "create_spring_data_repository"]

Issue: "Create a new Prescription value object as an embeddable record and embed a collection of it within the Visit entity.\nEnsure that a Visit can persist and retrieve multiple Prescription entries correctly through standard JPA repository operations."
Tools: ["create_or_update_entity_attribute"]

Issue: "Refactor the `Pet` entity to include validation annotations for the `birthDate` and `type` fields. Remove the `PetValidator` and its related initialization in `PetController`. Instead, implement field validation directly in the controller methods by leveraging the validation annotations in the `Pet` entity. Ensure that the birth date validation checks if the date is in the past or present."
Tools: ["create_or_update_entity_attribute"]

Issue: "Configure app to run correctly as a GraalVM native image with working JPA and Flyway integration.\nThe goal is to achieve a fully working native image build where the application can start, apply database migrations, and perform CRUD operations through standard JPA repositories, without runtime reflection errors or missing resources.\nFlyway should fully replace legacy schema.sql and data.sql initialization scripts, and the migration-based schema creation."
Tools: ["get_file_text_by_path", "replace_text_in_file", "create_new_file", "execute_terminal_command"]

Issue: "Implement functionality to allow users to add and remove features from their favorites list. Create a RESTful API in `FavoriteFeatureController` for managing favorite features, including proper handling of duplicate favorites and validation of inputs. Ensure integration with the database by creating the necessary `FavoriteFeature` entity and repository, and handle any exceptions via a global exception handler. Additionally, set up a migration script to create the corresponding database table for favorite features."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "**Objective**\nDefine the data structure for feature dependencies and ensure they are correctly stored in the database with referential integrity.\n\n**Requirements**\n* **Feature Dependency Entity:** Create a database entity or model with the fields: `id`, `feature` (many-to-one to dependent feature using `feature.code`), `dependsOnFeature` (many-to-one to dependency feature using `feature.code`), `dependencyType` (`hard`/`soft`/`optional`), `createdAt`, and `notes`.\n* **Relationships:** Implement the bidirectional one-to-many relationship allowing any feature to have zero or more dependencies.\n* **Database Schema:** Ensure the database schema is updated to store dependencies and enforces referential integrity (e.g., a dependency cannot be created for a non-existent `featureCode`).\n* **CRUD Repository:** Implement `FeatureDependencyRepository` that supports nested property queries:\n  * `findByFeature_Code`\n  * `findByDependsOnFeature_Code`\n  * `findByFeature_CodeAndDependsOnFeature_Code`\n\n**Test Coverage**\n* Unit tests for the `FeatureDependency` entity/model.\n* Integration tests for database persistence (create, read, update, delete).\n* Tests to ensure database constraints (like foreign keys) prevent invalid data entry and enforce unique constraints preventing duplicate dependencies and self-dependencies.\n\n**Acceptance Criteria**\n* The `FeatureDependency` entity is correctly defined in the codebase and migrated to the database.\n* Dependencies can be successfully saved to, and retrieved from, the database.\n* The system prevents the creation of dependencies that reference non-existent features.\n* The `dependencyType` field is validated against the `DependencyType` enum."
Tools: ["create_jpa_entity", "create_or_update_entity_attribute", "create_spring_data_repository", "find_jpa_entities", "get_jpa_entity_by_name"]

Issue: "Update current AccountResource and implement Password Policy Validation and Strength Requirements. Create comprehensive password policy validation including:\n\n- Strength requirements: min 6 chars with letters, digits, and symbols\n- History checking: prevent current password reuse \n- Expiration management: 90-day policy with forced change\n\nIntegrate with existing UserService and MailService where needed. Ensure password policies apply to all password-related operations: user registration, password change, password reset."
Tools: ["create_jpa_entity", "create_or_update_entity_attribute", "find_jpa_entities", "get_jpa_entity_by_name"]

Issue: "Review Service: store a date when a review was written.\n\n- Add a `date` field of type `LocalDate` to both the `Review` API record and the `ReviewEntity` JPA entity.\n- The field must be non-nullable.\n- Update constructors, getters, setters, and mappers to include the `date` field.\n- Update database schema to add `date` column with NOT NULL constraint.\n- Ensure JSON serialization and deserialization support `LocalDate` in ISO format.\n- Update any relevant services or utility classes to handle the `date` field correctly."
Tools: ["create_or_update_entity_attribute"]

Issue: "Introduce MedicalCondition as a JPA entity identified by a composite key (condition_code, locale) using @IdClass(MedicalConditionId.class). \nDefine the composite key class MedicalConditionId with code and locale fields and proper equals/hashCode.\nAdd an element collection names to MedicalCondition, stored in a separate table with join columns matching the composite key.\nExpose MedicalConditionRepository extending CrudRepository for CRUD by the composite key.\nUpdate Visit to reference MedicalCondition with @ManyToOne and @JoinColumns that map to (condition_code, locale) so a visit can point to a specific condition variant."
Tools: ["create_jpa_entity", "create_or_update_entity_attribute", "create_spring_data_repository"]

Issue: "Build REST endpoints under /api for owners, pets, and visits that expose standard CRUD plus the following public behavior. \n- Owners support listing (with optional lastName search and 1-based pagination), fetching by ID, create (rejects payloads with id), update by path ID (400 on ID mismatch), and delete. \n- Pets are nested under owners for creation and listing (/owners/{ownerId}/pets) and also addressable by ID for read/update/delete (/pets/{petId}); creating a pet must return 201 with a Location to the new resource and respond with 409 if the owner already has a pet with the same name. \n- Visits are nested under pets (/pets/{petId}/visits) and addressable by ID; creating a visit requires a non-empty description and returns 201 with Location. \nAll endpoints return 404 for missing parents or resources and validate inputs via standard bean validation."
Tools: ["find_jpa_entities", "get_jpa_entity_by_name", "find_spring_data_repositories", "create_spring_data_repository"]

Issue: "Start the migration to a reactive stack by transitioning the persistence layer from JPA to R2DBC.\nAt this step, we keep the controllers blocking using Spring MVC. Migration to spring-webflux controllers will happen in a later phase.\n* Migrate JPA Entities to R2DBC Entities\n* Replace JpaRepository with R2dbcRepository"
Tools: ["find_jpa_entities", "get_jpa_entity_by_name", "find_spring_data_repositories", "create_spring_data_repository"]

Issue: "Since R2DBC does not provide lazy or eager loading like JPA, introduce a reactive service layer that explicitly loads related entities to preserve expected domain behavior.\nImplement reactive service classes for Owners, Pets, Visits, and Vets that handle relationship assembly manually using Mono and Flux.\nSpecifically, manually join Vet and Specialty entities via the vet_specialties bridge table, and ensure Owner and Pet entities maintain correct many-to-one and one-to-many relationships.\nThe service layer should return fully populated domain objects similar to JDBC-based logic. DTO-level mapping will be added in a later step."
Tools: ["find_spring_data_repositories", "create_spring_data_repository", "find_jpa_entities", "get_jpa_entity_by_name"]

Issue: "- Upgrade Spring Boot to the latest 3.4.x.\n- Align dependencies via Spring Boot BOM; remove explicit versions managed by the BOM.\n- Adjust application configuration properties for changes introduced in 3.1\u20133.4.\n- Migrate usages of deprecated APIs to their current equivalents.\n- Remove annotations that are redundant or obsolete in 3.4.x.\n- Migrate/replace annotations to the new APIs where applicable."
Tools: ["get_file_problems"]

Issue: "Create a new Medication JPA entity that uses a UUID as its primary key.\nThe UUID must be automatically generated when the entity is first saved to the database, and stored as a 36-character string with hyphens.\nProvide a repository interface for persistence operations."
Tools: ["create_jpa_entity", "create_spring_data_repository"]

Issue: "Task Description:\nDetect and resolve the N+1 select problem in the JPA entity relationships within the Petclinic REST API, such as the Owner-to-Pet-Visit relationship. Use SQL query logging to identify where multiple unnecessary queries are executed due to unoptimized fetching strategies. Refactor the data access code using techniques like JPQL join fetch, @EntityGraph, or @BatchSize to minimize the number of SQL queries, thus improving overall application performance. Verify the solution with tests. If other redundant (not only n+1) queries executed - fix that too.\n\nAcceptance Criteria:\n- The N+1 select problem is identified and reproducible in a real service or controller method.\n- SQL logs clearly demonstrate the presence and subsequent elimination of unnecessary queries.\n- Entity relationships are optimized using appropriate JPA/Hibernate techniques.\n- Tests confirm the correctness of optimized fetching and reduced query count."
Tools: ["get_jpa_entity_by_name", "find_jpa_entities", "find_spring_data_repositories"]

Issue: "**Task Description:**\nImplement comprehensive event deduplication and idempotency system for the Feature Service Spring Boot application. The system must augment feature event payloads with unique event identifiers and build dual-level deduplication: API-level idempotency to prevent duplicate database operations before they occur, and Event-level deduplication to prevent duplicate Kafka event processing in listeners. The implementation must use PostgreSQL database for persistent storage, ensuring thread-safe operation in distributed setups while maintaining data integrity and business logic correctness.\n\nFor Event Deduplication:\n* eventId field must be present on all operations: in CreateFeaturePayload, UpdateFeaturePayload payloads, as query parameter for DELETE operations, in all Feature Commands, and in all Feature Events (FeatureCreatedEvent, FeatureUpdatedEvent, FeatureDeletedEvent)\n* use database-first deduplication pattern: claim event in database first, then execute business logic for Event Listener using EventType.EVENT, for FeatureService using EventType.API\n* The universal table processed_event for storing processed events with the following fields: event_id , event_type(API or EVENT), processed_at , expires_at , result_data with Primary key: (event_id, event_type)\n* Create service that made idempotency checks for all event types\n* Minimizes database calls\n* Prevents race conditions\n\n**Acceptance Criteria:**\n* Feature events contain a unique identifier eventId that can be passed from external services.\n* Feature API endpoints return the same result for repeated requests with the same eventId\n* Feature Event listeners process each event only once\n* System works correctly in distributed environment (thread-safe)\n* Covered by tests with duplicate event simulation\n* Minimal performance overhead\n* Business Logic Logging: Event listeners must log business logic execution with specific format requirements:\n  * Log messages must start with \"EventListener business logic\" marker\n  * Must literaly contain operation type (\"Processing feature created\", \"Processing feature updated\", or \"Processing feature deleted\")\n  * Must include eventId for traceability and testing verification\n  * Example format: \"EventListener business logic: Processing feature created: FEATURE-123 - Feature Title (eventId: uuid-here)\""
Tools: ["create_jpa_entity", "create_spring_data_repository"]
