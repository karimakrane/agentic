# e-fatourati — Canonical Domain Model

**Status** : Architecture Reference Document — Pre-Implementation
**Version** : 1.0
**Date** : 2026-05-24
**Audience** : Engineering leads, architects, compliance officers

> **Document conventions**
> - `[CONFIRMED]` — derives directly from validated product requirements
> - `[ASSUMPTION]` — architectural decision made in absence of explicit guidance; requires validation
> - `[VERIFY]` — regulatory or legal item requiring official verification before implementation

---

## 1. Executive Summary

e-fatourati is a Morocco-focused, multi-tenant SaaS platform for electronic invoicing and DGI clearance. This document defines the canonical domain model prior to any implementation.

**Five architectural pillars govern the design:**

1. **Multi-tenant isolation as a hard constraint** — tenant boundaries are enforced at every architectural layer, not delegated to application logic alone.
2. **Compliance-first design** — regulatory obligations (DGI, DGSSI, CNDP, ISO 27001) shape the model; technical convenience does not override compliance requirements.
3. **Bidirectional clearance** — a tenant operates simultaneously as supplier (issuing invoices) and client (receiving cleared invoices). Both roles are first-class.
4. **Fiduciaire delegation as a governed contract** — a FiduciaireRelationship is a formal, auditable, revocable relationship exclusively between two active platform tenants. No external entity may participate.
5. **Platform administration isolation** — the internal admin domain shares no authentication surface, no data access path, and no session model with the tenant domain.

The domain is organized into **14 bounded contexts** following Domain-Driven Design principles. Each context owns its entities, enforces its invariants, and communicates with adjacent contexts through defined interfaces.

---

## 2. Confirmed Business Requirements

Derived directly from validated product specification. No inference.

| ID | Requirement |
|---|---|
| BR-01 | The platform is multi-tenant SaaS. |
| BR-02 | One user may belong to multiple tenants simultaneously. |
| BR-03 | A user may operate under different business profiles per tenant: `fiduciaire`, `auto-entrepreneur`, `company`. |
| BR-04 | The first user created within a tenant automatically becomes the tenant admin. |
| BR-05 | Tenant admins may invite users, create users, assign roles, revoke access, and manage memberships. |
| BR-06 | A fiduciaire tenant may manage company tenants through a formal, governed relationship. |
| BR-07 | A fiduciaire **cannot** manage a company not registered on e-fatourati. External entities are forbidden as managed parties. |
| BR-08 | Both fiduciaire and company may initiate the relationship (bidirectional invitation). |
| BR-09 | The company must explicitly accept before the relationship becomes active. |
| BR-10 | The relationship must be revocable by either party at any time. |
| BR-11 | Fiscal year is mandatory domain context for invoicing, reporting, accounting, and compliance operations. |
| BR-12 | The platform has a separate internal administration console. |
| BR-13 | Platform administration is isolated from tenant administration. |
| BR-14 | Invoicing is bidirectional: a tenant is supplier in transactions it initiates and client in transactions directed at it. |
| BR-15 | DGI clearance is required before an invoice reaches the recipient. |
| BR-16 | External parties (non-platform entities) may only participate in invoice transactions, never in fiduciaire relationships. |

---

## 3. Assumptions

Architectural decisions made in the absence of explicit product decisions. Each requires validation before implementation begins.

| ID | Assumption | Architectural Impact |
|---|---|---|
| A-01 | A user's email address is globally unique across the platform, not per tenant. | Identity model, login routing |
| A-02 | A tenant has exactly one profile type (`fiduciaire`, `auto-entrepreneur`, `company`). The profile is immutable after tenant creation. | Invitation rules, relationship eligibility |
| A-03 | A company tenant may have active fiduciaire relationships with multiple fiduciaire tenants simultaneously. | Relationship cardinality |
| A-04 | A FiduciaireRelationship carries an explicit set of scoped permissions granted by the company to the fiduciaire. | Access control model |
| A-05 | FiscalYear is scoped per tenant. One tenant, one set of fiscal years. | Fiscal model boundary |
| A-06 | FiscalYear status transitions are irreversible: `open` → `closed` → `locked`. | Data integrity, write-lock enforcement |
| A-07 | The canonical domain is `efatourati.ma`. All other registered domains redirect to it. | Session model, cookie scope |
| A-08 | Session tokens carry tenant context (`tenantId`, active `fiscalYearId`). Switching tenant requires explicit session re-scoping. | Session design |
| A-09 | Invoices are associated with the supplier's fiscal year at creation time. The receiving tenant views received invoices independently of its own active fiscal year context. | Invoice ↔ fiscal year binding |
| A-10 | ExternalParty records are private to the owning tenant. They are not discoverable by other tenants. | Counterparty directory isolation |
| A-11 | Platform admin authentication uses a separate identity provider or credential store, not the tenant user identity system. | Admin isolation |
| A-12 | A FiduciaireRelationship may be suspended (temporary halt, resumable) or revoked (permanent, non-resumable). These are distinct states. | Relationship lifecycle |
| A-13 | Subscription and billing are managed at the tenant level, not at the user level. | Billing scope |
| A-14 | Notification delivery channels (email, in-app, SMS) are configurable per tenant and per user preference. | Notification model |
| A-15 | All write operations on financial entities are blocked when the target fiscal year is in `closed` or `locked` status. | FiscalYearGuard scope |

---

## 4. Core Bounded Contexts

### 4.1 Identity & Authentication

**Purpose** — Manages the global identity of platform users across all tenants. Owns authentication lifecycle independently of authorization.

**Responsibilities**
- User identity creation, verification, and deactivation
- Credential management (password, OAuth, SSO tokens)
- MFA factor enrollment and verification
- Session issuance, validation, and revocation
- Password reset and account recovery workflows

**Boundary**
- Has no knowledge of tenants, roles, or permissions
- Makes no authorization decisions
- Provides authenticated identity assertions to other contexts
- Does not store business data

**Interactions**
- → Access Control Context: provides identity assertion for authorization evaluation
- → Audit Domain: emits authentication events (login, logout, failed attempts, MFA events)
- → Notification Domain: triggers account verification, password reset notifications

---

### 4.2 Access Control

**Purpose** — Determines what an authenticated identity may do within a resolved tenant context.

**Responsibilities**
- Role definition and assignment (tenant-scoped)
- Permission catalog management
- Policy evaluation for ABAC extensions
- Fiduciaire delegation permission resolution via DelegatedAccessGrant
- First-user-admin bootstrap on tenant creation
- Cross-tenant access resolution (fiduciaire reading company data)

**Boundary**
- Operates only within a resolved tenant context
- Does not perform authentication
- Does not define business rules — receives eligibility from other contexts
- Delegated access is always bounded by the originating FiduciaireRelationship scope

**Interactions**
- ← Identity & Authentication: receives authenticated identity
- ← Fiduciaire Relationship Management: receives DelegatedAccessGrant records
- → All domains: provides authorization decisions on every operation

---

### 4.3 Tenant Management

**Purpose** — Manages the lifecycle of tenant workspaces as the fundamental isolation unit of the platform.

**Responsibilities**
- Tenant creation, activation, suspension, termination
- Tenant profile type assignment (immutable post-creation)
- Tenant configuration and settings management
- First-user-admin assignment on tenant creation
- Tenant lifecycle event emission

**Boundary**
- Owns the Tenant and TenantProfile entities
- Does not handle user identity
- Does not handle invoicing, fiscal, or compliance operations directly
- Does not make billing decisions

**Interactions**
- → Identity & Authentication: coordinates user creation on tenant provisioning
- → Platform Administration: exposes tenant state for operational management
- → Subscription/Billing: emits tenant lifecycle events that drive billing
- → Audit Domain: emits all tenant lifecycle events

---

### 4.4 Organization Management

**Purpose** — Manages the legal and business entity details associated with a tenant.

**Responsibilities**
- Legal entity information: NIF, ICE, trade name, legal name
- Organization profile and tax registration details
- Contact information and address management
- Tax configuration per organization

**Boundary**
- Describes the business entity behind a tenant — not the tenant itself
- Does not handle authentication, access control, or invoicing directly
- Feeds legal entity data to Invoicing and Compliance contexts

