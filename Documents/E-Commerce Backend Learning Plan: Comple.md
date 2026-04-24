E-Commerce Backend Learning Plan: Complete Sprint Guide
Project Overview
Project Name: CommerceCraft
Tech Stack: Java 21, Spring Boot 3.3+, PostgreSQL 16, Redis 7, Docker
Architecture Pattern: Hexagonal Architecture
Duration: 12 weeks (6 sprints × 2 weeks)

SPRINT 0: Project Foundation & Architecture Setup
Duration: 1 Week | Theme: "Build strong foundations before writing business logic"

Sprint 0 Requirements
0.1 Multi-Module Project Setup

Requirement: Create a Maven multi-module project with proper separation of concerns

Requirement: Core domain module must have zero framework dependencies

Requirement: Infrastructure module should contain all external integrations

Requirement: API module should only handle HTTP concerns

Steps to Achieve:

Create parent POM with Spring Boot starter parent, define Java 21

Define modules: commerce-craft-core, commerce-craft-infrastructure, commerce-craft-api, commerce-craft-bom, commerce-craft-starter

Configure dependency management section for all third-party libraries

Set up checkstyle, PMD, SpotBugs Maven plugins with custom rulesets

Create .editorconfig at project root with consistent formatting rules

Configure Maven enforcer plugin to restrict dependency usage between modules

Verify core module has zero Spring dependencies by analyzing dependency tree

0.2 Domain Base Classes

Requirement: Create abstract base classes for all domain objects

Requirement: Every entity must have creation and modification timestamps

Requirement: Domain events must be registered and collected from aggregates

Requirement: Value objects must be immutable with proper equality

Steps to Achieve:

Create Identifier abstract class with UUID generation and validation

Design AggregateRoot<ID extends Identifier> with domain event collection

Implement BaseEntity with createdAt, updatedAt, version fields

Create DomainEvent interface with eventId, occurredAt, aggregateId

Implement ValueObject base with reflection-based equals/hashCode

Add JPA annotations only in infrastructure module, not in core

Create domain event publisher interface as output port

0.3 Hexagonal Architecture Ports

Requirement: All external dependencies must be accessed through ports

Requirement: Use cases should be defined as input port interfaces

Requirement: Repository interfaces should be defined as output ports

Requirement: No infrastructure code should leak into domain logic

Steps to Achieve:

Create input package for use case interfaces (e.g., CreateProductUseCase)

Create output package for repository and external service interfaces

Define ProductRepository interface in core module without JPA annotations

Create exception base classes: DomainException, EntityNotFoundException, BusinessRuleViolationException

Implement use case classes that depend only on port interfaces

Document each port with JavaDoc explaining its purpose and contract

Create service layer interfaces for complex business operations

0.4 Testing Infrastructure

Requirement: Integration tests must run against real database containers

Requirement: Tests must be isolated and independent of execution order

Requirement: Code coverage must be measured and enforced

Requirement: Architecture rules must be tested automatically

Steps to Achieve:

Add TestContainers dependency with PostgreSQL and Redis modules

Create abstract IntegrationTestBase configuring dynamic datasource

Implement schema-per-test-class strategy using Flyway migrations

Create test data builder classes with randomized but valid data

Configure JaCoCo plugin with 80% minimum coverage (100% for domain)

Add ArchUnit dependency and create base architecture test class

Write architecture test verifying module dependency rules

0.5 Code Quality Gates

Requirement: Architecture violations must fail the build

Requirement: Code style violations must fail the build

Requirement: Test coverage below threshold must fail the build

Steps to Achieve:

Create ArchUnit test checking core module has no Spring imports

Create ArchUnit test verifying ports are in correct packages

Add naming convention rules (repositories end with Repository, etc.)

Configure Maven failsafe plugin for integration tests

Set up build profile for CI with strict quality checks

Create pre-commit hook script for local development

Document architecture decisions in ADRs in /docs/adr folder

Sprint 0 Exit Criteria
✅ Multi-module project compiles successfully with mvn clean install

✅ All architecture tests pass with zero violations

✅ Integration test successfully creates and queries a test entity

✅ Code quality gates configured and verified locally

✅ Core module dependency tree shows zero Spring Framework imports

SPRINT 1: Product Catalog Service
Duration: 2 Weeks | Theme: "Master complex domain modeling and advanced persistence"

Sprint 1 Requirements
1.1 Category Management

Requirement: Categories must support unlimited nesting (hierarchical tree)

Requirement: Querying a category must return its full path

Requirement: Category deletion must handle child categories

Requirement: Category-product count must be accurate and cached

Steps to Achieve:

Design Category entity with parentId self-referencing foreign key

Implement Materialized Path pattern with path column (e.g., /1/5/12/)

Add database trigger to auto-update path on parent change

Create recursive CTE queries for ancestor/descendant retrieval

Implement category move operation with path update for all children

Add @Cacheable on category tree endpoint with 30-minute TTL

Create cache eviction on category create/update/delete operations

Design product count field updated via database trigger

Implement validation preventing circular parent references

