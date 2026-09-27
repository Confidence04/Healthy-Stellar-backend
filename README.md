# Healthy-Stellar-backend

NestJS Backend Documentation - Decentralized Healthcare System
Comprehensive documentation for the NestJS backend that interfaces with Stellar Soroban smart contracts for the decentralized healthcare management system.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Local Development with Docker](#local-development-with-docker)
- [Installation & Setup](#installation--setup)
- [Configuration](#configuration)
- [Scripts Reference](#scripts-reference)
- [Core Modules](#core-modules)
- [API Endpoints](#api-endpoints)
- [Postman Collection](#postman-collection)
- [Database Schema](#database-schema)
- [Medical Records System](#medical-records-system)
- [Authentication & Authorization](#authentication--authorization)
- [Stellar Integration](#stellar-integration)
- [Error Handling](#error-handling)
- [Testing](#testing)
- [Deployment](#deployment)

## Architecture Overview

The NestJS backend serves as the application layer between the frontend and Stellar blockchain, providing:

- RESTful API endpoints for client applications
- Off-chain data caching and indexing
- User authentication and session management
- Encryption/decryption of sensitive health data
- Event listening and blockchain synchronization
- Business logic orchestration
- File upload and storage management

## Project Structure

### Entry points

| File | Purpose |
|---|---|
| `src/main.ts` | HTTP API process — bootstraps the NestJS app, applies global middleware, starts listening |
| `src/worker.ts` | BullMQ worker process — runs queue processors without exposing an HTTP port |
| `src/worker.module.ts` | NestJS module that wires only the queue processors (no HTTP layer) |
| `packages/sdk/` | Auto-generated TypeScript client SDK published to npm |

---

### Module map

Modules are grouped below by domain. Each entry is a top-level directory under `src/`.

#### Clinical

| Directory | Description |
|---|---|
| `patients/` | Patient registry, demographics, geo-restrictions, notification preferences |
| `medical-records/` | Core medical records with version control, consent, clinical notes, attachments |
| `records/` | Versioned record storage with IPFS anchoring and Stellar event-sourcing |
| `pharmacy/` | Drugs, prescriptions, drug interactions, recalls, inventory, controlled substances |
| `laboratory/` | Lab orders, results, specimens, equipment, quality control |
| `appointments/` | Scheduling, consultations, doctor availability |
| `diagnosis/` | Diagnosis management and ICD code handling |
| `treatment-planning/` | Care plans, clinical guidelines, decision-support alerts |
| `medication-administration/` | MAR (Medication Administration Record), barcode verification, adverse reactions |
| `infection-control/` | Outbreak tracking, isolation protocols, antibiotic resistance surveillance |
| `emergency-operations/` | Triage, rapid-response teams, critical care coordination |
| `emergency-medical-info/` | Break-glass emergency access to patient data |
| `pathology/` | Histology, cytology, genetic testing, digital pathology |
| `hospital-registry/` | Hospital and facility configuration |

#### Billing & Finance

| Directory | Description |
|---|---|
| `billing/` | Claims, invoices, insurance, payment processing |

#### Blockchain & Data Integrity

| Directory | Description |
|---|---|
| `stellar/` | Stellar Horizon / Soroban SDK wrapper, transaction retry/recovery |
| `stellar-stream/` | Real-time Stellar event streaming and re-indexing |
| `ledger-reconciliation/` | Detects discrepancies between the DB and on-chain state |
| `reconciliation/` | General data reconciliation jobs |
| `blockchain/` | Low-level blockchain utilities and abstract contract interface |
| `event-store/` | Append-only event-sourcing store for domain aggregates |
| `projections/` | CQRS read-model projections built from the event store |

#### Platform / Infrastructure

| Directory | Description |
|---|---|
| `app.module.ts` | Root module — wires all application modules together |
| `common/` | Shared DTOs, interceptors, filters, guards, audit, pagination, throttler, circuit-breaker |
| `config/` | Database config, env validation schema, logger config |
| `auth/` | JWT, MFA, API keys, session management, OIDC/OAuth2 |
| `OAuth2/` | OIDC provider module (wraps the auth OIDC flow) |
| `rbac/` | Role-Based Access Control policy engine |
| `roles/` | Medical RBAC decorators and guards (MedicalRole enum, MedicalRbacGuard) |
| `access-control/` | Fine-grained access grants, revocations, and consent enforcement |
| `tenant/` | Multi-tenancy — tenant resolution, Row-Level Security (RLS), tenant context |
| `tenant-config/` | Per-tenant feature flags and configuration overrides |
| `notifications/` | WebSocket gateway, transactional outbox, email, notification preferences |
| `queues/` | BullMQ job queues, DLQ, queue dashboard |
| `graphql/` | Apollo GraphQL server, subscriptions, dataloaders, cursor pagination |
| `pubsub/` | GraphQL PubSub transport (Redis-backed) |
| `subscriptions/` | GraphQL subscription lifecycle management |
| `metrics/` | Prometheus metrics, Grafana dashboards, SLO tracking |
| `health/` | `/health` endpoint (Terminus health indicators) |
| `analytics/` | Admin statistics and platform-level activity tracking |
| `jobs/` | Scheduled background jobs (cron-based) |
| `data-retention/` | Automated data-retention policy enforcement |
| `webhooks/` | Outbound webhook delivery, signature verification |
| `idempotency/` | Idempotency-key middleware for safe request replays |
| `dlq/` | Dead-letter queue inspection and replay |
| `incident/` | Incident tracking and escalation |
| `operator-runbook/` | Runbook endpoints for on-call operators |
| `versioning/` | API versioning helpers and deprecation interceptor |
| `admin/` | Admin-only endpoints, user management, system configuration |
| `security/` | Security headers config, IP allowlist, brute-force detection |
| `key-management/` | AWS KMS / local envelope encryption, DEK rotation |
| `encryption/` | PHI field-level encryption (AES-GCM, deterministic column transforms) |
| `i18n/` | Internationalisation (nestjs-i18n), translation files, i18n exception filter |
| `feature-flags/` | Runtime feature-flag evaluation |
| `circuit-breaker/` _(under `common/`)_ | Cockatiel-based circuit breaker for external calls |

#### Compliance & GDPR

| Directory | Description |
|---|---|
| `gdpr/` | GDPR data-subject requests (access, erasure, portability) |
| `data-residency/` | Geo-based data residency enforcement |
| `fhir/` | FHIR R4 resource mapping and bulk export |
| `research-export/` | De-identified, k-anonymous data exports for research |
| `ehr-import/` | Structured import of external EHR data (HL7, CSV) |
| `governance-analytics/` | Compliance dashboards and governance metrics |
| `consistency-checker/` | Cross-system data consistency validation |

---

### Directories that need cleanup ⚠️

The following directories are **named after GitHub issues** rather than their domain, which makes the codebase harder to navigate. They contain valid implementations but should be renamed or merged in a future clean-up PR.

| Directory | Status | Recommended action |
|---|---|---|
| `src/Auto-Generate TypeScript Client SDK from OpenAPI Spec/` | Duplicate — CI workflows only | Move workflows to `.github/workflows/`; delete directory |
| `src/SwaggerOpenAPI Documentation with Full Schema Coverage/` | Contains `export-openapi.ts` | Move script to `scripts/`; delete directory |
| `src/Profile and Optimize Database Query Performance Under Load/` | Contains benchmark scripts | Merge into `scripts/`; delete directory |
| `src/profile-and-optimize-database-query-performance-under-load/` | Duplicate of above (different casing) | Delete after merging |
| `src/Dockerfile and Docker Compose for Production-Ready Containerization/` | Contains CI yml files | Move to `.github/workflows/`; delete directory |
| `src/Build Admin Analytics Dashboard Endpoints/` | Unclear overlap with `analytics/` | Audit and merge or delete |
| `src/Telemedicine and Remote/` | Contains spaces in name; likely overlaps with… | Merge into `telemedicine-and-remote/` and delete |
| `src/telemedicine-and-remote/` | …this directory | Keep this one; delete the spaced variant |
| `src/Tenant Provisioning and Onboarding Workflow/` | Overlaps with… | Merge into `tenant-provisioning-and-onboarding-workflow/` |
| `src/tenant-provisioning-and-onboarding-workflow/` | …this directory | Keep; delete the spaced variant |
| `src/Department and Ward Management/` | Unclear if wired into app.module | Audit and either wire or delete |
| `src/Surgical Management System/` | Unclear if wired into app.module | Audit and either wire or delete |
| `src/Email Notification Service for Critical Access Events/` | Likely superseded by `notifications/` | Audit and delete if redundant |
| `src/Hospital config/` | Likely superseded by `hospital-registry/` | Audit and merge or delete |
| `src/Migration-CLI/` | Likely superseded by TypeORM CLI scripts | Audit and delete if redundant |
| `src/modules/` | Contains a `patient` sub-module alongside `src/patients/` | Audit which is canonical; delete the other |

> Track clean-up work in a dedicated issue. Do not import from the spaced-name directories in new code.

## Local Development with Docker

The fastest way to get a fully working environment is Docker — no local Node, Postgres, or Redis installation required.

### Services started

| Service  | Container    | Exposed port(s)       | Purpose                        |
|----------|--------------|-----------------------|--------------------------------|
| api      | hs-api       | 3000                  | NestJS app with hot reload     |
| postgres | hs-postgres  | 5432                  | PostgreSQL 15                  |
| redis    | hs-redis     | 6379                  | Redis 7                        |
| mailhog  | hs-mailhog   | 1025 (SMTP), 8025 (UI)| Local email capture            |

### Quick start

```bash
# 1. Copy the Docker env file (values are pre-wired to compose service names)
cp .env.docker .env.docker.local   # optional: customise secrets

# 2. Start all services
docker compose -f docker-compose.local.yml up --build

# 3. (First run) run migrations inside the running api container
docker compose -f docker-compose.local.yml exec api npm run migration:run
```

The API is available at **http://localhost:3000**  
Swagger UI is at **http://localhost:3000/api**  
MailHog web UI is at **http://localhost:8025**

### Hot reload

The `src/` directory is bind-mounted into the container. NestJS runs with `nest start --watch`, so any file save triggers an automatic rebuild inside the container — no restart needed.

### Useful commands

```bash
# Tail logs for a single service
docker compose -f docker-compose.local.yml logs -f api

# Run a one-off command inside the api container
docker compose -f docker-compose.local.yml exec api npm run migration:run

# Stop and remove containers (keeps volumes)
docker compose -f docker-compose.local.yml down

# Stop and wipe all data volumes
docker compose -f docker-compose.local.yml down -v
```

### Environment file

`.env.docker` is committed to the repo and contains safe local-only defaults. All hostnames (`postgres`, `redis`, `mailhog`) match the compose service names so they resolve inside the Docker network automatically. Copy and edit it if you need to override any value:

```bash
cp .env.docker .env.docker.local
# then pass it explicitly:
docker compose -f docker-compose.local.yml --env-file .env.docker.local up
```

> **Note:** `.env.docker` uses placeholder secrets. Never use these values outside of local development.

---

## Installation & Setup

### Prerequisites

- Node.js (v18 or higher)
- PostgreSQL (v12 or higher)
- npm or yarn

### Installation Steps

1. Install dependencies:
```bash
npm install
```

2. Create a `.env` file from `.env.example`:
```bash
cp .env.example .env
```

3. Update the `.env` file with your database credentials and configuration.

4. Run database migrations (if using migrations):
```bash
npm run migration:run
```

5. Start the development server:
```bash
npm run start:dev
```

The application will be available at `http://localhost:3000`
Swagger documentation will be available at `http://localhost:3000/api`

## Scripts Reference

The project has 66 npm scripts. A full reference — including descriptions, prerequisites, and safety warnings for destructive scripts — is in **[docs/scripts.md](docs/scripts.md)**.

Quick-start summary:

```bash
npm run start:dev          # API with hot-reload
npm run migration:run      # apply pending DB migrations
npm run seed               # load development reference data
npm run test               # unit tests
npm run test:e2e           # end-to-end tests (requires Docker)
npm run load-test:ci       # k6 load test + CI gate (requires k6)
npm run build:sdk          # build the TypeScript client SDK
```

---

## Configuration

### Environment variables

Copy `.env.example` to `.env` and fill in the values:

```bash
cp .env.example .env
```

`.env.example` is the authoritative reference for every environment variable the application reads. Variables are grouped by area and annotated with:

- `[REQUIRED]` — must be set before the app starts in production
- `[OPTIONAL]` — has a safe default; override only when needed
- `[SECRET]` — never commit the real value; use a secrets manager in production

**Key sections in `.env.example`:**

| Section | Variables |
|---|---|
| Application | `NODE_ENV`, `PORT`, `API_URL`, `APP_BASE_URL` |
| Database (primary) | `DATABASE_URL` / `DB_HOST` … `DB_POOL_MAX` |
| Database (read replica) | `DB_REPLICA_HOST` … `DB_REPLICA_POOL_MAX` |
| Redis | `REDIS_URL` / `REDIS_HOST` … `REDIS_DB` |
| JWT & Auth | `JWT_SECRET`, `JWT_EXPIRATION`, `REFRESH_TOKEN_SECRET` |
| Encryption / KMS | `ENCRYPTION_MASTER_KEY`, `MASTER_KEY`, `KMS_ENABLED`, `AWS_*` |
| Stellar / Soroban | `STELLAR_SECRET_KEY`, `STELLAR_CONTRACT_ID`, `SOROBAN_RPC_URL` |
| IPFS | `IPFS_NODE_URL`, `IPFS_GATEWAY`, `IPFS_FALLBACK_GATEWAY` |
| Email / SMTP | `SMTP_HOST`, `SMTP_USER`, `SMTP_PASS`, `MAIL_FROM_ADDRESS` |
| Telemedicine | `TELEMEDICINE_SIGNALING_URL`, `TELEMEDICINE_TOKEN_SECRET` |
| Research export | `RESEARCH_EXPORT_BUCKET`, `RESEARCH_K_ANONYMITY` |
| EHR import | `EHR_IMPORT_S3_BUCKET`, `EHR_IMPORT_MAX_RETRIES` |
| GraphQL subscriptions | `SUBSCRIPTIONS_IDLE_TIMEOUT_MS`, `SUBSCRIPTIONS_MAX_PER_CONNECTION` |
| Ops / alerting | `OPS_SLACK_WEBHOOK_URL`, `QUEUE_DEPTH_THRESHOLD` |
| Migrations | `MIGRATION_EXECUTOR`, `CONFIRM_PRODUCTION_MIGRATION` |

## Security Headers

The API applies `helmet()` in `src/main.ts` using the shared configuration in `src/security/http-security.config.ts`.

- `Content-Security-Policy`: Restricts the browser to loading scripts, styles, images, and network connections only from approved sources, reducing XSS and asset injection risk.
- `X-Frame-Options: DENY`: Prevents the API from being embedded in frames or iframes, blocking clickjacking attacks.
- `X-Content-Type-Options: nosniff`: Stops browsers from MIME-sniffing responses into a different content type than declared.
- `Strict-Transport-Security`: Tells browsers to use HTTPS for future requests and resist protocol downgrade attacks.
- `Referrer-Policy: no-referrer`: Prevents browsers from leaking request origin or path information in the `Referer` header.
- `X-XSS-Protection: 0`: Explicitly disables the legacy browser XSS filter so CSP remains the single, predictable browser-side XSS control.

## Core Modules

### Medical Records Module

The medical records module provides comprehensive functionality for managing medical records, including:

- Medical record CRUD operations
- Version control and audit trails
- Clinical note templates
- Medical history and timeline tracking
- File attachments (images, documents)
- Consent management
- Search and reporting

## API Endpoints

### Medical Records

- `POST /medical-records` - Create a new medical record
- `GET /medical-records/search` - Search medical records
- `GET /medical-records/:id` - Get a medical record by ID
- `GET /medical-records/:id/versions` - Get version history
- `GET /medical-records/timeline/:patientId` - Get patient timeline
- `PUT /medical-records/:id` - Update a medical record
- `PUT /medical-records/:id/archive` - Archive a medical record
- `PUT /medical-records/:id/restore` - Restore an archived record
- `DELETE /medical-records/:id` - Delete a medical record (soft delete)

### Clinical Templates

- `POST /clinical-templates` - Create a clinical template
- `GET /clinical-templates` - Get all active templates
- `GET /clinical-templates/:id` - Get a template by ID
- `PUT /clinical-templates/:id` - Update a template
- `DELETE /clinical-templates/:id` - Delete a template

### Consent Management

- `POST /consents` - Create a new consent
- `GET /consents/record/:recordId` - Get consents for a record
- `GET /consents/patient/:patientId` - Get consents for a patient
- `GET /consents/check` - Check if consent exists
- `GET /consents/:id` - Get a consent by ID
- `PUT /consents/:id/revoke` - Revoke a consent

### File Attachments

- `POST /attachments/upload` - Upload a file attachment
- `GET /attachments/record/:recordId` - Get attachments for a record
- `GET /attachments/:id` - Get an attachment by ID
- `GET /attachments/:id/download` - Download an attachment
- `DELETE /attachments/:id` - Delete an attachment

### Reporting

- `GET /reports/patient/:patientId/summary` - Get patient summary
- `GET /reports/activity` - Get activity report
- `GET /reports/consent` - Get consent report
- `GET /reports/statistics` - Get statistics

## Postman Collection

A comprehensive Postman collection is available for testing and exploring the API:

- **Location**: `docs/postman/`
- **Collection**: `MedChain.postman_collection.json`
- **Environments**: Local, Testnet, and Staging
- **Documentation**: `docs/postman/README.md`

### Features

- Organized into folders matching API modules (Auth, Records, Access Control, etc.)
- Pre-configured authentication with automatic JWT token management
- Collection-level tests for response validation
- Environment-specific configurations
- Example requests and responses for all endpoints

### Quick Start

1. Import the collection and environment files into Postman
2. Select the appropriate environment (Local/Testnet/Staging)
3. Run the "Login" request in the Auth folder
4. All subsequent requests will automatically use the JWT token

## Database Schema

### Medical Records

The system uses the following main entities:

1. **MedicalRecord** - Main medical record entity with version control
2. **MedicalRecordVersion** - Version history for audit trails
3. **MedicalHistory** - Timeline and activity tracking
4. **ClinicalNoteTemplate** - Reusable clinical note templates
5. **MedicalAttachment** - File attachments (images, documents)
6. **MedicalRecordConsent** - Consent management and sharing

## Medical Records System

### Features

#### 1. Medical Record Entity with Version Control
- Complete version history tracking
- Change tracking with before/after states
- Change reason documentation
- Automatic version numbering

#### 2. Clinical Note Templates and Structured Data
- Reusable templates for common clinical notes
- Structured field definitions
- Template categorization
- System and custom templates

#### 3. Medical History and Timeline Tracking
- Complete audit trail of all record activities
- Event types: created, updated, viewed, shared, archived, deleted
- IP address and user agent tracking
- Chronological timeline view

#### 4. Medical Image and Document Attachment
- Support for multiple file types (images, PDFs, documents)
- File size validation (10MB max)
- Secure file storage
- Metadata tracking

#### 5. Medical Record Sharing and Consent Management
- Granular consent types (view, share, download, modify, delete)
- Consent expiration management
- Consent revocation with reason tracking
- Sharing with users and organizations

#### 6. Medical Record Search and Reporting
- Advanced search with filters
- Patient summary reports
- Activity reports
- Consent reports
- Statistical analysis

### Acceptance Criteria Met

✅ **Medical records maintain complete audit trails**
- All changes are tracked in MedicalRecordVersion
- All activities are logged in MedicalHistory
- IP addresses and user agents are recorded

✅ **Clinical documentation follows medical standards**
- Structured templates for consistent documentation
- Version control ensures data integrity
- Metadata support for additional context

✅ **Medical history is easily accessible and searchable**
- Timeline endpoint for chronological view
- Search functionality with multiple filters
- Activity reports for analysis

✅ **Patient consent is properly managed and documented**
- Comprehensive consent entity with status tracking
- Expiration management
- Revocation with audit trail
- Consent verification endpoints

## Authentication & Authorization

(To be implemented - placeholder for future authentication system)

## Stellar Integration

(To be implemented - placeholder for Stellar Soroban smart contract integration)

## Error Handling

The application uses a global exception filter (`HttpExceptionFilter`) that:
- Catches all exceptions
- Formats error responses consistently
- Logs errors appropriately
- Provides detailed error information in development
- Sanitizes error messages in production

## Clinical Workflow APIs (#68)

The backend now includes an integrated clinical workflow surface across diagnosis, treatment planning, pharmacy, and documentation modules.

### Implemented API capabilities

- Diagnosis and treatment planning integration:
  - Get treatment plans by diagnosis
  - Get patient diagnoses with linked treatment plans
  - Validate diagnosis IDs on treatment plan create/update
- Prescription and medication workflow improvements:
  - Search prescriptions by status/patient/prescriber/date
  - Update eligible prescriptions
  - Add and retrieve prescription note history
- Clinical documentation enhancements:
  - Dedicated `clinical-notes` endpoints
  - SOAP/progress/discharge/consultation note support
  - Note completeness checks and signing workflow
- Procedure/care tracking and decision support:
  - Procedure cancellation endpoint
  - Auto decision-support alerts on treatment/procedure lifecycle changes
  - Treatment plan progress endpoint for care coordination dashboards

### Key endpoint groups

- `GET /diagnosis/:id/treatment-plans`
- `GET /diagnosis/patient/:patientId/treatment-plans`
- `GET /treatment-plans` (search filters)
- `GET /treatment-plans/:id/progress`
- `GET /pharmacy/prescriptions` (search filters)
- `PATCH /pharmacy/prescriptions/:id`
- `POST /pharmacy/prescriptions/:id/notes`
- `GET /pharmacy/prescriptions/:id/notes`
- `POST /clinical-notes`
- `GET /clinical-notes`
- `POST /clinical-notes/:id/sign`
- `GET /clinical-notes/:id/completeness`

## Testing

See **[docs/scripts.md](docs/scripts.md)** for the full list of test scripts and their prerequisites.

### Quick reference

```bash
# Unit tests
npm run test

# Unit tests with coverage
npm run test:cov

# E2E tests (requires Docker — starts a PostgreSQL container)
npm run test:e2e

# Full coverage (unit + e2e)
npm run test:all:cov

# HIPAA/GDPR compliance tests
npm run test:compliance
```

## Deployment

### Production Build

```bash
npm run build
npm run start:prod
```

### Environment Considerations

- Set `NODE_ENV=production`
- Configure proper database credentials
- Set up secure file storage
- Configure CORS appropriately
- Enable HTTPS
- Set up proper logging and monitoring

## License

MIT