**Interactions**
- ← Tenant Management: linked 1:1 to Tenant
- → Invoicing Domain: provides legal entity data stamped on invoices
- → Compliance Domain: provides regulatory identity data for CNDP/DGI obligations

---

### 4.5 Fiduciaire Relationship Management

**Purpose** — Governs the formal, lifecycle-managed, auditable relationship between a fiduciaire tenant and a company tenant. Enforces the platform-only constraint.

**Responsibilities**
- Invitation issuance and tracking (bidirectional: fiduciaire → company or company → fiduciaire)
- Relationship activation upon mutual acceptance
- Relationship suspension, resumption, and revocation
- Scope definition: which company data and operations the fiduciaire may access
- DelegatedAccessGrant lifecycle linked to relationship status
- Fiduciaire action attribution in audit trail

**Boundary**
- Both parties must be active tenants on e-fatourati — no exceptions
- Does not handle invoice processing directly
- Does not duplicate Access Control logic — delegates to it via DelegatedAccessGrant
- Relationship revocation synchronously invalidates all active grants (not eventual)

**Interactions**
- ← Tenant Management: validates tenant existence and activity status before allowing relationship
- → Access Control: issues and revokes DelegatedAccessGrant records
- → Audit Domain: all relationship lifecycle events are audited
- → Notification Domain: triggers invitation, acceptance, rejection, suspension, revocation notifications

---

### 4.6 Fiscal Context

**Purpose** — Manages the mandatory temporal and fiscal framework that governs all financial and compliance operations.

**Responsibilities**
- Fiscal year lifecycle: creation, closure, locking
- Fiscal period sub-division (monthly, quarterly) for reporting granularity
- Tax profile management per organization
- Write-lock enforcement on closed and locked fiscal years
- Archival scope definition for compliance and audit purposes
- Fiscal year context propagation rules

**Boundary**
- Fiscal years are tenant-scoped
- Does not contain invoice data — provides context to invoicing context
- Status transitions are irreversible

**Interactions**
- → Invoicing Domain: mandatory context for all invoice creation
- → Clearance Domain: defines temporal scope of clearance operations
- → Audit Domain: defines retention and archival scope
- → Compliance Domain: defines reporting periods for DGI obligations

---

### 4.7 Customer & Counterparty Management

**Purpose** — Manages the registry of customers and suppliers a tenant transacts with — both platform-registered tenants and external parties.

**Responsibilities**
- Customer directory management (per tenant, private)
- Supplier directory management (per tenant, private)
- ExternalParty record management (for invoice transactions only)
- Counterparty type resolution: platform tenant vs external
- PII classification of counterparty data

**Boundary**
- ExternalParty records appear only in invoice transactions, never in FiduciaireRelationships
- Directories are strictly private — not discoverable across tenants
- Platform tenant counterparties link to validated tenant records

**Interactions**
- ← Tenant Management: resolves whether a counterparty is an active platform tenant
- → Invoicing Domain: provides counterparty data for invoice generation
- → Compliance Domain: provides counterparty PII classification

---

### 4.8 Invoicing Domain

**Purpose** — Manages the complete lifecycle of fiscal documents from creation to submission for clearance.

**Responsibilities**
- Invoice creation, drafting, and editing (draft phase only)
- Invoice series management and sequential numbering per fiscal year
- Invoice format validation against DGI specifications
- Credit note issuance linked to original invoices
- Payment recording (fact recording, not payment execution)
- Invoice handoff to Clearance Domain upon signature

**Boundary**
- Does not execute DGI clearance — delegates to Clearance Domain
- Does not execute payments — records payment facts only
- Invoice content is immutable after clearance submission
- All invoices must be scoped to an open FiscalYear

**Interactions**
- ← Fiscal Context: FiscalYear validation on every write
- → Clearance Domain: submits signed, validated invoices for DGI clearance
- ← Clearance Domain: receives clearance status updates and delivery status
- → Audit Domain: all invoice state transitions are audited
- → Notification Domain: notifies recipient on delivery, clearance outcome

---

### 4.9 Clearance Domain

**Purpose** — Manages the DGI clearance lifecycle — the mandatory regulatory step that validates and authenticates invoices before delivery to the recipient.

**Responsibilities**
- Clearance submission to DGI external system
- Clearance status tracking and event journaling (immutable log)
- Post-clearance delivery to recipient tenant or external party
- Client response collection (accept / dispute / reject)
- Retry and error handling on DGI rejection
- Correction workflow initiation on clearance failure

**Boundary**
- DGI protocol specifics require official verification — no implementation assumptions here
- Invoice content cannot be modified after clearance submission
- ClearanceEvent log is append-only — no updates, no deletes
- Client response is scoped to the identified clientId on the invoice

**Interactions**
- ← Invoicing Domain: receives signed invoices for clearance
- → DGI External System: submits invoices for clearance `[VERIFY: protocol, format, authentication]`
- → Notification Domain: emits clearance outcome and delivery status events
- → Audit Domain: all clearance events are audited

---

### 4.10 Audit Domain

**Purpose** — Provides an immutable, append-only audit trail for all significant platform events across all bounded contexts. Distinct platform-admin audit trail exists separately.

**Responsibilities**
- Centralized audit event collection from all domains
- Append-only immutability enforcement at storage level
- Retention policy enforcement per regulatory requirements
- Audit query and export for compliance inspections and legal holds
- Separate platform admin audit log (AdminAuditEvent) — disjoint from tenant audit log

**Boundary**
- Strictly append-only — no UPDATE, no DELETE during retention period
- Receives events from all contexts but drives no business logic
- Platform admin audit trail is completely isolated from tenant audit trail
- Audit data cannot be queried by tenant users (compliance officers may have read access)

**Interactions**
- ← All domains: receives audit event emissions
- → Compliance Domain: provides event stream for control evidence collection
- ← Fiscal Context: determines archival and retention scope

---

### 4.11 Notification Domain

**Purpose** — Manages asynchronous notifications to users and tenants across configurable delivery channels.

**Responsibilities**
- Notification routing by channel (email, in-app, SMS)
- Template management and variable resolution
- Delivery tracking and retry logic
- User notification preference management
- Tenant-level channel configuration

**Boundary**
- Does not make business decisions
- Consumes events from other domains — does not access business data directly
- Delivery failure handling is internal to this context

**Interactions**
- ← All domains: subscribes to notification-triggering events
- → External providers: email, SMS gateways (providers undecided)

---

### 4.12 Compliance Domain

**Purpose** — Tracks regulatory controls, compliance evidence, PII classification, and access reviews across the DGI, DGSSI, CNDP, ISO 27001, and GDPR compatibility frameworks.

**Responsibilities**
- Compliance control registry (per regulatory framework)
- Control evidence collection and linkage
- Retention policy catalog management
- PII classification registry
- Access review scheduling and tracking
- Compliance status reporting for platform administration

**Boundary**
- Does not enforce application-level access — that belongs to Access Control
- Documents control existence and status; does not replace operational controls
- Reads from Audit Domain for evidence; does not write to it

**Interactions**
- ← Audit Domain: reads audit event stream for evidence collection
- → Platform Administration: compliance status and control gap reporting
- ← Organization Management: legal entity data for PII classification

---

### 4.13 Subscription & Billing

**Purpose** — Manages tenant subscriptions to the platform, available plans, and feature availability gates.

**Responsibilities**
- Subscription lifecycle (trial, active, past_due, cancelled, expired)
- Plan definition and feature entitlements
- Feature flag evaluation per tenant
- Billing event emission (payment processing is external)
- Usage limit enforcement

**Boundary**
- Billing is at tenant level, never at user level
- Payment execution is external — this context records billing facts only
- Feature availability is the primary gate for capability access

**Interactions**
- ← Tenant Management: tenant provisioning triggers subscription initialization
- → Platform Administration: subscription status visibility
- → All domains: feature flag evaluation determines capability availability

---

### 4.14 Platform Administration

**Purpose** — Provides operational management of the platform itself. Completely isolated from the tenant domain at every layer.