Add endpoint for flat category list, tree structure, and breadcrumbs

1.2 Product Domain Model

Requirement: Products must have unique SKU across the system

Requirement: Products must support multiple variants (size, color, etc.)

Requirement: Product images must be ordered and have metadata

Requirement: Product attributes must be flexible (different per category)

Requirement: Products must follow a defined lifecycle state machine

Steps to Achieve:

Create Product aggregate with ProductId as embedded identifier

Design SKU value object with format validation (alphanumeric with hyphens)

Implement ProductVariant entity with option combination map (JSON)

Create ProductImage entity with sortOrder, url, altText, isPrimary

Design attributes field as JSONB column for category-specific data

Implement ProductStatus enum: DRAFT, ACTIVE, DISCONTINUED, ARCHIVED

Create state transition validation (cannot go from ARCHIVED back to ACTIVE)

Add Money value object with currency and amount (BigDecimal)

Implement Dimensions and Weight value objects

Create database index on SKU, category_id, and created_at columns

1.3 Product-Specific Pricing

Requirement: Products must support multiple price types

Requirement: Prices must be in different currencies

Requirement: Price changes must be tracked historically

Requirement: Effective pricing must consider date ranges

Steps to Achieve:

Create ProductPrice entity with priceType (RETAIL, WHOLESALE, PROMOTIONAL)

Add validFrom and validTo timestamp fields for time-bound pricing

Implement Currency as enum with ISO 4217 codes

Create composite index on (product_id, price_type, valid_from, valid_to)

Design pricing service that resolves effective price for a given date

Implement price overlap validation (two retail prices cannot overlap)

Add audit logging for price changes

Create endpoint for price history retrieval

Implement bulk price update with validation

Design promotional price activation/deactivation scheduler

1.4 Full-Text Search

Requirement: Product search must support relevance ranking

Requirement: Search must include product name, description, and category

Requirement: Autocomplete suggestions must appear as user types

Requirement: Faceted search must filter by category, price range, attributes

Steps to Achieve:

Add PostgreSQL tsvector column on products table

Create database trigger to auto-generate tsvector from name+description+category

Add GIN index on tsvector column for fast full-text search

Implement weighted search (name: A, description: B, category: C)

Create search function using ts_rank with normalization

Add pg_trgm extension for trigram-based similarity search

Implement autocomplete endpoint using word_similarity() function

Design faceted search using GROUP BY with COUNT on category, price ranges

Create combined search+filter+sort endpoint with pagination

Add search query logging for analytics and relevance tuning

1.5 Product API Design

Requirement: API must support versioning and backward compatibility

Requirement: Responses must include HATEOAS links

Requirement: Clients must be able to request specific response fields

Requirement: Pagination must support offset and cursor-based navigation

Steps to Achieve:

Create REST controllers at /api/v1/products and /api/v2/products

Implement Spring HATEOAS with EntityModel and CollectionModel

Add fields query parameter for sparse fieldset selection

Implement offset pagination with Pageable interface

Create cursor-based pagination using encoded cursor (productId + createdAt)

Add ETag support using @version field for conditional GET

Implement Last-Modified header based on updatedAt

Create comprehensive error responses following RFC 7807 Problem Details

Add OpenAPI annotations for automated documentation

Implement request validation groups (Create vs Update vs Patch)

1.6 Bulk Operations

Requirement: Support import of products from CSV/JSON

Requirement: Bulk operations must handle partial failures

Requirement: Import status must be trackable

Steps to Achieve:

Create bulk product creation endpoint accepting array of products

Implement async processing with @Async and CompletableFuture

Design import job tracking with jobId, status, progress, errors

Create validator that collects all errors without failing on first

Implement transaction per product (not entire batch)

Add rate limiting on bulk endpoints (max 1000 products per request)

Create CSV parser with header mapping configuration

Implement dry-run mode that validates without persisting

Design error report with row number, field, error message

Add webhook notification on import completion

Sprint 1 Exit Criteria
✅ CRUD operations for products, variants, images, and categories working

✅ Full-text search returns relevant results ranked by relevance

✅ Category tree queries execute under 100ms with caching

✅ Product state machine prevents invalid transitions

✅ API supports filtering, pagination, field selection, and HATEOAS links

✅ Bulk import handles 1000 products with partial error reporting

✅ 90%+ unit test coverage on domain logic, 80% overall

SPRINT 2: User Management & Security
*Duration: 2 Weeks | Theme: "Enterprise-grade security and compliance"*

Sprint 2 Requirements
2.1 User Registration & Profile

Requirement: Users must register with email verification

Requirement: Users must have multiple addresses (shipping, billing)

Requirement: Profile updates must trigger security review for sensitive changes

Requirement: User deletion must comply with data retention policies

Steps to Achieve:

Create User aggregate with embedded profile information

Design Email value object with format and MX record validation

Implement registration flow with email verification token (time-limited)

Create token generation with secure random, SHA-256 hashing for storage

Design Address value object with validation and geocoding hooks

Implement address management (add, update, delete, set default)

Create profile change tracking for sensitive fields (email, phone, password)