**Responsibilities**
- Tenant lifecycle management from an operational perspective
- Subscription and plan administration
- Compliance oversight and reporting
- Support action documentation with mandatory justification
- System health monitoring
- Platform admin-specific audit trail (AdminAuditEvent)

**Boundary**
- Platform admins have no implicit access to tenant business data
- Access to tenant data requires an explicit, audited SupportAction with justification and authorization
- Authentication uses a completely separate credential surface
- The AdminAuditEvent log is disjoint from the tenant AuditEvent log
- This context has no overlap with TenantAdmin capabilities

**Interactions**
- ← All domains: read-only operational visibility into platform health and tenant state
- → Tenant Management: administrative tenant actions (suspension, termination)
- → Audit Domain (admin): all admin actions emit AdminAuditEvent records

---

## 5. Core Entities

### 5.1 Identity Entities

---

#### User
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | Globally unique |
| `email` | string | Globally unique, primary login identifier `[ASSUMPTION A-01]` |
| `status` | enum | `pending_verification` \| `active` \| `suspended` \| `deactivated` |
| `emailVerifiedAt` | timestamp | Null until email confirmed |
| `createdAt` | timestamp | |
| `lastActiveAt` | timestamp | Updated on session activity |

**Owning context** : Identity & Authentication
**Lifecycle** : Created on registration → email verified → active. Suspended or deactivated by admin action. Never deleted during retention period.
**Tenant scope** : Global — User is not tenant-scoped. TenantMembership links a User to a Tenant.
**Security sensitivity** : High — PII (email). Password and credential data are in Credential entity, never on User.

---

#### Credential
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `userId` | UUID | FK → User |
| `type` | enum | `password` \| `oauth_google` \| `oauth_microsoft` \| `sso` |
| `hashedValue` | string | Bcrypt/Argon2 hash — NEVER returned in API responses |
| `isActive` | boolean | |
| `lastUsedAt` | timestamp | |
| `expiresAt` | timestamp | For OAuth tokens |

**Owning context** : Identity & Authentication
**Security sensitivity** : Critical — never exposed outside the authentication context, never logged.

---

#### Session
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `userId` | UUID | FK → User |
| `activeTenantId` | UUID | FK → Tenant — resolved on login or tenant switch |
| `activeFiscalYearId` | UUID | FK → FiscalYear — resolved on tenant context |
| `issuedAt` | timestamp | |
| `expiresAt` | timestamp | |
| `ipAddress` | string | Pseudonymized in logs `[VERIFY: CNDP requirements]` |
| `userAgent` | string | |
| `revokedAt` | timestamp | Null until logout or forced revocation |

**Owning context** : Identity & Authentication
**Lifecycle** : Issued on authentication → active → expired or revoked on logout/admin action.
**Security sensitivity** : Critical — session ID is the authentication surface.

---

#### MFAFactor
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `userId` | UUID | FK → User |
| `type` | enum | `totp` \| `sms` \| `email` |
| `status` | enum | `pending_verification` \| `active` \| `revoked` |
| `enrolledAt` | timestamp | |
| `lastVerifiedAt` | timestamp | |
| `maskedIdentifier` | string | Last 4 digits for SMS, masked email — display only |

**Owning context** : Identity & Authentication
**Security sensitivity** : High — factor secrets stored separately, never in this entity.

---

### 5.2 Access Control Entities

---

#### Role
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant. Null for system roles. |
| `name` | string | e.g., `TenantAdmin`, `Accountant`, `Invoicer`, `Viewer` |
| `permissions` | string[] | Array of permission codes |
| `isSystem` | boolean | System roles cannot be modified by tenant admins |
| `createdAt` | timestamp | |

**Owning context** : Access Control
**Tenant scope** : Tenant-scoped (or global for system roles).

---

#### Permission
| Attribute | Type | Notes |
|---|---|---|
| `code` | string | e.g., `invoice:create`, `fiscalyear:close`, `fiduciaire:delegate` |
| `resource` | string | Resource type |
| `action` | string | `read` \| `write` \| `delete` \| `submit` \| `approve` \| `export` |
| `description` | string | Human-readable |

**Owning context** : Access Control
**Tenant scope** : Global catalog — assignment is tenant-scoped via Role.

---

#### Policy
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `name` | string | |
| `conditions` | JSON | ABAC conditions (e.g., `fiscalYear.status = open`) |
| `effect` | enum | `allow` \| `deny` |
| `description` | string | |

**Owning context** : Access Control
**Notes** : Used for ABAC extensions beyond standard RBAC. See section 8.

---

#### TenantMembership
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `userId` | UUID | FK → User |
| `tenantId` | UUID | FK → Tenant |
| `roleId` | UUID | FK → Role (tenant-scoped) |
| `isAdmin` | boolean | True for the first user; can be granted by existing admins |
| `status` | enum | `active` \| `suspended` \| `revoked` |
| `joinedAt` | timestamp | |
| `revokedAt` | timestamp | Null until revoked |
| `invitedBy` | UUID | FK → User who initiated the membership |

**Owning context** : Access Control
**Lifecycle** : Created via invitation acceptance or direct creation by tenant admin. Revoked by tenant admin or platform admin via SupportAction.
**Tenant scope** : Scoped to one Tenant. A User has one TenantMembership per Tenant they belong to.
**Security sensitivity** : High — isAdmin flag controls full tenant access.

---

### 5.3 Tenant Entities

---

#### Tenant
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `name` | string | Display name |
| `status` | enum | `provisioning` \| `active` \| `suspended` \| `terminated` |
| `createdAt` | timestamp | |
| `suspendedAt` | timestamp | |
| `terminatedAt` | timestamp | |

**Owning context** : Tenant Management
**Lifecycle** : Provisioned on creation → activated → optionally suspended → terminated. Terminated tenants retain data per retention policy.
**Security sensitivity** : Medium.

---

#### TenantProfile
| Attribute | Type | Notes |
|---|---|---|
| `tenantId` | UUID | FK → Tenant (1:1) |
| `type` | enum | `fiduciaire` \| `auto-entrepreneur` \| `company` |
| `configuredAt` | timestamp | Set at tenant creation |

**Owning context** : Tenant Management
**Notes** : Immutable after creation `[ASSUMPTION A-02]`. Profile type governs what relationships and operations are available to the tenant.

---

#### Organization
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant (1:1) |
| `legalName` | string | Official registered name |
| `tradeName` | string | Commercial name |
| `nif` | string | Numéro d'Identification Fiscale — `[VERIFY: DGI format]` |
| `ice` | string | Identifiant Commun de l'Entreprise — applicable for company and fiduciaire |
| `legalForm` | string | SA, SARL, auto-entrepreneur, etc. |
| `address` | JSON | Structured address |
| `phone` | string | |
| `vatRegistered` | boolean | |

**Owning context** : Organization Management
**Security sensitivity** : High — PII and fiscal identity data. Subject to CNDP and DGI obligations.

---

### 5.4 Fiduciaire Relationship Entities

---

#### Invitation
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `type` | enum | `user_membership` \| `fiduciaire_relationship` |
| `initiatorTenantId` | UUID | FK → Tenant — always a platform tenant |
| `targetTenantId` | UUID | FK → Tenant — for fiduciaire_relationship type |
| `targetEmail` | string | For user_membership type |
| `status` | enum | `pending` \| `accepted` \| `rejected` \| `expired` \| `cancelled` |
| `token` | string | Signed, time-limited token — never stored in plain text |
| `expiresAt` | timestamp | |
| `createdAt` | timestamp | |
| `respondedAt` | timestamp | |

**Owning context** : Fiduciaire Relationship Management (for relationship invitations) / Access Control (for user membership invitations)
**Security sensitivity** : Medium — invitation token must be unpredictable and time-limited.

---

#### FiduciaireRelationship
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `fiduciaireTenantId` | UUID | FK → Tenant — must have profile type `fiduciaire` |
| `companyTenantId` | UUID | FK → Tenant — must have profile type `company` |
| `status` | enum | `invited` \| `pending_acceptance` \| `active` \| `suspended` \| `revoked` \| `expired` |
| `initiatedBy` | enum | `fiduciaire` \| `company` |
| `invitationId` | UUID | FK → Invitation |
| `activatedAt` | timestamp | Null until both parties have completed acceptance |
| `suspendedAt` | timestamp | |
| `suspendedBy` | UUID | FK → TenantMembership |
| `revokedAt` | timestamp | |
| `revokedBy` | UUID | FK → TenantMembership |

**Owning context** : Fiduciaire Relationship Management
**Invariants** :
- `fiduciaireTenantId` and `companyTenantId` must both be active platform tenants — no external parties.
- `fiduciaireTenantId.profile.type` must be `fiduciaire`.
- `companyTenantId.profile.type` must be `company`.
- Revocation is terminal — status cannot transition from `revoked` to any other state.
**Security sensitivity** : Critical — controls cross-tenant data access.

---

#### RelationshipScope
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `relationshipId` | UUID | FK → FiduciaireRelationship |
| `scopeCode` | string | e.g., `invoices:read`, `invoices:write`, `accounting:read`, `fiscal:submit` |
| `grantedAt` | timestamp | |
| `grantedBy` | UUID | FK → TenantMembership (must be company admin) |
| `revokedAt` | timestamp | |

**Owning context** : Fiduciaire Relationship Management
**Notes** : Scopes define the ceiling of what a fiduciaire may do in the company tenant. DelegatedAccessGrant is derived from active scopes.

---

#### DelegatedAccessGrant
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `relationshipId` | UUID | FK → FiduciaireRelationship |
| `beneficiaryUserId` | UUID | FK → User (fiduciaire side) |
| `targetTenantId` | UUID | FK → Tenant (the company) |
| `scopeCode` | string | FK → RelationshipScope.scopeCode |
| `isActive` | boolean | Set to false on relationship suspension or revocation |
| `validFrom` | timestamp | |
| `validUntil` | timestamp | Null for open-ended, set on expiry-based grants |

**Owning context** : Access Control (issued by Fiduciaire Relationship Management)
**Lifecycle** : Created when a fiduciaire user is authorized under an active relationship. Immediately deactivated on relationship suspension or revocation — synchronous, not eventual.
**Security sensitivity** : Critical — enables cross-tenant access.

---

### 5.5 Fiscal Entities

---

#### FiscalYear
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant |
| `label` | string | e.g., "2025" |
| `startDate` | date | |
| `endDate` | date | |
| `status` | enum | `open` \| `closed` \| `locked` |
| `createdAt` | timestamp | |
| `closedAt` | timestamp | |
| `lockedAt` | timestamp | |
| `closedBy` | UUID | FK → TenantMembership |

**Owning context** : Fiscal Context
**Lifecycle** : Created by tenant admin → `open` → `closed` (accounting finalized) → `locked` (archive, immutable). Transitions are irreversible `[ASSUMPTION A-06]`.
**Tenant scope** : Strictly tenant-scoped.
**Notes** : `[VERIFY: DGI requirements on fiscal year structure and Morocco-specific period rules]`

---

#### FiscalPeriod
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `fiscalYearId` | UUID | FK → FiscalYear |
| `tenantId` | UUID | FK → Tenant |
| `type` | enum | `monthly` \| `quarterly` |
| `startDate` | date | |
| `endDate` | date | |
| `status` | enum | `open` \| `closed` |

**Owning context** : Fiscal Context
**Notes** : Used for sub-period reporting and VAT declaration granularity. `[VERIFY: DGI VAT declaration frequency requirements]`

---

#### TaxProfile
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant |
| `vatRegime` | string | e.g., `normal`, `simplified`, `exempt` — `[VERIFY: DGI regime codes]` |
| `vatRate` | decimal | Default VAT rate for this tenant `[VERIFY: DGI applicable rates]` |
| `effectiveFrom` | date | |
| `effectiveTo` | date | Null if currently active |

**Owning context** : Fiscal Context
**Security sensitivity** : High — fiscal regulatory data.

---

### 5.6 Business Entities

---

#### Customer
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `ownerTenantId` | UUID | FK → Tenant — the tenant managing this customer record |
| `type` | enum | `platform_tenant` \| `external` |
| `platformTenantId` | UUID | FK → Tenant — populated if type = `platform_tenant` |
| `name` | string | |
| `fiscalId` | string | NIF or equivalent `[VERIFY: format by country for international invoicing]` |
| `ice` | string | ICE if applicable |
| `email` | string | Contact email — PII |
| `address` | JSON | Structured address — PII |
| `createdAt` | timestamp | |

**Owning context** : Customer & Counterparty Management
**Tenant scope** : Private to `ownerTenantId`.

---

#### Supplier

Symmetric structure to Customer. Represents entities from which the owning tenant receives invoices.

---

#### InvoiceSeries
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant |
| `fiscalYearId` | UUID | FK → FiscalYear |
| `prefix` | string | e.g., `FAC-2025-` |
| `lastSequenceNumber` | integer | Atomic increment — no gaps allowed |
| `format` | string | Pattern for invoice number generation |

**Owning context** : Invoicing Domain
**Notes** : Sequential invoice numbering is a DGI requirement. `[VERIFY: exact DGI sequential numbering rules]`. Gaps in sequence require formal justification.

---