Implement notification service for security-sensitive changes

Design soft-delete mechanism for GDPR right-to-erasure

Create scheduled job for permanent deletion after retention period

2.2 Authentication System

Requirement: Must support JWT-based authentication

Requirement: Refresh tokens must be rotated on use

Requirement: Compromised tokens must be revocable

Requirement: Must support social login (Google, GitHub) as extension points

Steps to Achieve:

Generate RSA key pair for JWT signing (store private key securely)

Implement access token (15 min) and refresh token (7 days) generation

Create JWT builder with custom claims (userId, roles, permissions, tokenId)

Design refresh token rotation: issue new refresh token, invalidate old one

Implement token blacklist in Redis with expiration matching JWT TTL

Create token validation filter checking signature, expiry, and blacklist

Detect refresh token reuse (replay attack) and revoke all user tokens

Implement token revocation endpoint (logout)

Design OAuth2 abstraction layer with provider-specific adapters

Create /auth/me endpoint returning current user with permissions

2.3 Authorization System

Requirement: Must support role-based and permission-based access

Requirement: Roles must be hierarchical (ADMIN inherits MANAGER permissions)

Requirement: Permissions must be checkable at method level

Requirement: Resource ownership must be verifiable (own data only)

Steps to Achieve:

Design Role entity with parent reference for hierarchy

Create Permission entity (e.g., PRODUCT_CREATE, ORDER_READ, USER_DELETE)

Implement many-to-many relationship between roles and permissions

Create permission inheritance resolver (traverse role hierarchy)

Design custom @RequirePermission annotation for controller methods

Implement SpEL-based permission evaluator for complex rules

Create ownership checker service (does this user own this order?)

Add method security with @PreAuthorize("hasPermission(#orderId, 'ORDER_READ')")

Implement permission caching with invalidation on role changes

Design dynamic permission groups for tenant/team-based access

2.4 Multi-Factor Authentication

Requirement: MFA must be optional but encouraged

Requirement: Must support TOTP (Google Authenticator)

Requirement: Backup codes must be provided and one-time use

Requirement: Trusted devices must be remembered (30 days)

Steps to Achieve:

Implement TOTP secret generation per user (Base32 encoded)

Create QR code generation for easy setup (Google Charts or ZXing library)

Design TOTP verification with time-step window (allow 1 step drift)

Generate 10 backup codes, hash with bcrypt, store hashes only

Implement backup code verification and one-time use (mark used)

Create trusted device token (signed JWT stored in cookie, 30-day expiry)

Add step-up authentication requirement for sensitive operations

Implement failed MFA attempt tracking with account lockout

Design MFA enrollment flow (verify before enabling)

Create MFA reset process (requires email verification + support)

2.5 Brute Force Protection

Requirement: Failed login attempts must be rate-limited

Requirement: Account lockout must be temporary with exponential backoff

Requirement: Suspicious activity must generate alerts

Steps to Achieve:

Implement login attempt tracking in Redis (key: login:attempts:{username})

Design progressive lockout: 5 fails → 15 min, 10 fails → 1 hour, 15 → 24 hours

Create IP-based rate limiting for login endpoint (20 per minute per IP)

Implement account lockout notification to user email

Design suspicious activity detection (new device, new location, unusual time)

Create security event publishing to analytics/alerting system

Add captcha requirement after 3 failed attempts

Implement audit logging for all authentication events

Design admin endpoint to unlock accounts

Create monitoring dashboard for failed login metrics

2.6 Data Privacy & GDPR

Requirement: Users must be able to export all their data

Requirement: Account deletion must cascade appropriately

Requirement: Data retention policies must be enforced

Requirement: PII access must be audit-logged

Steps to Achieve:

Create data export service compiling user data from all aggregates

Design export format as machine-readable JSON

Implement async export with notification when ready

Create anonymization service replacing PII with random data

Design retention policy enum: RETAIN, ANONYMIZE_AFTER, DELETE_AFTER

Implement scheduled job processing retention policies

Create audit log for all PII access with reason tracking

Add consent management (marketing, analytics, third-party sharing)

Implement consent version tracking for proof of consent

Design privacy policy acceptance timestamp tracking

Sprint 2 Exit Criteria
✅ Registration with email verification working

✅ JWT authentication with refresh token rotation implemented

✅ Role hierarchy and permission checks functioning at method level

✅ MFA with TOTP and backup codes fully functional

✅ Brute force protection preventing rapid login attempts

✅ Data export and account deletion complying with GDPR principles

✅ All authentication events audit-logged

SPRINT 3: Shopping Cart & Session Management
Duration: 2 Weeks | Theme: "Distributed state management and cache strategies"

Sprint 3 Requirements
3.1 Cart Domain Model

Requirement: Cart must support guest and authenticated users

Requirement: Cart must enforce maximum items limit (100 items)

Requirement: Cart item quantity must respect available stock

Requirement: Cart must calculate totals in real-time

Requirement: Cart must have TTL and auto-cleanup

Steps to Achieve:

Create Cart aggregate with CartId and optional UserId