#### Invoice
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `supplierId` | UUID | FK → Tenant — the issuing tenant |
| `clientId` | UUID | FK → Tenant — null if external client |
| `externalClientRef` | UUID | FK → Customer (external) — null if platform client |
| `fiscalYearId` | UUID | FK → FiscalYear (supplier's fiscal year) |
| `seriesId` | UUID | FK → InvoiceSeries |
| `invoiceNumber` | string | Generated from series — immutable once assigned |
| `clearanceStatus` | enum | `draft` \| `pending_signature` \| `pending_clearance` \| `cleared` \| `clearance_rejected` |
| `deliveryStatus` | enum | `pending_delivery` \| `delivered` \| `acknowledged` \| `accepted` \| `disputed` \| `rejected` |
| `totalHT` | decimal | Total excluding tax |
| `totalVAT` | decimal | Total VAT |
| `totalTTC` | decimal | Total including tax |
| `currency` | string | ISO 4217 — default MAD `[VERIFY: DGI multi-currency rules]` |
| `issuedAt` | timestamp | |
| `dgiClearanceRef` | string | Assigned by DGI upon clearance `[VERIFY: format]` |
| `dgiClearedAt` | timestamp | |

**Owning context** : Invoicing Domain
**Invariants** :
- Exactly one of `clientId` or `externalClientRef` must be populated.
- Content is immutable once `clearanceStatus` reaches `pending_clearance`.
- `fiscalYearId` must reference an `open` FiscalYear at creation time.
**Security sensitivity** : Critical — fiscal document with regulatory retention obligations.

---

#### InvoiceLine
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `invoiceId` | UUID | FK → Invoice |
| `tenantId` | UUID | FK → Tenant (supplier) — for row-level isolation |
| `description` | string | |
| `quantity` | decimal | |
| `unitPrice` | decimal | |
| `vatRate` | decimal | |
| `totalHT` | decimal | |
| `totalTTC` | decimal | |
| `sortOrder` | integer | Display ordering |

**Owning context** : Invoicing Domain

---

#### CreditNote
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `originalInvoiceId` | UUID | FK → Invoice |
| `supplierId` | UUID | FK → Tenant |
| `fiscalYearId` | UUID | FK → FiscalYear |
| `reason` | string | |
| `clearanceStatus` | enum | Same status model as Invoice |
| `totalHT` | decimal | |
| `totalVAT` | decimal | |
| `totalTTC` | decimal | |
| `issuedAt` | timestamp | |
| `dgiClearanceRef` | string | `[VERIFY: DGI credit note clearance requirements]` |

**Owning context** : Invoicing Domain
**Notes** : CreditNote follows the same clearance flow as Invoice. `[VERIFY: DGI rules for credit note issuance and clearance]`

---

#### PaymentRecord
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `invoiceId` | UUID | FK → Invoice |
| `tenantId` | UUID | FK → Tenant |
| `amount` | decimal | |
| `paymentDate` | date | |
| `method` | string | `bank_transfer` \| `cheque` \| `cash` \| `other` |
| `reference` | string | Bank reference or cheque number |
| `recordedBy` | UUID | FK → TenantMembership |
| `recordedAt` | timestamp | |

**Owning context** : Invoicing Domain
**Notes** : Records the fact of payment. Payment execution is external to the platform.

---

### 5.7 Clearance & Audit Entities

---

#### ClearanceEvent
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `invoiceId` | UUID | FK → Invoice |
| `sequenceNumber` | integer | Monotonically increasing per invoice — detects gaps |
| `eventType` | enum | `submitted` \| `cleared` \| `clearance_rejected` \| `delivered` \| `response_received` \| `resubmitted` |
| `occurredAt` | timestamp | |
| `dgiResponseCode` | string | `[VERIFY: DGI response codes]` |
| `dgiResponsePayload` | JSON | Raw DGI response — sanitized of sensitive data |
| `triggeredBy` | string | System component or user reference |

**Owning context** : Clearance Domain
**Invariants** : Immutable after insertion. No UPDATE or DELETE permitted. Sequence gaps are an integrity alert.
**Security sensitivity** : Critical — regulatory evidence.

---

#### InvoiceResponse
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `invoiceId` | UUID | FK → Invoice |
| `clientTenantId` | UUID | FK → Tenant — must match `Invoice.clientId` |
| `responseType` | enum | `accepted` \| `disputed` \| `rejected` |
| `reason` | string | Mandatory for `disputed` and `rejected` |
| `respondedAt` | timestamp | |
| `respondedBy` | UUID | FK → TenantMembership |

**Owning context** : Clearance Domain
**Invariants** : Can only be submitted by the tenant identified in `Invoice.clientId`.

---

#### AuditEvent
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | Null for platform-level events |
| `actorId` | UUID | FK → User or system component |
| `actorType` | enum | `user` \| `system` \| `platform_admin` |
| `eventType` | string | Namespaced event code e.g. `invoice.status_changed` |
| `resourceType` | string | |
| `resourceId` | UUID | |
| `payload` | JSON | Sanitized — no credentials, no raw PII |
| `occurredAt` | timestamp | |
| `ipAddress` | string | Pseudonymized `[VERIFY: CNDP retention requirements]` |
| `sessionId` | UUID | FK → Session |

**Owning context** : Audit Domain
**Invariants** : INSERT only. No UPDATE, no DELETE during regulatory retention period. Written by all domains.
**Security sensitivity** : Critical — compliance evidence and legal record.

---

#### AdminAuditEvent
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `adminUserId` | UUID | FK → PlatformAdminUser |
| `actionType` | string | |
| `targetResourceType` | string | |
| `targetResourceId` | UUID | |
| `targetTenantId` | UUID | FK → Tenant — when action concerns a specific tenant |
| `justification` | string | Mandatory — explains why admin accessed this resource |
| `authorizedBy` | UUID | FK → PlatformAdminUser — for high-risk actions requiring dual authorization |
| `occurredAt` | timestamp | |
| `ipAddress` | string | |

**Owning context** : Platform Administration — stored in separate audit schema.
**Security sensitivity** : Critical — separate storage, no access from tenant domain.

---

### 5.8 Compliance Entities

---

#### ComplianceControl
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `framework` | enum | `dgi` \| `dgssi` \| `cndp` \| `iso27001` \| `gdpr` |
| `controlCode` | string | e.g., `ISO27001-A.8.24`, `CNDP-ART-15` |
| `description` | string | |
| `status` | enum | `compliant` \| `partial` \| `non_compliant` \| `not_applicable` |
| `evidenceRefs` | string[] | References to evidence artifacts |
| `lastReviewedAt` | timestamp | |
| `reviewedBy` | string | |

**Owning context** : Compliance Domain

---

#### RetentionPolicy
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `dataType` | string | e.g., `fiscal_invoice`, `audit_log`, `personal_data_contact` |
| `minimumRetentionDays` | integer | `[VERIFY: DGI, CNDP minimum retention periods]` |
| `maximumRetentionDays` | integer | `[VERIFY: CNDP maximum retention limits]` |
| `legalBasis` | string | Reference to legal text |
| `regulatorySource` | enum | `dgi` \| `cndp` \| `iso27001` |

**Owning context** : Compliance Domain

---

#### AccessReview
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant |
| `scheduledAt` | timestamp | |
| `completedAt` | timestamp | |
| `reviewerId` | UUID | FK → TenantMembership |
| `scope` | string | e.g., `fiduciaire_grants`, `tenant_memberships` |
| `findings` | JSON | |
| `status` | enum | `scheduled` \| `in_progress` \| `completed` \| `overdue` |

**Owning context** : Compliance Domain

---

### 5.9 Platform Entities

---

#### PlatformAdminUser
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `email` | string | Separate from tenant User email space |
| `role` | enum | `super_admin` \| `support` \| `ops` \| `compliance_officer` |
| `status` | enum | `active` \| `suspended` \| `deactivated` |
| `mfaEnrolled` | boolean | MFA mandatory — `true` always enforced |
| `createdAt` | timestamp | |
| `lastLoginAt` | timestamp | |

**Owning context** : Platform Administration
**Notes** : Completely separate entity from User. Separate authentication surface, separate session model.

---

#### SupportAction
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `adminUserId` | UUID | FK → PlatformAdminUser |
| `targetTenantId` | UUID | FK → Tenant |
| `actionType` | string | e.g., `view_tenant_data`, `reset_user_mfa`, `unlock_fiscal_year` |
| `justification` | string | Mandatory — human-readable reason |
| `authorizedBy` | UUID | FK → PlatformAdminUser — senior authorization for critical actions |
| `executedAt` | timestamp | |
| `expiresAt` | timestamp | Time-bound access window for sensitive operations |
| `status` | enum | `active` \| `expired` \| `revoked` |

**Owning context** : Platform Administration
**Notes** : All access to tenant data from platform admin context requires an active SupportAction. No implicit access.

---

#### Subscription
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `tenantId` | UUID | FK → Tenant |
| `planId` | UUID | FK → Plan |
| `status` | enum | `trialing` \| `active` \| `past_due` \| `cancelled` \| `expired` |
| `startedAt` | timestamp | |
| `trialEndsAt` | timestamp | |
| `currentPeriodStart` | timestamp | |
| `currentPeriodEnd` | timestamp | |
| `cancelledAt` | timestamp | |

**Owning context** : Subscription & Billing

---

#### Plan
| Attribute | Type | Notes |
|---|---|---|
| `id` | UUID | |
| `name` | string | e.g., `Starter`, `Professional`, `Enterprise` |
| `features` | string[] | Feature flag codes included |
| `limits` | JSON | e.g., `{ "invoices_per_month": 500 }` |
| `priceMAD` | decimal | Monthly price in MAD |
| `billingCycle` | enum | `monthly` \| `annual` |
| `isActive` | boolean | |

**Owning context** : Subscription & Billing

---

#### FeatureFlag
| Attribute | Type | Notes |
|---|---|---|
| `code` | string | Feature identifier |
| `tenantId` | UUID | Null for global flags |
| `isEnabled` | boolean | |
| `rolloutPercentage` | integer | 0–100 for gradual rollout |
| `overriddenAt` | timestamp | When explicitly set for a tenant |

**Owning context** : Subscription & Billing

---

## 6. Relationship Model

### Primary Relationships

| Relationship | Cardinality | Description |
|---|---|---|
| User ↔ Tenant | N:M via TenantMembership | A user belongs to multiple tenants; a tenant has multiple users |
| Tenant ↔ TenantProfile | 1:1 | Immutable profile type |
| Tenant ↔ Organization | 1:1 | Legal entity details |
| Tenant ↔ FiscalYear | 1:N | A tenant owns its fiscal years |
| FiscalYear ↔ Invoice | 1:N | Invoices are scoped to one fiscal year |
| FiscalYear ↔ InvoiceSeries | 1:N | Series reset per fiscal year |
| Invoice ↔ InvoiceLine | 1:N | An invoice has multiple lines |
| Invoice ↔ ClearanceEvent | 1:N | Immutable event log per invoice |
| Invoice ↔ InvoiceResponse | 1:0..1 | One formal response per invoice from client |
| Invoice ↔ CreditNote | 1:N | Multiple credit notes against an invoice |
| FiduciaireRelationship ↔ RelationshipScope | 1:N | Multiple scopes per relationship |
| FiduciaireRelationship ↔ DelegatedAccessGrant | 1:N | One grant per user per scope |
| User ↔ DelegatedAccessGrant | 1:N | A fiduciaire user holds multiple grants |
| PlatformAdminUser ↔ SupportAction | 1:N | Admin actions are documented |
| Tenant ↔ Subscription | 1:1 | One active subscription per tenant |

### Critical Isolation Boundaries

| Boundary | Rule |
|---|---|
| FiduciaireRelationship | Both tenants must be platform-registered. No ExternalParty. |
| ExternalParty | Scoped to ownerTenantId. Invoice transactions only. No relationship access. |
| PlatformAdminUser | No access to tenant data without SupportAction. Separate identity space. |
| AuditEvent | Tenant-scoped events have tenantId. Admin events go to AdminAuditEvent. No cross-contamination. |
| DelegatedAccessGrant | Immediately deactivated on relationship suspension or revocation. |

---

## 7. ASCII Domain Diagram

```
═══════════════════════════════════════════════════════════════════════════════
  PLATFORM ADMINISTRATION (ISOLATED DOMAIN)
═══════════════════════════════════════════════════════════════════════════════

  ┌─────────────────────┐         ┌──────────────┐
  │  PlatformAdminUser  │────────▶│ SupportAction│
  └─────────────────────┘         └──────┬───────┘
           │                             │ targets
           │ emits                       ▼
           ▼                     ┌───────────────┐
  ┌──────────────────┐           │ AdminAuditEvent│
  │  AdminAuditEvent │           │  (isolated log)│
  └──────────────────┘           └───────────────┘

═══════════════════════════════════════════════════════════════════════════════
  TENANT DOMAIN
═══════════════════════════════════════════════════════════════════════════════

           ┌───────────────┐
           │     User      │◀─────── global identity (email unique)
           └───────┬───────┘
                   │ N:M via
                   ▼
  ┌────────────────────────┐
  │    TenantMembership    │
  │  (userId, tenantId,    │
  │   roleId, isAdmin)     │
  └───────────┬────────────┘
              │ belongs to
              ▼
  ┌───────────────────┐      1:1     ┌───────────────┐
  │      Tenant       │─────────────▶│ TenantProfile │
  │  (active/susp..)  │              │ (immutable)   │
  └─────────┬─────────┘              └───────────────┘
            │
    ┌───────┼────────────────────────────┐
    │       │                            │
    ▼       ▼                            ▼
┌────────┐ ┌──────────────┐   ┌─────────────────────┐
│  Org.  │ │  FiscalYear  │   │   Subscription      │
│(NIF,   │ │  open/closed │   │   Plan, Features    │
│ ICE..) │ │  /locked     │   └─────────────────────┘
└────────┘ └──────┬───────┘
                  │
        ┌─────────┴──────────┐
        │                    │
        ▼                    ▼
┌───────────────┐   ┌──────────────────┐
│ FiscalPeriod  │   │  InvoiceSeries   │
│(monthly/qtrly)│   │ (sequential num) │
└───────────────┘   └────────┬─────────┘
                             │
                             ▼
                    ┌────────────────────────────────────┐
                    │            Invoice                  │
                    │  supplierId (Tenant)                │
                    │  clientId (Tenant | null)           │
                    │  externalClientRef (Customer | null)│
                    │  clearanceStatus / deliveryStatus   │
                    └───────────┬────────────────────────┘
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
   ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
   │ InvoiceLine  │  │  ClearanceEvent  │  │InvoiceResponse│
   │              │  │  (immutable log) │  │(client reply) │
   └──────────────┘  └──────────────────┘  └──────────────┘

═══════════════════════════════════════════════════════════════════════════════
  FIDUCIAIRE RELATIONSHIP (PLATFORM-ONLY)
═══════════════════════════════════════════════════════════════════════════════

  ┌──────────────────┐              ┌───────────────────────┐
  │  Tenant          │              │  FiduciaireRelationship│
  │  (fiduciaire)    │◀─────────────│  invited/active/revoked│
  └──────────────────┘  manages     └───────────┬───────────┘
                                                │ manages
                                    ┌───────────▼──────────┐
                                    │  Tenant              │
                                    │  (company)           │
                                    └──────────────────────┘
                                                │
                                    ┌───────────▼──────────────┐
                                    │    RelationshipScope     │
                                    │  (scopeCode, grantedBy)  │
                                    └───────────┬──────────────┘
                                                │ generates
                                    ┌───────────▼──────────────┐
                                    │  DelegatedAccessGrant    │
                                    │  (userId, scope, active) │
                                    └──────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
  AUDIT & COMPLIANCE (APPEND-ONLY)
═══════════════════════════════════════════════════════════════════════════════

  All domains ──emit──▶ AuditEvent (tenantId, actorId, eventType, payload)
                              │
                              ▼
                    ComplianceControl ◀── evidence refs
                    RetentionPolicy   ◀── data type rules
                    AccessReview      ◀── periodic reviews
```

---

## 8. Access Control Model

### Evaluation

| Model | Pros | Cons | Fit for e-fatourati |
|---|---|---|---|
| **Pure RBAC** | Simple, auditable, well-understood | Cannot express fiduciaire delegation, context-based rules | Insufficient |
| **Pure ABAC** | Maximum flexibility | Complex to reason about, hard to audit, expensive to evaluate | Overkill for standard operations |
| **Hybrid RBAC + ABAC** | Simple for standard ops, flexible for delegation and context | Slightly more complex to implement | **Recommended** |

### Recommended Model: Hybrid RBAC + ABAC

**RBAC layer** governs standard operations within a tenant:

| System Role | Scope | Capabilities |
|---|---|---|
| `TenantAdmin` | Tenant | Full control: members, settings, fiscal years, all operations |
| `Accountant` | Tenant | Invoicing (read/write), accounting, reporting. No user management. |
| `Invoicer` | Tenant | Invoice creation and management. Read-only on accounting. |
| `Viewer` | Tenant | Read-only on all tenant data |
| `Auditor` | Tenant | Read-only on audit events and compliance data |

Tenant admins may create custom roles by composing permissions from the catalog.

**ABAC extensions** govern contextual and delegated access:

| Policy | Condition | Effect |
|---|---|---|
| Fiscal write gate | `fiscalYear.status == open` | Deny all write operations on closed/locked fiscal years |
| Fiduciaire delegation gate | `DelegatedAccessGrant.isActive == true AND scopeCode matches operation` | Allow cross-tenant operation within granted scope |
| Tenant profile gate | `tenant.profile.type == fiduciaire` | Allow fiduciaire-specific operations |
| Admin action gate | `SupportAction.status == active AND SupportAction.expiresAt > now` | Allow platform admin access to specific tenant resource |

### Trust Hierarchy

```
Level 1 (highest): PlatformAdminUser — isolated domain, explicit SupportAction required
Level 2:           TenantAdmin — full control within own tenant
Level 3:           Fiduciaire user + DelegatedAccessGrant — scoped cross-tenant access
Level 4:           Regular tenant user + Role — standard tenant permissions
Level 5 (lowest):  Viewer / read-only — no write capabilities
```

### Least Privilege Enforcement

- Roles are assigned to the minimum set of permissions required.
- DelegatedAccessGrant is scoped to explicit `scopeCode` values — no wildcard grants.
- Platform admin access to tenant data requires an expiring, justified SupportAction.
- MFA is required for TenantAdmin operations and all PlatformAdminUser operations `[ASSUMPTION]`.
- DelegatedAccessGrant.validUntil enforces time-bound fiduciaire access.

---

## 9. Multi-Tenant Isolation Analysis

### Option A — Shared Schema (Row-Level Isolation)

All tenants share the same database tables. Every table has a `tenant_id` column. Row-Level Security (RLS) is enforced at the database level.

| Dimension | Assessment |
|---|---|
| Scalability | Excellent — single schema scales horizontally |
| Cost | Lowest — shared infrastructure |
| Operational complexity | Low — single migration path |
| Compliance suitability | Moderate — requires rigorous RLS proof; risk of policy bypass |
| Security isolation | Moderate — logical isolation only; a RLS bug exposes all data |
| Auditability | Good — all data in one place; tenant filter on audit queries |

### Option B — Separate Schema (Schema-per-Tenant)

Each tenant has its own schema (set of tables). The same database engine hosts all schemas.

| Dimension | Assessment |
|---|---|
| Scalability | Good — schema-level isolation, easier point-in-time recovery per tenant |
| Cost | Medium — schema management overhead |
| Operational complexity | Medium — migrations must be applied to N schemas |
| Compliance suitability | Good — schema-level permissions demonstrable to auditors |
| Security isolation | Good — schema-level isolation reduces RLS bypass risk |
| Auditability | Good — schema ownership is clear |

### Option C — Separate Database (Database-per-Tenant)

Each tenant has its own database instance.

| Dimension | Assessment |
|---|---|
| Scalability | Complex — connection pooling, instance management |
| Cost | Highest — N database instances |
| Operational complexity | Very high — migrations, monitoring, backups per tenant |
| Compliance suitability | Excellent — physical isolation is demonstrable |
| Security isolation | Highest — no shared infrastructure between tenants |
| Auditability | Excellent — per-tenant logs and backups |

### Recommendation: Tiered Isolation Strategy

```
┌────────────────────────────────────────────────────┐
│ LAYER 1 — Operational data                        │
│ Strategy: Shared schema + strict RLS               │
│ Applies to: tenants, invoices, memberships,        │
│             customers, fiscal years, sessions      │
│ Rationale: Cost-effective, operationally simple    │
│            at current scale                        │
├────────────────────────────────────────────────────┤
│ LAYER 2 — Audit & Compliance data                 │
│ Strategy: Dedicated append-only schema             │
│ Applies to: AuditEvent, ClearanceEvent,            │
│             ComplianceControl, RetentionPolicy     │
│ Rationale: INSERT-only permissions enforced at DB  │
│            level; cannot be modified by app layer  │
├────────────────────────────────────────────────────┤
│ LAYER 3 — Platform administration data            │
│ Strategy: Separate schema, separate connection pool│
│ Applies to: PlatformAdminUser, SupportAction,      │
│             AdminAuditEvent                        │
│ Rationale: Zero sharing with tenant domain;        │
│            separate credentials; separate access   │
└────────────────────────────────────────────────────┘
```

This tiered approach provides compliance-appropriate isolation without the operational complexity of Option C at the initial scale. It can evolve toward separate schemas per tenant for large enterprise customers `[ASSUMPTION A-07]`.

---

## 10. Fiscal Year Model

### Scoping Decision

FiscalYear is **tenant-scoped** `[ASSUMPTION A-05]`. Each tenant manages its own set of fiscal years independently.

Rationale:
- A fiduciaire tenant has its own business fiscal year (for its own accounting).
- A company tenant has its own fiscal year (independent of any fiduciaire managing it).
- A fiduciaire acting on behalf of a company reads invoices in the context of the **company's fiscal year**, not the fiduciaire's own.

### Lifecycle State Machine

```
              create
                │
                ▼
             [open]
                │
                │ TenantAdmin closes (finalize accounting)
                ▼
            [closed] ◀── read-only, no financial writes
                │
                │ System locks after retention trigger or admin action
                ▼
            [locked] ◀── fully immutable, archive-only

  Transitions are irreversible. No rollback.
```

### Invariants

- An Invoice must be associated with an `open` FiscalYear at creation time.
- A `closed` or `locked` FiscalYear rejects all write operations at the service layer.
- A FiscalYear cannot be deleted. It transitions to `locked` and is retained per RetentionPolicy.
- FiscalPeriod (monthly/quarterly) inherits the write-lock of its parent FiscalYear.

### Fiscal Year Context in API Requests

- Every authenticated session carries an `activeFiscalYearId`.
- Switching fiscal year context is an explicit user action — it updates the session.
- Read operations may span fiscal years (e.g., multi-year reporting) via dedicated reporting APIs, not via the standard session context.
- A fiduciaire user accessing a company tenant operates in the **company's** fiscal year context, resolved at cross-tenant access time.

### Regulatory Notes

> `[VERIFY]` Morocco fiscal year rules, DGI requirements on fiscal year declarations, and permitted fiscal year structures (calendar vs offset years) must be verified against the Code Général des Impôts and applicable DGI circulars before implementation.

---

## 11. Admin Console Model

### Trust Boundaries

```
═══════════════════════════════════════════════════════════════
  TRUST BOUNDARY: PLATFORM ADMIN DOMAIN
  Authentication: separate IdP or credential store
  Session model: separate, shorter TTL, MFA mandatory
  Audit log: AdminAuditEvent — isolated storage
═══════════════════════════════════════════════════════════════

  PlatformAdminUser roles:
  ├── super_admin      ─── full platform access, all actions
  ├── support          ─── tenant data access via SupportAction only
  ├── ops              ─── infrastructure and deployment actions
  └── compliance_officer ── read-only on compliance and audit data

═══════════════════════════════════════════════════════════════
  TRUST BOUNDARY: TENANT ADMIN DOMAIN
  Authentication: tenant user credentials
  Session model: tenant-scoped, standard TTL
  Audit log: AuditEvent (tenant-scoped)
═══════════════════════════════════════════════════════════════

  TenantAdmin capabilities (within own tenant only):
  ├── Manage tenant members (invite, create, suspend, revoke)
  ├── Assign and manage roles
  ├── Open and close fiscal years
  ├── Manage organization settings
  ├── Initiate or accept fiduciaire relationships
  └── View tenant audit log (read-only)

═══════════════════════════════════════════════════════════════
  TRUST BOUNDARY: DELEGATED FIDUCIAIRE DOMAIN
  Authentication: fiduciaire user credentials (own tenant)
  Cross-tenant access: via DelegatedAccessGrant only
  Audit log: actions attributed to fiduciaire user, logged in company tenant audit
═══════════════════════════════════════════════════════════════

  Fiduciaire user with active grant:
  ├── Access company data within granted RelationshipScope
  ├── Perform operations on company behalf within scope
  └── All actions attributed to the fiduciaire user identity

═══════════════════════════════════════════════════════════════
  TRUST BOUNDARY: REGULAR TENANT USER
  Authentication: tenant user credentials
  Scope: own tenant only, bounded by assigned role
═══════════════════════════════════════════════════════════════
```

### Key Separation Rules

1. A PlatformAdminUser account cannot also be a tenant User — separate identity spaces.
2. Platform admin access to tenant data requires an active, expiring, documented SupportAction — no implicit access.
3. All platform admin actions emit AdminAuditEvent — stored in an isolated schema with no tenant-side access.
4. Tenant admins cannot escalate to platform admin capabilities — no privilege path between trust boundaries.
5. A fiduciaire user accessing a company sees that company's data attributed to their own identity — no anonymization of cross-tenant actions.

---

## 12. Security Risk Analysis

| Risk | Description | Likelihood | Impact | Mitigations |
|---|---|---|---|---|
| **R-01 Tenant isolation failure** | Missing `tenantId` filter exposes data across tenants | Medium | Critical | RLS at DB layer; mandatory `tenantId` in all repository methods; integration tests asserting cross-tenant data leakage |
| **R-02 Privilege escalation** | User manipulates role or membership to gain elevated access | Low | High | Role changes require TenantAdmin; all role assignments emit AuditEvent; roles are server-side only |
| **R-03 Stale delegated access** | DelegatedAccessGrant remains active after relationship revocation | Medium | Critical | Revocation synchronously deactivates all grants — not eventual; grants are validated on every cross-tenant request |
| **R-04 Unauthorized fiduciaire access** | Fiduciaire reads company data beyond granted scope | Medium | High | RelationshipScope enforced at access control layer; scope codes are explicit; over-permissive grants are audited |
| **R-05 Audit tampering** | Attempt to modify or delete AuditEvent records | Low | Critical | INSERT-only DB permissions on audit tables; separate schema; no application-layer delete path; integrity checksums |
| **R-06 Invoice integrity violation** | Invoice content modified after DGI clearance submission | Low | Critical | Invoice is read-only once `clearanceStatus` ≥ `pending_clearance`; write attempts are rejected at service layer |
| **R-07 Relationship abuse** | Fiduciaire creates relationship to access data without genuine company consent | Low | High | Dual-party acceptance required; invitation expiry; all relationship events audited; company can revoke at any time |
| **R-08 Admin misuse** | Platform admin accesses tenant data without justification | Low | High | Mandatory SupportAction with justification; dual authorization for critical actions; AdminAuditEvent — all admin actions logged |
| **R-09 Fiscal year corruption** | Write operation against closed or locked fiscal year | Medium | High | FiscalYearGuard evaluated before every financial write; service-layer enforcement independent of API layer |
| **R-10 Insider threat** | Authorized user exfiltrates data | Medium | High | Least privilege role assignments; AuditEvent on all reads of sensitive resources; access reviews; data export requires explicit audit trail |
| **R-11 Session hijacking** | Session token stolen and used to impersonate user | Low | Critical | Short session TTL; tenant context re-validation on sensitive operations; IP binding on admin sessions `[ASSUMPTION]` |
| **R-12 External party in fiduciaire relationship** | Attempt to create fiduciaire relationship with non-platform entity | Low | High | Tenant existence and status validation required before relationship creation; enforced at domain service level, not just API |
| **R-13 Clearance replay attack** | Duplicate clearance submission processed by DGI | Low | Medium | Idempotency keys on clearance submissions; ClearanceEvent sequence tracking `[VERIFY: DGI idempotency guarantees]` |
| **R-14 Cryptographic key exposure** | Invoice signing keys leaked | Very Low | Critical | Keys stored in HSM or dedicated KMS; never in application memory longer than signing operation; key rotation policy |

---

## 13. Open Architectural Decisions

Items requiring validation before implementation begins, ordered by blocking priority.

| Priority | Decision | Options | Who Must Decide |
|---|---|---|---|
| P0 | **DGI integration protocol** — exact API format, authentication method, clearance flow | `[VERIFY: official DGI documentation required]` | Regulatory/Legal + DGI liaison |
| P0 | **CNDP declaration requirements** — which treatments require prior declaration | `[VERIFY: CNDP official guidance]` | Legal + DPO |
| P1 | **Database technology** — PostgreSQL, MySQL, other | Impacts RLS implementation, schema design | Engineering lead |
| P1 | **Monorepo tooling** — Nx, Turborepo, pnpm workspaces | Impacts build pipeline and dev experience | Engineering lead |
| P1 | **Session strategy** — stateless JWT vs server-side sessions | Impacts tenant context propagation, revocation latency | Architecture + Security |
| P1 | **FiscalYear: Morocco-specific calendar** — can fiscal years be non-calendar? | `[VERIFY: DGI CGI applicable articles]` | Regulatory |
| P1 | **TaxProfile: DGI VAT regime codes** — exact values and applicable rules | `[VERIFY: DGI official reference]` | Regulatory |
| P2 | **HSM provider** — for invoice signing keys in production | Cloud HSM vs on-premise | Infrastructure + Security |
| P2 | **Email/SMS provider** — notification delivery | SendGrid, Mailgun, Twilio, other | Engineering |
| P2 | **Maximum fiduciaire relationships per company** — cardinality business rule | Unlimited vs capped | Product |
| P2 | **FiduciaireRelationship expiry** — auto-expiration after inactivity? | Business and compliance consideration | Product + Legal |
| P2 | **Billing platform** — external billing system | Stripe, custom, other | Product + Finance |
| P3 | **Event sourcing for audit domain** — full event sourcing vs hybrid | Impacts auditability and replay capability | Architecture |
| P3 | **GDPR applicability analysis** — formal legal analysis of EU customer exposure | Requires legal review | Legal |
| P3 | **Multi-org per tenant** — future: can one tenant have multiple legal entities? | Currently 1:1 assumed | Product roadmap |

---

## 14. Implementation Phase Recommendations

Phases are sequenced by dependency and compliance criticality. No code is specified here — only domain areas and rationale.

---

### Phase 1 — Identity, Tenant Foundation & Admin Isolation
**Rationale**: Everything depends on identity and tenant isolation. Admin isolation must be established from day one — retrofitting is expensive and risky.

- Identity & Authentication context (User, Credential, Session, MFAFactor)
- Tenant Management context (Tenant, TenantProfile)
- Organization Management context (Organization)
- Access Control context (Role, Permission, TenantMembership, first-user-admin bootstrap)
- Platform Administration context (PlatformAdminUser, AdminAuditEvent — isolated from day one)
- Audit Domain foundation (AuditEvent append-only infrastructure)

**Exit criteria**: A user can register, verify email, create a tenant, become tenant admin, and invite another user. Platform admin can log in to a separate console with MFA.

---

### Phase 2 — Fiscal Context & Fiduciaire Relationships
**Rationale**: Fiscal year is a blocker for invoicing. Fiduciaire relationships are a core differentiator and their security model must be designed before any delegated access is built.

- Fiscal Context (FiscalYear, FiscalPeriod, TaxProfile)
- Fiduciaire Relationship Management (Invitation, FiduciaireRelationship, RelationshipScope)
- Access Control extensions (DelegatedAccessGrant, ABAC policy evaluation)
- FiscalYearGuard enforcement

**Exit criteria**: A fiduciaire tenant can invite a company tenant. The company accepts. Delegated access is granted and enforced. Both parties can revoke. Fiscal years can be opened and closed.

---

### Phase 3 — Invoicing Domain
**Rationale**: Core product capability. Depends on fiscal context and counterparty management.

- Customer & Counterparty Management (Customer, Supplier, ExternalParty)
- Invoicing Domain (Invoice, InvoiceLine, InvoiceSeries, CreditNote, PaymentRecord)
- Invoice format validation (DGI format — requires Phase 0 regulatory clarity)

**Exit criteria**: A tenant can create, edit (draft), and finalize an invoice with correct numbering, counterparty data, and fiscal year context.

---

### Phase 4 — Clearance Domain
**Rationale**: Regulatory requirement. Depends on DGI integration specifications obtained before implementation.

- Clearance Domain (ClearanceEvent, InvoiceResponse)
- DGI Gateway integration (requires official DGI documentation)
- Bidirectional clearance status tracking
- Client notification on invoice delivery

**Exit criteria**: A signed invoice can be submitted to DGI for clearance. Clearance outcome is tracked. Cleared invoice is delivered to the client tenant or external party. Client can respond.

---

### Phase 5 — Compliance, Audit Hardening & Retention
**Rationale**: Required before operating in production with real fiscal data.

- Compliance Domain (ComplianceControl, RetentionPolicy, AccessReview)
- AuditEvent hardening (append-only enforcement, integrity checksums)
- RetentionPolicy implementation per regulatory requirements
- CNDP compliance controls (requires legal validation from Phase 0)

**Exit criteria**: All significant domain events are audited. Retention policies are enforced. Compliance control registry is populated for DGI, DGSSI, CNDP.

---

### Phase 6 — Subscription, Billing & Notifications
**Rationale**: Commercial operations layer. Depends on core product being stable.

- Subscription & Billing (Subscription, Plan, FeatureFlag)
- Notification Domain (Notification, templates, channel routing)
- Feature gating enforcement across all contexts

**Exit criteria**: Tenants are on plans. Features are gated by plan. Notifications are delivered for key events (invoice received, relationship invitation, clearance outcome).

---

### Pre-Implementation Phase 0 — Regulatory Clarity (Parallel to Phase 1)

Before Phase 3 and Phase 4 can be completed, the following must be resolved:

1. Official DGI documentation on invoice format, clearance protocol, sequential numbering rules
2. CNDP legal guidance on required declarations for this treatment profile
3. Morocco fiscal year legal framework (CGI applicable articles)
4. DGSSI cybersecurity requirements for this platform category
5. Legal analysis of GDPR applicability for EU-facing transactions

These items are non-negotiable blockers for regulatory features and must be pursued in parallel with Phase 1 engineering work.

---

*End of document — e-fatourati Canonical Domain Model v1.0*