Design CartItem with productId, variantId, quantity, addedAt

Implement cart capacity validation (max unique items, max quantity per item)

Create CartPricingService calculating subtotal, tax, discounts, total

Design cart state machine: ACTIVE, ABANDONED, CONVERTED, EXPIRED

Implement stock availability check on item add and update

Create price snapshot on item addition (don't recalculate from live price)

Design cart serialization for Redis storage (JSON with GZIP)

Add cart-level metadata (coupon codes, notes, gift wrapping)

Implement cart validation rules (minimum order amount, restricted items)

3.2 Dual Storage Strategy

Requirement: Active carts must live in Redis for fast access

Requirement: Carts must be persisted to PostgreSQL for recovery

Requirement: Redis failure must not result in cart loss

Requirement: Cart sync must handle race conditions

Steps to Achieve:

Configure Redis with persistence (AOF + RDB) and maxmemory-policy

Implement Redis repository for cart storage with TTL (30 minutes for guests, 7 days for users)

Create PostgreSQL cart table mirroring cart structure

Design write-through cache: write to DB, then update Redis

Implement cache-aside read: check Redis, fallback to DB, populate Redis

Create scheduled sync job (every 5 minutes) for dirty carts

Implement cart versioning for conflict detection

Design merge strategy for Redis+DB reconciliation

Add circuit breaker for Redis operations (fallback to DB only)

Create cart recovery process on application restart

3.3 Cart Merging (Guest to User)

Requirement: Guest cart must merge with user cart on login

Requirement: Duplicate items must be handled intelligently

Requirement: Merging conflicts must be reported to user

Steps to Achieve:

Create merge service that combines guest and user carts

Implement duplicate detection by product+variant combination

Design merge strategies: SUM_QUANTITY, KEEP_MAX, KEEP_USER, KEEP_GUEST

Add merge preview endpoint (show what will change before committing)

Create conflict resolution API where user chooses strategy per item

Implement merged cart pricing recalculation

Add validation post-merge (stock check, max items)

Design guest cart expiration after merge (mark as MERGED)

Create notification if merge couldn't add all items due to stock

Implement audit log of merge operations

3.4 Concurrent Modification Handling

Requirement: Multiple tabs/devices modifying cart must not cause data loss

Requirement: Version conflicts must be detected and reported

Requirement: Cart operations must be atomic

Steps to Achieve:

Add version field to Cart entity (optimistic locking)

Implement version check on cart update operations

Return 409 Conflict with current cart state on version mismatch

Add ETag header with version hash for conditional requests

Create retry mechanism with exponential backoff on client side

Implement Redis WATCH command for transactional updates

Design last-write-wins strategy for non-critical metadata updates

Add distributed lock on cart during checkout process

Create conflict resolution examples in API documentation

Implement cart operation idempotency keys

3.5 Abandoned Cart Recovery

Requirement: Cart abandonment must be detected after 2 hours of inactivity

Requirement: Recovery emails must be sent at intervals (2h, 24h, 72h)

Requirement: Users must be able to opt-out of recovery emails

Requirement: Abandoned cart statistics must be tracked

Steps to Achieve:

Create scheduled job running every 15 minutes checking cart activity

Design abandonment detection: lastModifiedAt < 2 hours ago AND status = ACTIVE

Implement recovery email service with product details and images

Create recovery email schedule: first after 2h, second after 24h, third after 72h

Add opt-out flag on cart and user profile

Design deep-link generation returning user directly to cart

Implement conversion tracking (did user complete order after recovery email?)

Create A/B testing capability for email templates

Add analytics events for cart stages (created, abandoned, recovered, converted)

Design dashboard showing abandonment rate and recovery rate

3.6 Cart Performance Optimization

Requirement: Cart read operations must complete under 10ms

Requirement: Cart pricing must not block on external services

Requirement: Bulk cart operations must be supported

Steps to Achieve:

Implement Redis pipelining for batch cart operations

Create cart summary projection (id, itemCount, total) for list views

Design price calculation caching with 5-minute TTL

Implement lazy loading of cart items on list endpoints

Add database index on user_id and status for cart queries

Create cart preload on user login (background async operation)

Implement connection pooling optimization for Redis

Design cart size monitoring and alerts

Add performance metrics for cart operations

Create load test scripts simulating 1000 concurrent carts

Sprint 3 Exit Criteria
✅ Cart operations working with Redis for sub-10ms reads

✅ Dual storage ensures no cart data loss on Redis failure

✅ Guest-to-user cart merging handles all conflict scenarios

✅ Concurrent modification detection prevents lost updates

✅ Abandoned cart detection and recovery emails triggering

✅ Cart items over 100 or exceeding stock properly rejected

✅ Cart conversion rate metrics being collected

SPRINT 4: Order Management System
*Duration: 2 Weeks | Theme: "Transactional integrity and event-driven workflows"*

Sprint 4 Requirements
4.1 Order Domain Model

Requirement: Order must be an immutable snapshot of cart at purchase time

Requirement: Order must have unique human-readable order number

Requirement: Order must track complete lifecycle (15+ states)

Requirement: Order lines must preserve price and product details at time of order

Requirement: Order must track all modifications in audit log

Steps to Achieve:

Create Order aggregate with complete shipping/billing address snapshot

Design OrderNumber generator: prefix + date + sequence + checksum

Implement sequence number generation using database sequence or Redis

Create OrderLine entity with product snapshot (name, SKU, price, image URL)

Design OrderStatus enum: PENDING, CONFIRMED, PAYMENT_PENDING, PAID,
PROCESSING, READY_TO_SHIP, SHIPPED, IN_TRANSIT, DELIVERED,
CANCELLED, REFUNDED, PARTIALLY_REFUNDED, ON_HOLD, FAILED

Implement status transition validation matrix

Create OrderHistory entity recording every status change with timestamp and user

Design Money snapshots for all financial fields (subtotal, tax, shipping, discount, total)

Add order-level metadata (coupon used, gift message, delivery instructions)

Implement soft-delete for cancelled orders (retain for audit)

4.2 Order Creation Process

Requirement: Order creation must be atomic (all-or-nothing)

Requirement: Inventory must be reserved immediately on order creation

Requirement: Failed order creation must not reserve inventory

Requirement: Order creation must be idempotent (duplicate prevention)

Steps to Achieve:

Create OrderService.createOrder() with @Transactional annotation

Implement idempotency key check at start of order creation

Design inventory reservation service (decrement available, increment reserved)

Validate stock availability before reservation (optimistic lock check)

Calculate final prices including promotions and taxes

Create order snapshot from cart items (copy, don't reference)

Generate order number after successful persistence

Clear or mark cart as converted after successful order

Publish OrderCreatedEvent within transaction boundary

Implement rollback handler releasing inventory reservations

4.3 Order State Machine

Requirement: Every status transition must be validated

Requirement: Certain transitions must require specific roles/permissions

Requirement: State changes must trigger side effects (emails, inventory updates)

Requirement: Status change history must be complete and immutable

Steps to Achieve:

Implement state machine using Spring State Machine or custom implementation

Define all valid transitions in configuration:

PENDING → CONFIRMED, CANCELLED

CONFIRMED → PAYMENT_PENDING, CANCELLED

PAYMENT_PENDING → PAID, FAILED, CANCELLED

PAID → PROCESSING

PROCESSING → READY_TO_SHIP, ON_HOLD, CANCELLED

READY_TO_SHIP → SHIPPED

SHIPPED → IN_TRANSIT

IN_TRANSIT → DELIVERED

DELIVERED → REFUNDED (within 30 days)

Create guard conditions (e.g., can't ship without payment confirmation)

Implement transition actions (update inventory, send email, release holds)

Add permission checks on transitions (only warehouse can mark SHIPPED)

Create bulk status update for batch processing

Design undo/rollback mechanism for accidental transitions

Implement scheduled auto-cancellation for unpaid orders after 30 minutes

Add status aging monitoring (orders stuck in state too long)

Create admin override for exceptional manual interventions

4.4 Outbox Pattern for Reliable Events

Requirement: Domain events must be published reliably (no lost events)

Requirement: Event publishing must survive application crashes

Requirement: Events must be published in order per aggregate

Requirement: Duplicate event delivery must be tolerated by consumers

Steps to Achieve:

Create outbox table: id, aggregateId, eventType, payload (JSONB),
createdAt, publishedAt, retryCount, status (PENDING, PUBLISHED, FAILED)

Insert outbox record in same transaction as domain changes

Create OutboxPublisher scheduled job polling every 1 second

Implement publisher with batch fetch (100 events at a time)

Design retry strategy: 3 retries with 1s, 5s, 30s delays

Move to dead letter queue after max retries

Create dead letter handler with admin UI for manual retry

Implement outbox cleanup job (delete events older than 7 days)

Add metrics: outbox size, publish rate, failure rate, lag

Design idempotent event consumers using eventId deduplication

4.5 Order Fulfillment Workflow

Requirement: Orders must support partial fulfillment

Requirement: Multiple shipments per order must be tracked

Requirement: Fulfillment must integrate with inventory management

Requirement: Shipping labels must be generatable via carrier API

Steps to Achieve:

Create Shipment entity with tracking number and carrier details

Design fulfillment service that splits order into shipments

Implement shipping carrier abstraction (FedEx, UPS, DHL adapters)

Create label generation service with retry on API failure

Design package dimensions and weight calculation

Implement tracking number update webhook

Create delivery confirmation process

Design partial cancellation handling

Implement backorder handling for out-of-stock items

Add fulfillment SLA monitoring and alerts

4.6 Order Query & Reporting

Requirement: Orders must be searchable by multiple criteria

Requirement: Order timeline must show all events chronologically

Requirement: Order metrics must be available for dashboards

Requirement: Order export must support CSV and PDF formats

Steps to Achieve:

Create order search with filters: status, date range, amount range, customer

Design order timeline endpoint returning all history entries

Implement order statistics: daily revenue, average order value, order count

Create sales by product/category aggregation

Design PDF invoice generation with customizable templates

Implement order export with selected fields and filters

Add pagination with cursor for large order lists

Create order status summary dashboard data

Design real-time order notification for admin dashboard

Implement order search indexing for performance

Sprint 4 Exit Criteria
✅ Order creation atomically reserves inventory and persists order

✅ State machine enforces all valid and invalid transitions

✅ Outbox pattern ensures zero event loss (verified with chaos testing)

✅ Partial fulfillment supporting multiple shipments per order

✅ Order search returning results under 200ms with filters

✅ Invoice generation producing correct PDF documents

✅ Idempotency preventing duplicate order creation

SPRINT 5: Payment Integration & Financial System
Duration: 2 Weeks | Theme: "External integration resilience and financial accuracy"

Sprint 5 Requirements
5.1 Payment Provider Abstraction

Requirement: Multiple payment providers must be supported (Stripe, PayPal)

Requirement: Provider switching must be configuration-based

Requirement: Provider-specific features must be abstracted

Requirement: Provider health must be monitored

Steps to Achieve:

Create PaymentProvider interface with methods: authorize, capture, refund, void

Implement Stripe adapter using Stripe SDK

Implement PayPal adapter using PayPal REST SDK

Create provider factory selecting provider based on configuration

Design payment method types: CARD, BANK_TRANSFER, DIGITAL_WALLET, BNPL

Implement provider configuration (API keys, webhook secrets from vault)

Add provider health check endpoints

Create provider load balancing/failover strategy

Design mock provider for testing

Implement provider-specific error mapping to domain exceptions

5.2 Payment Processing

Requirement: Payment authorization must precede capture

Requirement: Payment lifecycle must be: AUTHORIZED → CAPTURED → SETTLED

Requirement: Failed payments must be retryable

Requirement: Duplicate payment submissions must be prevented

Requirement: Payment must be atomic with order status update

Steps to Achieve:

Create Payment entity with status tracking

Implement payment authorization flow (validate card, hold funds)

Design capture strategy: automatic on order confirmation or manual

Create idempotency using idempotencyKey stored and validated

Implement retry logic with exponential backoff for transient failures

Design payment status mapping: PENDING, AUTHORIZED, CAPTURED, FAILED,
REFUNDED, PARTIALLY_REFUNDED, VOIDED, EXPIRED

Create payment status synchronized with order status

Implement webhook handler for asynchronous payment confirmations

Add payment reconciliation cron job (compare with provider records)

Design payment timeout (release authorization after 7 days if not captured)

5.3 Refund Management

Requirement: Full and partial refunds must be supported

Requirement: Refunds must be traceable to original payment

Requirement: Refund amount must not exceed captured amount

Requirement: Multiple refunds per payment must be allowed

Steps to Achieve:

Create Refund entity linked to Payment entity

Implement refund validation (amount check, time limits)

Design refund processing via payment provider

Create partial refund calculation for multi-item orders

Implement refund-idempotency separate from payment-idempotency

Add refund reason codes for analytics

Create refund notification to customer

Design refund reconciliation process

Implement refund reporting (daily, weekly totals)

Add refund fraud detection rules

5.4 Financial Data Integrity

Requirement: All monetary calculations must use BigDecimal

Requirement: Financial data must never be modified, only appended

Requirement: Every financial transaction must have audit trail

Requirement: Data integrity must be verifiable via checksums

Steps to Achieve:

Enforce BigDecimal with MathContext(DECIMAL64) for all monetary operations

Create immutable financial transaction table (INSERT only, no UPDATE/DELETE)

Design audit trail with before/after snapshots for any necessary corrections

Implement row-level checksum using hash of all financial columns

Create daily financial reconciliation process

Design correction entries (not modification) with reason and approval

Implement financial report generation with immutable audit log

Add tamper detection by verifying checksum chains

Create financial data archival to append-only storage

Design retention policy for financial records (7 years minimum)

5.5 Resilience & Fault Tolerance

Requirement: Payment provider failures must not crash the application

Requirement: Circuit breaker must prevent cascading failures

Requirement: Payment timeouts must have sensible defaults

Requirement: Payment queue must handle traffic spikes

Steps to Achieve:

Implement Resilience4j CircuitBreaker for each payment provider

Configure circuit breaker: 50% failure rate threshold, 10s wait duration

Create Retry with exponential backoff: 3 attempts, 100ms initial, 2x multiplier

Design TimeLimiter: 30s timeout for authorization, 60s for capture

Implement Bulkhead pattern limiting concurrent payment calls to 10

Create fallback mechanism: queue payment for retry, notify admin

Design rate limiter respecting provider API limits

Implement graceful degradation (accept order, process payment async)

Add payment processing dead letter queue

Create dashboard showing circuit breaker states and failure rates

5.6 Payment Security

Requirement: PCI DSS compliance considerations (tokenization)

Requirement: Never store raw card numbers

Requirement: All payment communications must be encrypted

Requirement: Payment webhooks must be verified

Steps to Achieve:

Use payment provider tokenization (never handle raw card data)

Store only payment tokens and last 4 digits

Implement webhook signature verification (Stripe webhook signing secret)

Create secure webhook endpoint with IP whitelist (optional)

Design sensitive data masking in logs and database

Implement encryption at rest for stored payment tokens

Add audit logging for all payment data access

Create regular security scan for payment modules

Implement rate limiting on payment endpoints

Design fraud detection integration points

Sprint 5 Exit Criteria
✅ Payment authorization and capture working with Stripe/PayPal

✅ Idempotency prevents duplicate charges

✅ Refunds (full and partial) processing correctly

✅ Circuit breaker opens on provider failure and closes after recovery

✅ Financial calculations accurate to decimal precision

✅ Immutable audit trail recording all financial operations

✅ Payment reconciliation matching provider records

✅ Zero raw card data stored, all interactions tokenized

SPRINT 6: Production Readiness & Operations
Duration: 2 Weeks | Theme: "From development to production excellence"

Sprint 6 Requirements
6.1 Observability

Requirement: All services must expose health metrics

Requirement: Distributed tracing must work across all operations

Requirement: Business metrics must be collected (orders/minute, revenue/hour)

Requirement: Error rates and response times must be monitored

Requirement: Logs must be structured and searchable

Steps to Achieve:

Add Spring Boot Actuator with health, metrics, info endpoints

Implement custom health indicators for database, Redis, payment providers

Configure Micrometer with Prometheus registry for metrics

Add distributed tracing with Micrometer Tracing and Zipkin/Jaeger

Create custom business metrics: orders.created, payments.processed, etc.

Implement structured logging with Logstash JSON encoder

Add MDC context with correlationId, userId, sessionId, requestId

Create Grafana dashboards for business and technical metrics

Configure alerting rules: error rate > 1%, p99 latency > 1s, cart abandonment > 80%

Implement log sampling strategy for high-volume endpoints

6.2 Caching Strategy

Requirement: Multi-level caching must be implemented (L1 in-memory, L2 Redis)

Requirement: Cache invalidation must be immediate on data change

Requirement: Cache stampede must be prevented for hot keys

Requirement: Cache hit ratio must be monitored

Steps to Achieve:

Configure Caffeine for L1 local cache (small, fast, frequently accessed)

Configure Redis for L2 distributed cache (larger, shared across instances)

Implement cache decorator with L1 → L2 → Database fallback

Design cache regions: products (1h TTL), categories (2h), user profiles (30m)

Create cache eviction on entity update (hook into service layer events)

Implement cache warming on application startup (preload popular products)

Prevent cache stampede with probabilistic early recomputation

Add cache metrics: hit rate, miss rate, eviction count, load time

Create admin endpoint to clear specific cache regions

Design null cache for cache penetration attacks

6.3 API Performance Optimization

Requirement: Response times must be under 200ms p95 for reads

Requirement: Response compression must be enabled

Requirement: Connection pooling must be optimized

Requirement: N+1 query problem must be eliminated

Steps to Achieve:

Enable GZIP compression for responses > 1KB

Configure Tomcat/database connection pool sizing (test under load)

Implement @EntityGraph and JOIN FETCH to avoid N+1 queries

Add pagination limit enforcement (max 100 items per page)

Create response caching with ETags for rarely changed resources

Implement async processing for long-running operations

Design query optimization using EXPLAIN ANALYZE

Add database index analysis and missing index detection

Create API performance test suite with k6 or JMeter

Implement HTTP/2 for connection multiplexing

6.4 Rate Limiting

Requirement: API must be protected from abuse

Requirement: Rate limits must be per-user and per-IP

Requirement: Rate limit headers must inform clients

Requirement: Premium users must have higher limits

Steps to Achieve:

Implement token bucket algorithm using Redis

Create rate limit tiers: anonymous (10 req/s), user (100 req/s), admin (1000 req/s)

Design rate limit configuration per endpoint category

Add response headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

Implement rate limit exceeded response with 429 status and Retry-After

Create tier upgrade on user membership level change

Add rate limit bypass for critical internal services

Design burst allowance (short bursts above sustained limit)

Implement rate limit monitoring and alerting

Create rate limit testing tools for load testing

6.5 Database Migration & Versioning

Requirement: Database schema must be version-controlled

Requirement: Migrations must be repeatable and idempotent

Requirement: Rollback strategy must exist for failed migrations

Requirement: No data loss during migrations

Steps to Achieve:

Configure Flyway with versioned migrations (V1__init.sql)

Create repeatable migrations for views/functions (R__product_search.sql)

Implement migration naming convention: V{version}__{description}.sql

Design backward-compatible migrations (add columns before dropping old ones)

Create migration test verifying schema state after all migrations

Implement Flyway callback for custom logic (seed data, validation)

Design blue-green deployment migration strategy

Add migration lock timeout configuration

Create migration dry-run mode for testing

Document rollback procedures for each migration type

6.6 Docker & Deployment

Requirement: Application must be containerized

Requirement: Multi-stage builds must optimize image size

Requirement: Docker Compose must run entire stack locally

Requirement: Graceful shutdown must handle in-flight requests

Steps to Achieve:

Create Dockerfile with multi-stage build (build, package, run)

Optimize base image (eclipse-temurin:21-jre-alpine for runtime)

Configure JVM options: heap sizing, GC tuning, container awareness

Implement graceful shutdown hook (30s timeout, complete in-flight requests)

Create health check endpoint used by Docker healthcheck

Design docker-compose.yml with PostgreSQL, Redis, app service

Implement wait-for-it script for dependency ordering

Create environment-specific Compose files (dev, staging, prod)

Add resource limits on containers

Design log aggregation using Docker logging drivers

6.7 Configuration Management

Requirement: Configuration must be external to application

Requirement: Secrets must never be in code or images

Requirement: Configuration changes must not require redeployment

Requirement: Different environments must use different configurations

Steps to Achieve:

Externalize all configuration to application.yml with Spring profiles

Create profile-specific configs: dev, staging, prod

Implement environment variable overrides for sensitive values

Design secrets management (Docker secrets, or reference to vault)

Add @ConfigurationProperties for type-safe configuration

Implement configuration validation on startup

Create encrypted property support with Jasypt

Design feature flags using configuration properties

Add configuration documentation in README

Create configuration audit log

6.8 API Documentation

Requirement: All endpoints must be documented

Requirement: Documentation must include request/response examples

Requirement: Authentication requirements must be clear

Requirement: Error responses must be documented

Steps to Achieve:

Add SpringDoc OpenAPI dependency

Annotate all controllers with operation descriptions

Document all request parameters, bodies, and response types

Add authentication documentation (Bearer token, scopes required)

Create example requests and responses for each endpoint

Document error codes and problem detail responses

Generate OpenAPI spec at /v3/api-docs

Add Swagger UI at /swagger-ui.html

Create Postman collection export from OpenAPI spec

Maintain CHANGELOG for API versioning changes

6.9 Load Testing & Benchmarking

Requirement: Critical paths must be load tested

Requirement: Performance baselines must be established

Requirement: Bottlenecks must be identified and documented

Requirement: Scaling limits must be known

Steps to Achieve:

Identify critical user journeys (search → view product → add to cart → checkout)

Create k6 load test scripts simulating realistic user behavior

Design test scenarios: smoke (1 user), load (100 concurrent), stress (1000 concurrent)

Implement ramp-up patterns and realistic think times

Add performance assertions (p95 < 200ms, error rate < 1%)

Create load test environment identical to production

Run soak test over 24 hours to detect memory leaks

Document baseline metrics for each endpoint

Identify and document database slow queries

Create load test report template

6.10 Production Checklist

Requirement: Application must pass production readiness review

Requirement: Runbook must exist for common operational tasks

Requirement: Backup and restore procedures must be tested

Requirement: Disaster recovery plan must be documented

Steps to Achieve:

Create production readiness checklist document

Implement database backup schedule (daily full, hourly incremental)

Test backup restore procedure and document

Design application restart procedure (rolling restart)

Create incident response runbook (who to call, what to check)

Implement dead letter queue monitoring and alerting

Add circuit breaker manual reset capability

Create capacity planning document

Design disaster recovery process with RTO/RPO targets

Document monitoring dashboards and alert meanings

Sprint 6 Exit Criteria
✅ Health checks and metrics visible in Grafana dashboard

✅ Distributed tracing showing complete request flows

✅ Two-level caching with >80% hit rate on product reads

✅ API p95 response times under 200ms under load

✅ Rate limiting preventing abuse with proper headers

✅ Docker Compose starting full stack with single command

✅ API documentation complete and accurate

✅ Load test results documented with baselines

✅ Backup/restore tested and documented

✅ Production runbook written and reviewed

Continuous Learning Throughout All Sprints
Daily Practices
Write failing test first, then implement (strict TDD)

Refactor after each feature (red-green-refactor cycle)

Document design decisions in Architecture Decision Records

Review own code as if reviewing a colleague's code

Weekly Reviews
Run all tests and ensure green

Check code coverage report for gaps

Run ArchUnit tests to catch architectural drift

Review performance of endpoints modified that week

Sprint Retrospective (End of Each Sprint)
What new pattern did I learn?

What would I do differently?

What technical debt did I accumulate?

What should I research deeper?

Advanced Challenges (Pick One Per Sprint)
Implement the same feature using WebFlux (reactive) and compare

Rewrite a synchronous flow using CompletableFuture for async

Add GraalVM native compilation support

Implement GraphQL endpoint alongside REST for one module

Create Kafka event streaming as alternative to outbox polling

Add CDC (Change Data Capture) with Debezium

Implement SAGA orchestration with temporal.io or Camunda

Add multi-tenancy support (database per tenant)

Implement read model projections for CQRS

Create custom Spring Boot starter library

This complete plan ensures that by the end, you've built a production-grade e-commerce backend while learning enterprise patterns, performance optimization, and operational excellence.

This response is AI-generated and for reference purposes only.
