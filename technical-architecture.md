# Technical Architecture Document
## Property Management SaaS Platform

**Version:** 1.0
**Date:** November 2025
**Strategic Alignment:** Based on saas-strategy-market-leadership.md
**Target:** Multi-country property management platform for European market

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Architecture Principles & Assumptions](#architecture-principles--assumptions)
3. [System Architecture Overview](#system-architecture-overview)
4. [Infrastructure Architecture](#infrastructure-architecture)
5. [Application Architecture](#application-architecture)
6. [Data Architecture](#data-architecture)
7. [Security Architecture](#security-architecture)
8. [Integration Architecture](#integration-architecture)
9. [Multi-tenancy Strategy](#multi-tenancy-strategy)
10. [Internationalization & Localization](#internationalization--localization)
11. [Scalability & Performance](#scalability--performance)
12. [Deployment & DevOps](#deployment--devops)
13. [Monitoring & Observability](#monitoring--observability)
14. [Technology Stack](#technology-stack)
15. [Migration & Rollout Strategy](#migration--rollout-strategy)
16. [Cost Optimization](#cost-optimization)
17. [Risk Mitigation](#risk-mitigation)

---

## Executive Summary

### Business Context

This technical architecture supports a **€36B market opportunity** across 6 European countries, targeting 301,983+ property management companies. The platform must:

- **Scale from 50 to 50,000+ customers over 5 years**
- **Support 6 countries with different languages, regulations, and integrations**
- **Achieve 99.9% uptime (SaaS standard)**
- **Process millions of transactions monthly (invoices, payments, communications)**
- **Maintain <200ms average response time**
- **Support mobile-first usage patterns (60%+ mobile traffic expected)**

### Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Cloud Provider** | AWS (primary) | EU data residency, comprehensive services, mature PropTech ecosystem |
| **Architecture Pattern** | Modular Monolith → Microservices | Start simple, extract services as needed (pragmatic evolution) |
| **Primary Language** | TypeScript/Node.js | Full-stack consistency, strong ecosystem, developer productivity |
| **Frontend Framework** | React (web), React Native (mobile) | Code sharing, mature ecosystem, hiring pool |
| **Database** | PostgreSQL (primary), Redis (cache) | ACID compliance, JSON support, proven at scale |
| **Multi-tenancy** | Shared database, row-level isolation | Cost efficiency, easier operations, adequate security |
| **API Strategy** | GraphQL (client apps), REST (integrations) | Flexible queries, mobile efficiency, partner compatibility |
| **Deployment** | AWS ECS Fargate | Serverless containers, auto-scaling, zero ops overhead, cost-effective |

---

## Architecture Principles & Assumptions

### Core Principles

**1. Pragmatic Scalability**
- **Principle:** Build for current scale + 10x, not 100x
- **Rationale:** Avoid premature optimization; 1,000 customers is very different from 50,000
- **Example:** Start with modular monolith, extract microservices when specific bottlenecks emerge

**2. European Data Sovereignty**
- **Principle:** All customer data stays in EU (GDPR compliance)
- **Rationale:** Legal requirement + customer trust (especially Germany)
- **Example:** AWS eu-central-1 (Frankfurt) as primary region

**3. Country-First Architecture**
- **Principle:** Localization is core, not an afterthought
- **Rationale:** Professional translation + regulatory compliance = competitive moat
- **Example:** I18n database tables, country-specific module system, legal template engine

**4. Mobile-First Performance**
- **Principle:** Optimize for mobile networks and devices
- **Rationale:** 60%+ of users will be mobile (property managers on-site)
- **Example:** GraphQL for efficient data fetching, aggressive caching, progressive web app

**5. Security by Design**
- **Principle:** Multi-layered security from day one
- **Rationale:** Financial data + compliance = zero tolerance for breaches
- **Example:** Encryption at rest/transit, row-level security, audit logs, SOC 2 compliance

**6. Developer Velocity**
- **Principle:** Ship features fast, iterate based on feedback
- **Rationale:** 2-3 year window to capture market before consolidation
- **Example:** TypeScript full-stack, shared code, automated testing, CI/CD

### Key Assumptions

**Assumption 1: Customer Growth Trajectory**
- Month 12: 1,000 customers
- Month 24: 5,000 customers
- Month 36: 10,000 customers
- Month 60: 30,000 customers
- **Architecture Impact:** Plan for 3x growth annually

**Assumption 2: Data Volume**
- Average customer: 500 units, 2 users, 50 owners
- Total at Month 36: 5M units, 20K users, 500K owners
- Transactions: 10M invoices/year, 100M communications/year
- **Architecture Impact:** Multi-region database, sharding strategy for Year 3+

**Assumption 3: Geographic Distribution**
- 70% Germany/Austria (Month 24)
- 60% Germany/Austria, 30% France, 10% Italy (Month 36)
- **Architecture Impact:** eu-central-1 (Frankfurt) primary, eu-west-3 (Paris) by Month 24

**Assumption 4: Integration Complexity**
- Germany: DATEV (accounting), 20+ German banks, SEPA
- France: 15+ French banks, specific accounting standards
- Italy: 10+ Italian banks, ANACI integration
- **Architecture Impact:** Pluggable integration framework, country adapters

**Assumption 5: Regulatory Requirements**
- GDPR compliance (all countries)
- GoBD compliance (Germany - financial record retention)
- Data residency (no data leaves EU)
- SOC 2 Type II (by Month 18 for enterprise sales)
- **Architecture Impact:** Encryption, audit logs, data retention policies, compliance monitoring

---

## System Architecture Overview

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  Web App (React)  │  iOS App (React Native)  │  Android App     │
│  Owner Portal     │  Mobile Browser (PWA)    │  (React Native)  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   CDN (CloudFront) │
                    │   + API Gateway    │
                    └─────────┬──────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────┐
│                    APPLICATION LAYER                           │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │         Core Platform (Modular Monolith - Phase 1)       │ │
│  ├──────────────────────────────────────────────────────────┤ │
│  │  • Property Management Module                            │ │
│  │  • Financial Management Module                           │ │
│  │  • Communication Module                                  │ │
│  │  • Assembly/Meeting Module                               │ │
│  │  • Compliance & Reporting Module                         │ │
│  │  • Vendor/Maintenance Module                             │ │
│  │  • User/Auth Module                                      │ │
│  │  • I18n/Localization Module                              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                                 │
│  Extracted Services (Phase 2+):                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │ Document     │  │ Integration  │  │ Analytics    │         │
│  │ Processing   │  │ Hub          │  │ Engine       │         │
│  │ Service      │  │ (DATEV, etc) │  │ (ML/AI)      │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────┐
│                      DATA LAYER                                │
├────────────────────────────────────────────────────────────────┤
│  PostgreSQL (RDS)  │  Redis (ElastiCache)  │  S3 (Documents)  │
│  - Multi-tenant    │  - Session cache      │  - File storage  │
│  - Transactions    │  - Application cache  │  - Backups       │
│  - Primary data    │  - Rate limiting      │  - Exports       │
└────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────┐
│                   INFRASTRUCTURE LAYER                         │
├────────────────────────────────────────────────────────────────┤
│  ECS Fargate  │  VPC  │  ALB  │  CloudWatch  │  Secrets Mgr   │
└────────────────────────────────────────────────────────────────┘
```

### Architecture Evolution Path

**Phase 1 (Months 0-12): Modular Monolith**
- Single deployable application with clear module boundaries
- All modules in one codebase but logically separated
- Shared database with clear schema namespaces
- **Why:** Fast development, easier debugging, sufficient for 1,000 customers
- **When to evolve:** When team size >20 engineers OR specific module bottleneck

**Phase 2 (Months 13-24): Selective Extraction**
- Extract bottleneck services: Document Processing, Integration Hub
- Keep core business logic in monolith
- Microservices communicate via message queue (SQS/SNS)
- **Why:** Scale specific bottlenecks, enable independent deployment
- **When to evolve:** When traffic >100 req/sec OR multi-country complexity

**Phase 3 (Months 25-36): Multi-Region Platform**
- Add regional deployments (Paris for France)
- Event-driven architecture for cross-region sync
- Extract more services: Analytics, Notifications, Billing
- **Why:** Performance for distant users, data residency compliance
- **When to evolve:** When France/Italy >30% of customer base

**Phase 4 (Months 37+): Domain-Driven Microservices**
- Full microservices architecture by bounded contexts
- Service mesh for observability
- Event sourcing for critical workflows
- **Why:** Team autonomy (50+ engineers), independent scaling, failure isolation

---

## Infrastructure Architecture

### Cloud Provider: AWS

**Decision:** Amazon Web Services (AWS)

**Rationale:**
1. **EU Data Centers:** eu-central-1 (Frankfurt), eu-west-3 (Paris) for data residency
2. **Mature Services:** RDS, EKS, S3, CloudFront - proven at scale
3. **PropTech Ecosystem:** Many German startups use AWS (developer familiarity)
4. **Compliance:** SOC 2, ISO 27001, GDPR-compliant infrastructure
5. **Cost:** Startup credits + reserved instances = cost-effective at our scale

**Alternatives Considered:**
- **Google Cloud (GCP):** Strong Kubernetes (GKE), but less PropTech presence in EU
- **Azure:** Good EU presence, but less startup ecosystem, more enterprise-focused
- **Multi-cloud:** Unnecessary complexity at our scale, avoid

### Regional Architecture

**Primary Region:** eu-central-1 (Frankfurt, Germany)
- **Rationale:** Germany = 60-70% of customers (Month 24), low latency for largest segment
- **Services:** All core infrastructure, primary database, production workloads

**Secondary Region:** eu-west-3 (Paris, France) - Month 24+
- **Rationale:** France expansion (Phase 3), reduce latency for French/Belgian customers
- **Services:** Read replicas, CDN edge locations, regional API deployments
- **Data Sync:** Async replication for owner portals, sync for critical writes

**Backup Region:** eu-west-1 (Ireland)
- **Rationale:** Disaster recovery, geographic diversity from Frankfurt
- **Services:** Database backups, encrypted snapshots, failover capacity

### Availability Zones Strategy

**Production:** Multi-AZ deployment across 3 availability zones
- Load balancers in all AZs
- Application servers in all AZs (minimum 2 per AZ)
- Database: Multi-AZ RDS with automatic failover
- **Target:** 99.95% uptime (SLA: 99.9%)

**Development/Staging:** Single AZ (cost optimization)

### Network Architecture

```
VPC (10.0.0.0/16)
│
├── Public Subnets (10.0.0.0/20) - 3 AZs
│   ├── NAT Gateways
│   ├── Application Load Balancers
│   └── Bastion Hosts (locked down)
│
├── Private Subnets - Application (10.0.16.0/20) - 3 AZs
│   ├── ECS Fargate Tasks (serverless containers)
│   ├── Application Containers
│   └── No direct internet access (via NAT)
│
├── Private Subnets - Data (10.0.32.0/20) - 3 AZs
│   ├── RDS PostgreSQL (Multi-AZ)
│   ├── ElastiCache Redis
│   ├── Encrypted at rest
│   └── No internet access
│
└── VPC Endpoints
    ├── S3 Endpoint (private S3 access)
    ├── Secrets Manager Endpoint
    └── CloudWatch Endpoint
```

**Security Groups:**
- Web tier: Allow 443 from CloudFront/ALB only
- App tier: Allow from web tier + internal services
- Data tier: Allow from app tier only, explicit port rules (5432, 6379)

### DNS & CDN

**DNS:** Route 53
- Health checks with automatic failover
- Latency-based routing for multi-region
- Example: `app.propertysaas.de`, `app.propertysaas.fr`

**CDN:** CloudFront
- Edge locations across EU (40+ locations)
- Cache static assets (JS, CSS, images) - 1 day TTL
- Cache API responses (owner portals) - 5 min TTL with cache invalidation
- **Performance:** Reduce latency from 200ms → 20ms for static content

---

## Application Architecture

### Modular Monolith Design (Phase 1)

**Decision:** Start with modular monolith, extract microservices as needed

**Rationale:**
1. **Development Speed:** Faster for small team (5-10 engineers) in Year 1
2. **Easier Debugging:** Single codebase, easier to trace issues
3. **Transactions:** Cross-module ACID transactions (property + billing)
4. **Refactoring:** Can evolve to microservices without client changes (API abstraction)
5. **Proven Pattern:** Shopify, GitHub started this way - works for $1B+ companies

**Module Boundaries** (enforced by dependency rules):

```
src/
├── modules/
│   ├── property/              # Property & Unit Management
│   │   ├── domain/            # Business logic
│   │   ├── application/       # Use cases
│   │   ├── infrastructure/    # DB, external services
│   │   └── interface/         # API controllers
│   │
│   ├── financial/             # Billing, Payments, Accounting
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── interface/
│   │
│   ├── communication/         # Owner Portal, Messages, Notifications
│   ├── assembly/              # Meetings, Voting, Minutes
│   ├── compliance/            # Reports, Filings, Audit Trails
│   ├── vendor/                # Maintenance, Vendor Management
│   ├── user/                  # Auth, Users, Permissions
│   ├── i18n/                  # Localization, Translations
│   └── integration/           # DATEV, Banks, External APIs
│
├── shared/
│   ├── kernel/                # Cross-cutting: events, errors, types
│   ├── infrastructure/        # DB connection, cache, queue
│   └── utils/                 # Helpers, validators
│
└── api/
    ├── graphql/               # GraphQL schema, resolvers
    ├── rest/                  # REST endpoints (partners)
    └── webhooks/              # Incoming webhooks (banks, DATEV)
```

**Dependency Rules:**
- Modules CANNOT import from other modules directly
- Cross-module communication via **Domain Events** (in-memory event bus initially)
- Example: `FinancialModule` publishes `InvoiceCreatedEvent`, `CommunicationModule` listens and sends notification
- **Future-proof:** When extracting to microservices, events become message queue (SQS)

### API Architecture

**Decision:** Dual API strategy - GraphQL + REST

**GraphQL API** (primary for client apps)
- **Use Case:** Web app, mobile apps, owner portal
- **Rationale:**
  - **Efficient Mobile:** Fetch exact data needed (reduce over-fetching)
  - **Strongly Typed:** Auto-generated TypeScript types (frontend/backend consistency)
  - **Flexible:** Add fields without versioning
  - **Developer Experience:** GraphQL Playground for development
- **Technology:** Apollo Server, GraphQL Code Generator
- **Example Query:**
  ```graphql
  query GetProperty($id: ID!) {
    property(id: $id) {
      id
      name
      units {
        id
        number
        owner { name, email }
      }
      upcomingAssemblies(limit: 3) {
        id
        date
        agenda
      }
    }
  }
  ```

**REST API** (for integrations)
- **Use Case:** Partner integrations, webhooks, DATEV, banks
- **Rationale:**
  - **Industry Standard:** DATEV, banks expect REST
  - **Webhooks:** REST is standard for callbacks
  - **Documentation:** OpenAPI/Swagger well-understood
- **Technology:** Express.js, OpenAPI 3.0 spec
- **Versioning:** URL-based (`/api/v1/properties`)

**API Gateway:** AWS ALB + custom middleware
- Rate limiting (by tenant, by API key)
- Authentication (JWT for users, API keys for partners)
- Request validation
- Logging & metrics

### Frontend Architecture

**Web Application**

**Technology:** React 18 + TypeScript
- **State Management:** React Query (server state) + Zustand (UI state)
- **Routing:** React Router v6
- **Styling:** Tailwind CSS (utility-first, fast development)
- **Forms:** React Hook Form + Zod (validation)
- **Build:** Vite (fast dev server, optimized builds)

**Architecture Pattern:** Feature-based structure
```
src/
├── features/
│   ├── properties/
│   │   ├── PropertyList.tsx
│   │   ├── PropertyDetail.tsx
│   │   ├── useProperties.ts         # React Query hooks
│   │   └── propertySchema.ts        # Zod validation
│   ├── financial/
│   ├── assemblies/
│   └── ...
├── shared/
│   ├── components/                   # Button, Table, Modal
│   ├── hooks/                        # useAuth, useI18n
│   └── utils/
└── i18n/
    ├── de.json                        # German translations
    ├── fr.json                        # French translations
    └── it.json                        # Italian translations
```

**Performance Optimizations:**
- Code splitting by route (React.lazy)
- Progressive Web App (PWA) for offline access
- Virtual scrolling for large lists (react-virtual)
- Optimistic updates (immediate UI feedback)
- Image optimization (WebP, lazy loading)

**Mobile Applications**

**Technology:** React Native + TypeScript

**Rationale:**
1. **Code Sharing:** 70-80% code shared with web (business logic, API layer)
2. **Developer Efficiency:** One team builds web + mobile
3. **Mature Ecosystem:** Strong for business apps (Shopify, Coinbase use RN)
4. **Native Performance:** Hermes engine, native modules for critical paths

**Architecture:**
- Shared business logic (hooks, API calls, state management)
- Platform-specific UI (native components, gestures)
- Separate bundles for iOS/Android (no "universal app")

**Critical Native Features:**
- Camera (OCR for documents, invoice scanning)
- Push notifications (assembly reminders, payment alerts)
- Biometric auth (Face ID, fingerprint)
- Offline mode (sync when reconnected)

**Owner Portal** (for property owners)

**Technology:** Separate lightweight React app
- Simplified interface (view-only, no complex admin features)
- Public-facing (SEO-friendly, server-side rendering)
- Multi-language (owner chooses language)
- Mobile-optimized (80%+ of owners use mobile)

**Hosting:** Vercel or Netlify (edge deployment, automatic HTTPS)

---

## Data Architecture

### Database Strategy

**Primary Database:** PostgreSQL 15+ (AWS RDS)

**Rationale:**
1. **ACID Compliance:** Financial transactions require strong consistency
2. **JSON Support:** Store country-specific data flexibly (`jsonb` columns)
3. **Full-Text Search:** Property search, owner search (built-in)
4. **Mature:** Battle-tested, extensive tooling, large community
5. **Extensions:** PostGIS (future: map features), pg_cron (scheduled tasks)
6. **Scalability:** Proven to 10TB+ with proper indexing

**Configuration:**
- **Instance Type:** db.r6g.xlarge (Month 1-6) → db.r6g.2xlarge (Month 12+)
- **Storage:** 100GB gp3 → 1TB gp3 (auto-scaling enabled)
- **Multi-AZ:** Yes (automatic failover, <60 sec downtime)
- **Backups:** Automated daily, 30-day retention, point-in-time recovery
- **Encryption:** At rest (KMS), in transit (SSL/TLS)

**Read Replicas:**
- Month 12+: 2 read replicas for reporting, analytics
- Route read-only queries to replicas (reduce primary load)

### Database Schema Design

**Multi-Tenancy Model:** Shared database, tenant isolation via `tenant_id`

**Core Tables Structure:**

```sql
-- Tenant/Account table (top-level isolation)
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  country_code VARCHAR(2) NOT NULL,  -- DE, FR, IT, AT, ES, BE
  language_code VARCHAR(5) NOT NULL,  -- de-DE, fr-FR, it-IT
  plan_tier VARCHAR(50) NOT NULL,     -- starter, professional, business
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  settings JSONB,                     -- Country-specific settings
  subscription_id UUID REFERENCES subscriptions(id)
);

-- Row-Level Security (RLS) for tenant isolation
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON tenants
  USING (id = current_setting('app.current_tenant_id')::UUID);

-- Properties table (core entity)
CREATE TABLE properties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  address JSONB NOT NULL,             -- Flexible for country formats
  type VARCHAR(50) NOT NULL,          -- condominium, apartment_building
  total_units INTEGER NOT NULL,
  metadata JSONB,                     -- Country-specific: WEG number (DE), etc.
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_properties_tenant ON properties(tenant_id);
CREATE INDEX idx_properties_metadata ON properties USING GIN(metadata);

-- Units table
CREATE TABLE units (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  property_id UUID NOT NULL REFERENCES properties(id) ON DELETE CASCADE,
  unit_number VARCHAR(50) NOT NULL,
  floor INTEGER,
  square_meters DECIMAL(10,2),
  ownership_percentage DECIMAL(5,4),  -- For cost allocation
  owner_id UUID REFERENCES owners(id),
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Financial transactions (append-only for audit)
CREATE TABLE financial_transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  property_id UUID REFERENCES properties(id),
  type VARCHAR(50) NOT NULL,          -- invoice, payment, expense
  amount DECIMAL(15,2) NOT NULL,
  currency VARCHAR(3) NOT NULL DEFAULT 'EUR',
  status VARCHAR(50) NOT NULL,        -- pending, paid, overdue
  due_date DATE,
  metadata JSONB,                     -- Invoices, receipts, bank refs
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_transactions_tenant_date ON financial_transactions(tenant_id, created_at DESC);
CREATE INDEX idx_transactions_status ON financial_transactions(status) WHERE status != 'paid';

-- Audit log (immutable, append-only)
CREATE TABLE audit_logs (
  id BIGSERIAL PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id),
  action VARCHAR(100) NOT NULL,       -- created_invoice, sent_email, etc.
  entity_type VARCHAR(100) NOT NULL,
  entity_id UUID,
  changes JSONB,                      -- Before/after state
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Partitioned by month (performance for large logs)
CREATE INDEX idx_audit_tenant_time ON audit_logs(tenant_id, created_at DESC);
```

**Internationalization Tables:**

```sql
-- Translations (for system messages, templates)
CREATE TABLE translations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key VARCHAR(255) NOT NULL,          -- e.g., "invoice.overdue_notice"
  language_code VARCHAR(5) NOT NULL,  -- de-DE, fr-FR
  value TEXT NOT NULL,
  context VARCHAR(100),               -- For translation context
  UNIQUE(key, language_code)
);

-- Country-specific configurations
CREATE TABLE country_configs (
  country_code VARCHAR(2) PRIMARY KEY,
  config JSONB NOT NULL               -- Tax rates, date formats, regulations
);

-- Legal templates (localized)
CREATE TABLE legal_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  country_code VARCHAR(2) NOT NULL,
  template_type VARCHAR(100) NOT NULL, -- assembly_minutes, invoice
  language_code VARCHAR(5) NOT NULL,
  content TEXT NOT NULL,               -- Template with {{variables}}
  version INTEGER NOT NULL DEFAULT 1,
  active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Partitioning Strategy (Month 24+):**
- `audit_logs`: Partition by month (easier archival, query performance)
- `financial_transactions`: Partition by year (regulatory retention)
- `communications`: Partition by quarter (high volume)

### Caching Strategy

**Redis (AWS ElastiCache):**

**Use Cases:**
1. **Session Storage:** User sessions (JWT refresh tokens)
2. **Application Cache:** Frequently accessed data (property lists, user permissions)
3. **Rate Limiting:** API rate limits per tenant/user
4. **Temporary Data:** OTP codes, password reset tokens
5. **Real-time Features:** Online users, live notifications (Pub/Sub)

**Configuration:**
- **Instance Type:** cache.r6g.large (Month 1-12) → cache.r6g.xlarge (Month 24+)
- **Cluster Mode:** Enabled (sharding for >100GB data)
- **Replication:** Multi-AZ with automatic failover
- **Eviction Policy:** allkeys-lru (evict least recently used)

**Cache Patterns:**

```typescript
// Cache-Aside Pattern (lazy loading)
async function getProperty(propertyId: string, tenantId: string) {
  const cacheKey = `property:${tenantId}:${propertyId}`;

  // Check cache first
  let property = await redis.get(cacheKey);
  if (property) {
    return JSON.parse(property);
  }

  // Cache miss - fetch from DB
  property = await db.query(
    'SELECT * FROM properties WHERE id = $1 AND tenant_id = $2',
    [propertyId, tenantId]
  );

  // Store in cache (TTL: 1 hour)
  await redis.setex(cacheKey, 3600, JSON.stringify(property));

  return property;
}

// Cache Invalidation (on write)
async function updateProperty(propertyId: string, data: any) {
  await db.query('UPDATE properties SET ... WHERE id = $1', [propertyId]);

  // Invalidate cache
  await redis.del(`property:${data.tenantId}:${propertyId}`);

  // Invalidate list caches
  await redis.del(`properties:list:${data.tenantId}`);
}
```

**Cache TTLs:**
- User sessions: 7 days (sliding window)
- Property data: 1 hour (changes infrequent)
- Owner lists: 30 minutes (moderate changes)
- Dashboard stats: 5 minutes (frequent changes)
- Country configs: 24 hours (static)

### Document Storage

**AWS S3:**

**Bucket Structure:**
```
production-documents-eu-central-1/
├── {tenant-id}/
│   ├── contracts/
│   ├── invoices/
│   ├── photos/
│   ├── minutes/
│   └── exports/
```

**Features:**
- **Versioning:** Enabled (regulatory compliance, accidental deletion protection)
- **Lifecycle Policies:** Archive to Glacier after 1 year (90% cost reduction)
- **Encryption:** Server-side encryption (SSE-KMS)
- **Access Control:** Presigned URLs (time-limited, user-specific)
- **CDN:** CloudFront for fast delivery (especially photos)

**Upload Flow:**
1. Client requests upload URL (GraphQL mutation)
2. Server generates presigned S3 URL (valid 15 minutes)
3. Client uploads directly to S3 (no server bandwidth)
4. Client notifies server of completion (stores metadata in DB)

### Data Backup & Recovery

**Database Backups:**
- **Automated Daily:** RDS automated backups (30-day retention)
- **Manual Snapshots:** Before major deployments
- **Cross-Region Backup:** Daily snapshot to eu-west-1 (Ireland) for DR
- **Point-in-Time Recovery:** Up to 5 minutes before failure

**Recovery Time Objectives:**
- **RTO (Recovery Time Objective):** <1 hour for database
- **RPO (Recovery Point Objective):** <5 minutes of data loss

**Backup Testing:** Quarterly restore tests to staging environment

---

## Security Architecture

### Security Principles

1. **Defense in Depth:** Multiple layers of security
2. **Least Privilege:** Minimum necessary permissions
3. **Encryption Everywhere:** Data at rest + in transit
4. **Audit Everything:** Comprehensive logging
5. **Zero Trust:** Verify every request

### Authentication & Authorization

**User Authentication:**

**Technology:** JWT (JSON Web Tokens) + Refresh Tokens

**Flow:**
1. User login (email + password)
2. Server validates credentials (bcrypt hash comparison)
3. Server issues:
   - **Access Token:** Short-lived (15 minutes), contains user claims
   - **Refresh Token:** Long-lived (7 days), stored in httpOnly cookie
4. Client includes access token in Authorization header
5. When access token expires, use refresh token to get new access token

**Token Claims:**
```json
{
  "sub": "user-uuid",
  "tenant_id": "tenant-uuid",
  "role": "admin",
  "permissions": ["property.write", "financial.read"],
  "iat": 1234567890,
  "exp": 1234568800
}
```

**Password Security:**
- **Hashing:** bcrypt (cost factor: 12)
- **Requirements:** Minimum 12 characters, zxcvbn strength checker
- **MFA:** TOTP (Google Authenticator) for Business/Enterprise tiers
- **Password Reset:** Time-limited tokens (15 minutes), single-use

**OAuth 2.0 (Future - Month 12+):**
- Social login (Google, Microsoft) for owner portal
- SSO for enterprise customers (SAML 2.0)

**Authorization:**

**Model:** Role-Based Access Control (RBAC) + Permission-Based

**Roles:**
- **Owner:** Administrator role (full access)
- **Admin:** Senior staff (most access)
- **User:** Regular employee (limited access)
- **Viewer:** Read-only (reports, exports)
- **Accountant:** Financial access only (for external accountants)

**Permissions:** Granular (e.g., `property.create`, `invoice.delete`)

**Implementation:**
```typescript
// Middleware for permission checking
function requirePermission(permission: string) {
  return (req, res, next) => {
    const user = req.user; // From JWT

    if (!user.permissions.includes(permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
}

// Usage
app.post('/api/properties',
  requirePermission('property.create'),
  createProperty
);
```

**Tenant Isolation:**
- Every query includes `tenant_id` filter (enforced at ORM level)
- Database row-level security (RLS) as additional safeguard
- Middleware injects `tenant_id` from JWT into all queries

### Data Encryption

**Encryption at Rest:**
- **Database:** RDS encryption with AWS KMS (AES-256)
- **S3 Documents:** Server-side encryption (SSE-KMS)
- **Backups:** Encrypted snapshots
- **Redis:** ElastiCache encryption enabled

**Encryption in Transit:**
- **HTTPS Only:** TLS 1.3, redirect HTTP → HTTPS
- **Certificate:** AWS Certificate Manager (auto-renewal)
- **Database Connections:** SSL/TLS required
- **Internal Services:** mTLS (mutual TLS) for service-to-service

**Key Management:**
- **AWS KMS:** Customer-managed keys (CMK)
- **Key Rotation:** Automatic annual rotation
- **Secrets:** AWS Secrets Manager for API keys, DB credentials
- **Never in Code:** No secrets in source code (environment variables only)

### Network Security

**Firewall Rules:**
- **Web Tier:** Allow 443 from internet (CloudFront only)
- **App Tier:** No direct internet access (outbound via NAT gateway)
- **Data Tier:** No internet access, only from app tier

**DDoS Protection:**
- **AWS Shield Standard:** Automatic (free)
- **Rate Limiting:** API Gateway (1000 req/hour per IP, 10000 req/hour per tenant)
- **WAF (Month 12+):** AWS WAF for SQL injection, XSS protection

**VPN Access:**
- **Bastion Hosts:** SSH access for emergency debugging (MFA required)
- **VPN:** AWS Client VPN for developers (temporary access)

### Compliance & Auditing

**GDPR Compliance:**
- **Data Minimization:** Collect only necessary data
- **Right to Access:** Export user data (GraphQL query)
- **Right to Delete:** Hard delete user data (30-day grace period)
- **Data Portability:** JSON/CSV export
- **Consent Management:** Track consent for communications
- **DPO (Data Protection Officer):** Appoint by Month 12

**Audit Logging:**
- **What to Log:** User actions, API calls, data changes, access attempts
- **Immutable:** Append-only `audit_logs` table
- **Retention:** 7 years (German GoBD compliance)
- **Searchable:** Indexed by tenant, user, action, date

**SOC 2 Type II (Month 18+):**
- Required for enterprise sales
- Third-party audit of security controls
- Continuous monitoring and reporting

### Vulnerability Management

**Dependency Scanning:**
- **npm audit:** Run on every build (CI/CD)
- **Snyk:** Continuous monitoring for vulnerabilities
- **Automated PRs:** Dependabot for security updates

**Penetration Testing:**
- **Month 6:** First internal pentest
- **Month 12:** External pentest (third-party)
- **Annual:** Ongoing external pentests

**Security Headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'; ...
```

---

## Integration Architecture

### Integration Framework

**Challenge:** Support 50+ integrations (banks, accounting, government APIs) across 6 countries

**Architecture Decision:** Pluggable Integration Hub with Country Adapters

**Design Pattern:**
```typescript
// Abstract integration interface
interface Integration {
  connect(credentials: any): Promise<Connection>;
  sync(data: any): Promise<SyncResult>;
  disconnect(): Promise<void>;
}

// Country-specific adapters
class DATEVIntegration implements Integration {
  // Germany-specific DATEV integration
}

class OECDIntegration implements Integration {
  // French accounting integration
}

// Integration registry (plugin system)
class IntegrationRegistry {
  private integrations = new Map<string, Integration>();

  register(name: string, integration: Integration) {
    this.integrations.set(name, integration);
  }

  get(name: string): Integration {
    return this.integrations.get(name);
  }
}

// Usage
const registry = new IntegrationRegistry();
registry.register('datev', new DATEVIntegration());
registry.register('oecd', new OECDIntegration());
```

### Critical Integrations

**Germany: DATEV**

**Priority:** P0 (blocking for German market)

**Integration Type:** REST API + File Export

**Data Flow:**
1. Export transactions from our platform (monthly)
2. Transform to DATEV CSV format
3. Upload via DATEV REST API or SFTP
4. Accountant imports into DATEV software

**Implementation:**
- **Library:** Custom TypeScript adapter
- **Format:** DATEV CSV (specific column structure)
- **Scheduling:** Automated monthly export, manual on-demand
- **Error Handling:** Retry logic, email notifications on failure

**SEPA Banking (All Countries):**

**Integration Type:** SEPA XML + Bank APIs

**Data Flow:**
1. Generate SEPA XML for payments (pain.001)
2. Submit via bank API (FinTS/HBCI for Germany, EBICS)
3. Receive payment confirmations (camt.053)
4. Reconcile against invoices

**Banks:**
- **Germany:** Deutsche Bank, Commerzbank, Sparkasse (FinTS)
- **France:** BNP Paribas, Société Générale (EBICS)
- **Multi-country:** Wise, Stripe (API-based)

**Document OCR:**

**Technology:** AWS Textract + Custom ML model

**Use Cases:**
- Extract invoice data from photos/PDFs
- Parse bank statements
- Digitize paper contracts

**Flow:**
1. User uploads document (photo/PDF) → S3
2. Trigger AWS Textract (async processing)
3. Extract text + structured data (tables, key-value pairs)
4. Custom ML model (trained on invoices) refines extraction
5. Return structured data to frontend for review/confirmation

**Cost:** ~$0.015 per page (10,000 pages/month = $150)

### Webhooks & Events

**Outgoing Webhooks:** Notify partners of events

**Use Cases:**
- Notify accountant when monthly closing is ready
- Notify building manager when maintenance request created
- Notify owner when payment received

**Implementation:**
```typescript
// Webhook delivery system
class WebhookService {
  async deliver(url: string, event: any) {
    const signature = this.generateSignature(event);

    try {
      await axios.post(url, event, {
        headers: {
          'X-Webhook-Signature': signature,
          'Content-Type': 'application/json'
        },
        timeout: 5000
      });
    } catch (error) {
      // Retry with exponential backoff (3 attempts)
      await this.retry(url, event);
    }
  }

  private generateSignature(event: any): string {
    // HMAC-SHA256 for security
    return crypto
      .createHmac('sha256', process.env.WEBHOOK_SECRET)
      .update(JSON.stringify(event))
      .digest('hex');
  }
}
```

**Incoming Webhooks:** Receive events from partners

**Use Cases:**
- Bank payment notifications (SEPA confirmations)
- DATEV sync confirmations
- Stripe payment events

**Security:**
- Signature verification (HMAC)
- IP whitelist (for known partners)
- Rate limiting (prevent abuse)

### API Partner Ecosystem

**Month 12+:** Launch Integration Marketplace

**Partner Categories:**
1. **Accounting:** DATEV, Lexware, Sevdesk
2. **Banking:** Deutsche Bank, Commerzbank, Wise
3. **Legal:** Contract management, e-signature (DocuSign)
4. **Insurance:** Professional liability insurers
5. **Property Services:** Maintenance providers, cleaning services
6. **Energy:** Energy certificate providers (for EU Green Deal)

**Developer Portal:**
- API documentation (OpenAPI spec)
- Sandbox environment
- API keys management
- Usage analytics
- Support forum

**Revenue Model (Future):**
- Free for basic integrations
- Revenue share for premium integrations (e.g., 20% of fees)

---

## Multi-tenancy Strategy

### Tenancy Model Decision

**Decision:** Shared Database, Row-Level Isolation

**Alternatives Evaluated:**

| Model | Pros | Cons | Decision |
|-------|------|------|----------|
| **Database per Tenant** | Strong isolation, easy to scale specific tenants | Expensive (100+ databases), complex operations, slow provisioning | ❌ Rejected |
| **Schema per Tenant** | Good isolation, easier operations than DB-per-tenant | Still complex (1000+ schemas), migrations harder | ❌ Rejected |
| **Shared DB, Row-Level** | Cost-efficient, easy operations, fast provisioning | Risk of data leakage (requires careful coding) | ✅ **Selected** |

**Rationale for Shared DB:**
1. **Cost:** $200/month for 10,000 tenants vs $200,000/month for DB-per-tenant
2. **Operations:** Single database to backup, upgrade, monitor
3. **Provisioning:** Instant tenant creation (INSERT row) vs minutes for new DB
4. **Scale:** Proven to 100,000+ tenants (Salesforce, Shopify use this model)
5. **Risk Mitigation:** Strong coding standards + RLS + automated tests

### Tenant Isolation Mechanisms

**1. Application-Level Isolation (Primary):**

```typescript
// ORM-level tenant filtering (Prisma example)
// NEVER allow queries without tenant_id

// BAD - Missing tenant filter
const properties = await db.property.findMany();

// GOOD - Tenant filter enforced
const properties = await db.property.findMany({
  where: { tenant_id: req.user.tenant_id }
});

// Middleware to enforce tenant_id
app.use((req, res, next) => {
  // Extract tenant from JWT
  req.tenantId = req.user.tenant_id;

  // Set session variable for RLS
  db.query('SET app.current_tenant_id = $1', [req.tenantId]);

  next();
});
```

**2. Database Row-Level Security (Defense in Depth):**

```sql
-- Enable RLS on all tenant tables
ALTER TABLE properties ENABLE ROW LEVEL SECURITY;
ALTER TABLE units ENABLE ROW LEVEL SECURITY;
ALTER TABLE financial_transactions ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their tenant's data
CREATE POLICY tenant_isolation_policy ON properties
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Enforce for all users except superuser
ALTER TABLE properties FORCE ROW LEVEL SECURITY;
```

**3. Automated Testing:**

```typescript
// Test: Ensure tenant isolation
describe('Tenant Isolation', () => {
  it('should not allow access to other tenant data', async () => {
    const tenant1 = await createTenant();
    const tenant2 = await createTenant();

    const property = await createProperty({ tenant_id: tenant1.id });

    // Switch to tenant 2
    const result = await getProperty(property.id, tenant2.id);

    expect(result).toBeNull(); // Should not access tenant1's data
  });
});
```

### Tenant Lifecycle Management

**Provisioning:**
1. User signs up (email, company name, country)
2. Create tenant record (UUID, settings)
3. Send welcome email with setup link
4. User completes onboarding (import data, invite team)
5. Activate subscription (Stripe Checkout)

**Offboarding:**
1. User cancels subscription
2. Grace period (30 days) - read-only access
3. Export all data (GDPR compliance)
4. Hard delete after grace period (anonymize for audit logs)
5. Send confirmation email

**Tenant Settings (Per-Tenant Customization):**

```json
{
  "country_code": "DE",
  "language_code": "de-DE",
  "currency": "EUR",
  "timezone": "Europe/Berlin",
  "date_format": "DD.MM.YYYY",
  "number_format": "1.234,56",
  "fiscal_year_start": "01-01",
  "branding": {
    "logo_url": "https://...",
    "primary_color": "#1a73e8",
    "custom_domain": "portal.example.de"
  },
  "integrations": {
    "datev": { "enabled": true, "api_key": "..." },
    "bank_account": { "iban": "DE..." }
  },
  "features": {
    "advanced_reporting": true,
    "api_access": false,
    "white_label_portal": false
  }
}
```

### Resource Limits (Prevent Noisy Neighbors)

**Enforcement:**
- API rate limiting: 1000 req/hour per tenant (Starter), 10000 (Business)
- Storage quota: 5GB (Starter), 50GB (Business), unlimited (Enterprise)
- Database query timeout: 30 seconds (prevent long-running queries)
- Concurrent connections: 10 per tenant

**Monitoring:**
- Track resource usage per tenant
- Alert when tenant approaches limits
- Throttle before hard limits (gradual degradation)

---

## Internationalization & Localization

### I18n Strategy

**Framework:** i18next (industry standard, 100k+ GitHub stars)

**Architecture:**
```typescript
// Backend: Translation keys stored in database
const translations = {
  'de-DE': {
    'invoice.overdue': 'Rechnung überfällig',
    'property.created': 'Immobilie erstellt'
  },
  'fr-FR': {
    'invoice.overdue': 'Facture en retard',
    'property.created': 'Propriété créée'
  }
};

// Frontend: Load translations based on user language
import i18n from 'i18next';

i18n.init({
  lng: user.language_code,
  resources: {
    de: { translation: deTranslations },
    fr: { translation: frTranslations }
  }
});

// Usage in React
import { useTranslation } from 'react-i18next';

function InvoiceAlert() {
  const { t } = useTranslation();
  return <div>{t('invoice.overdue')}</div>;
}
```

### Country-Specific Formatting

**Dates:**
- Germany: DD.MM.YYYY (31.12.2025)
- France: DD/MM/YYYY (31/12/2025)
- Italy: DD/MM/YYYY (31/12/2025)

**Numbers:**
- Germany: 1.234,56 (period = thousands, comma = decimal)
- France: 1 234,56 (space = thousands, comma = decimal)
- Italy: 1.234,56 (period = thousands, comma = decimal)

**Currency:**
- All countries: EUR (€)
- Format: €1.234,56 (Germany), 1 234,56 € (France)

**Implementation:**
```typescript
// Use Intl API (built-in JavaScript)
const formatter = new Intl.NumberFormat(user.language_code, {
  style: 'currency',
  currency: 'EUR'
});

formatter.format(1234.56);
// de-DE: "1.234,56 €"
// fr-FR: "1 234,56 €"
```

### Legal Templates System

**Challenge:** Different legal requirements per country (assembly minutes, invoices, contracts)

**Solution:** Template engine with country-specific templates

```typescript
// Template definition (Handlebars syntax)
const assemblyMinutesDE = `
PROTOKOLL DER EIGENTÜMERVERSAMMLUNG

Datum: {{date}}
Ort: {{location}}
WEG: {{weg_number}}

Anwesende Eigentümer: {{attendees}}
...
Beschlüsse:
{{#each resolutions}}
- {{this.description}} ({{this.votes_yes}} Ja, {{this.votes_no}} Nein)
{{/each}}
`;

// Generate document
const doc = generateDocument('assembly_minutes', 'DE', {
  date: '31.12.2025',
  location: 'Berlin',
  weg_number: '12345',
  attendees: ['Max Mustermann', 'Erika Musterfrau'],
  resolutions: [
    { description: 'Dachsanierung', votes_yes: 8, votes_no: 2 }
  ]
});
```

**Template Management:**
- Store in database (versioned)
- Country-specific templates reviewed by local lawyers
- Easy to update (no code deployment)
- PDF generation (Puppeteer for complex layouts)

### Localization Workflow

**Process:**
1. **Developer:** Add English strings as keys (`invoice.overdue`)
2. **Extract:** CI/CD extracts new keys to translation files
3. **Translate:** Professional translators (not Google Translate) translate to DE, FR, IT
4. **Review:** Native speakers review for quality
5. **Deploy:** Translations pushed to production (no code change)

**Tools:**
- **Lokalise:** Translation management platform
- **CI/CD Integration:** Auto-sync translations on deployment

**Quality:**
- Professional translators (legal/financial domain expertise)
- Native speaker review (especially legal templates)
- Context provided (screenshots, explanations)
- NO machine translation for legal/financial content

---

## Scalability & Performance

### Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| **API Response Time (p50)** | <100ms | Feels instant, mobile-friendly |
| **API Response Time (p95)** | <500ms | Acceptable for complex queries |
| **API Response Time (p99)** | <2s | Edge cases, heavy reports |
| **Page Load Time (p50)** | <1s | Fast perceived performance |
| **Time to Interactive (TTI)** | <3s | User can start using app |
| **Database Query Time (p95)** | <50ms | Fast queries = fast APIs |
| **Uptime** | 99.9% | ~8 hours downtime/year (standard SaaS) |

### Scalability Strategy

**Vertical Scaling (Months 0-12):**
- Increase instance sizes (db.r6g.xlarge → 2xlarge)
- Increase connection pools
- Optimize queries (indexes, EXPLAIN ANALYZE)
- **Target:** Support 1,000-5,000 customers

**Horizontal Scaling (Months 12-24):**
- Add read replicas (reporting, analytics)
- Add application server instances (auto-scaling)
- CDN for static assets (CloudFront)
- **Target:** Support 5,000-15,000 customers

**Sharding (Months 24+, if needed):**
- Shard by tenant_id (large tenants get dedicated shards)
- Example: Tenants 1-1000 → Shard 1, Tenants 1001-2000 → Shard 2
- Application routing layer (transparent to clients)
- **Target:** Support 50,000+ customers

**Auto-Scaling Configuration:**

```yaml
# Kubernetes HPA (Horizontal Pod Autoscaler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-server
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-server
  minReplicas: 3  # Always at least 3 (multi-AZ)
  maxReplicas: 30 # Scale up to 30 during peak
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale when CPU >70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Database Optimization

**Query Optimization:**
1. **Index Strategy:**
   ```sql
   -- Tenant isolation (every multi-tenant query)
   CREATE INDEX idx_properties_tenant ON properties(tenant_id);

   -- Frequent filters
   CREATE INDEX idx_transactions_status ON financial_transactions(status)
     WHERE status != 'paid';

   -- Composite indexes for common queries
   CREATE INDEX idx_transactions_tenant_date
     ON financial_transactions(tenant_id, created_at DESC);

   -- JSONB indexes for flexible queries
   CREATE INDEX idx_properties_metadata ON properties USING GIN(metadata);
   ```

2. **Connection Pooling:**
   ```typescript
   // PgBouncer for connection pooling
   const pool = new Pool({
     host: 'pgbouncer.internal',
     port: 6432,
     max: 20,           // Max connections per app instance
     idleTimeoutMillis: 30000,
     connectionTimeoutMillis: 2000
   });
   ```

3. **Query Analysis:**
   - Weekly review of slow queries (CloudWatch RDS)
   - EXPLAIN ANALYZE for complex queries
   - Add indexes based on actual usage patterns

**Read Replicas:**
- Route analytics queries to replicas
- Route reporting to replicas
- Primary for all writes + real-time reads

### Caching Strategy

**Multi-Layer Caching:**

1. **CDN (CloudFront):** Static assets, owner portal pages
2. **Application Cache (Redis):** API responses, user sessions
3. **Database Cache:** PostgreSQL shared_buffers (25% of RAM)
4. **Client Cache:** Browser cache, React Query (stale-while-revalidate)

**Cache Invalidation:**
```typescript
// Event-based cache invalidation
eventBus.on('property.updated', async (event) => {
  const { propertyId, tenantId } = event;

  // Invalidate specific caches
  await redis.del(`property:${tenantId}:${propertyId}`);
  await redis.del(`properties:list:${tenantId}`);

  // Invalidate CDN (if cached)
  await cloudfront.invalidate(`/api/properties/${propertyId}`);
});
```

### Load Testing

**Tools:** k6 (open-source load testing)

**Test Scenarios:**
1. **Normal Load:** 100 concurrent users, 10 req/sec
2. **Peak Load:** 500 concurrent users, 50 req/sec
3. **Stress Test:** 2000 concurrent users, 200 req/sec (find breaking point)
4. **Soak Test:** 100 users for 24 hours (memory leaks, connection issues)

**Schedule:**
- Pre-launch: Comprehensive load testing
- Monthly: Regression tests (ensure performance doesn't degrade)
- Before major events: E.g., end-of-month billing surge

---

## Deployment & DevOps

### Container Orchestration: ECS Fargate

**Decision:** AWS ECS (Elastic Container Service) with Fargate (serverless)

**Rationale:**
1. **Serverless Containers:** No cluster management, AWS manages infrastructure
2. **Cost-Effective:** Pay only for vCPU/memory used, no idle costs (vs EKS $73/month for control plane)
3. **Simple Operations:** No Kubernetes complexity, no worker node patching
4. **Auto-Scaling:** Built-in, scales based on CPU/memory or custom metrics
5. **AWS Integration:** Native integration with ALB, CloudWatch, Secrets Manager, IAM
6. **Fast Deployment:** Faster to set up than EKS (minutes vs hours)
7. **Perfect for Monolith:** Single container deployment, straightforward architecture

**Why NOT Kubernetes/EKS:**
- **Complexity:** Kubernetes has steep learning curve, overkill for monolith
- **Cost:** EKS control plane = $73/month + worker nodes = $200-400/month minimum
- **Operational Overhead:** Node upgrades, security patching, cluster management
- **Team Size:** Need Kubernetes expertise, harder to hire for early-stage startup

**ECS Fargate Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Internet / CloudFront                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │  Application Load Balancer (ALB)    │
        │  - Multi-AZ (3 availability zones)  │
        │  - Health checks                    │
        │  - SSL termination                  │
        └────────────────┬───────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                  │
        ▼                                  ▼
┌──────────────┐                  ┌──────────────┐
│ ECS Service  │                  │ ECS Service  │
│ (Production) │                  │ (Staging)    │
└──────┬───────┘                  └──────┬───────┘
       │                                  │
       │ Target: 4 tasks                 │ Target: 2 tasks
       │ Min: 2, Max: 20                 │ Min: 1, Max: 4
       │                                  │
  ┌────┴─────┬──────┬──────┐       ┌────┴────┐
  ▼          ▼      ▼      ▼        ▼         ▼
┌──────┐  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ Task │  │ Task │ │ Task │ │ Task │ │ Task │ │ Task │
│ AZ-1 │  │ AZ-2 │ │ AZ-3 │ │ AZ-1 │ │ AZ-1 │ │ AZ-2 │
└──┬───┘  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
   │         │        │        │        │        │
   └─────────┴────────┴────────┴────────┴────────┘
                      │
        ┌─────────────┴──────────────┐
        │                            │
        ▼                            ▼
  ┌────────────┐              ┌────────────┐
  │ RDS        │              │ Redis      │
  │ PostgreSQL │              │ ElastiCache│
  └────────────┘              └────────────┘
```

**ECS Task Definition:**

```json
{
  "family": "property-management-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789.dkr.ecr.eu-central-1.amazonaws.com/app:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:eu-central-1:123:secret:db-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/property-management",
          "awslogs-region": "eu-central-1",
          "awslogs-stream-prefix": "app"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

**Auto-Scaling Configuration:**

```yaml
# Target Tracking Scaling Policy
ScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: cpu-scaling-policy
    PolicyType: TargetTrackingScaling
    ScalingTargetId: !Ref ScalableTarget
    TargetTrackingScalingPolicyConfiguration:
      TargetValue: 70.0  # Target 70% CPU utilization
      PredeclaredMetricSpecification:
        PredeclaredMetricType: ECSServiceAverageCPUUtilization
      ScaleInCooldown: 300   # 5 min before scale down
      ScaleOutCooldown: 60   # 1 min before scale up

# Also scale on memory
MemoryScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: memory-scaling-policy
    PolicyType: TargetTrackingScaling
    TargetTrackingScalingPolicyConfiguration:
      TargetValue: 80.0  # Target 80% memory utilization
      PredeclaredMetricSpecification:
        PredeclaredMetricType: ECSServiceAverageMemoryUtilization
```

**Resource Allocation by Environment:**

| Environment | CPU | Memory | Tasks (Min/Target/Max) | Monthly Cost |
|-------------|-----|--------|------------------------|--------------|
| **Development** | 256 | 512 MB | 1/1/2 | ~$15 |
| **Staging** | 512 | 1 GB | 1/2/4 | ~$45 |
| **Production** | 1024 | 2 GB | 2/4/20 | ~$180 (avg load) |

### CI/CD Pipeline

**Tools:** GitHub Actions (CI/CD - no separate CD tool needed)

**Pipeline Stages:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to ECS

on:
  push:
    branches: [main, develop]

env:
  AWS_REGION: eu-central-1
  ECR_REPOSITORY: property-management-app
  ECS_SERVICE_PROD: app-production
  ECS_SERVICE_STAGING: app-staging
  ECS_CLUSTER: property-management-cluster
  CONTAINER_NAME: app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Run E2E tests
        run: npm run test:e2e

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build-and-push:
    needs: [test, security]
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build-image.outputs.image }}

    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build Docker image
          docker build \
            --build-arg NODE_ENV=production \
            --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
            --build-arg VCS_REF=${{ github.sha }} \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:latest \
            .

          # Push to ECR
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

          # Output image URI
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' || github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.propertysaas.de

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Download task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition app-staging \
            --query taskDefinition > task-definition.json

      - name: Fill in new image ID in task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ needs.build-and-push.outputs.image }}

      - name: Deploy to ECS Staging
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE_STAGING }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true

      - name: Run smoke tests
        run: |
          # Wait for deployment
          sleep 30
          # Health check
          curl -f https://staging.propertysaas.de/health || exit 1

  deploy-production:
    needs: [build-and-push, deploy-staging]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://app.propertysaas.de

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Download task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition app-production \
            --query taskDefinition > task-definition.json

      - name: Fill in new image ID in task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ needs.build-and-push.outputs.image }}

      - name: Deploy to ECS Production (Blue-Green)
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE_PROD }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
          # ECS manages blue-green deployment automatically

      - name: Verify deployment
        run: |
          # Wait for all tasks to be healthy
          sleep 60
          # Run production smoke tests
          curl -f https://app.propertysaas.de/health || exit 1
          curl -f https://app.propertysaas.de/api/health/db || exit 1

      - name: Notify deployment success
        if: success()
        run: |
          # Send Slack notification
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text":"✅ Production deployment successful: ${{ github.sha }}"}'

      - name: Rollback on failure
        if: failure()
        run: |
          # ECS automatically keeps previous task definition
          # Manual rollback: update service to previous task definition revision
          aws ecs update-service \
            --cluster ${{ env.ECS_CLUSTER }} \
            --service ${{ env.ECS_SERVICE_PROD }} \
            --task-definition app-production:PREVIOUS_REVISION \
            --force-new-deployment
```

**Dockerfile (Optimized for Production):**

```dockerfile
# Multi-stage build for smaller image size
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy source
COPY . .

# Build TypeScript
RUN npm run build

# Production image
FROM node:20-alpine

# Security: Run as non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

WORKDIR /app

# Copy built artifacts and dependencies
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "dist/server.js"]
```

### Deployment Strategy

**Blue-Green Deployment:**
- Deploy new version alongside old version
- Route small % of traffic to new version (canary)
- Monitor metrics (error rates, latency)
- If healthy: gradually increase traffic to 100%
- If unhealthy: instant rollback (route back to old version)

**Database Migrations:**
- **Backward-compatible migrations only** (for zero-downtime)
- Example: Adding column → OK, Removing column → Requires 2-step deployment

```typescript
// Good: Backward-compatible
// Step 1: Add column (nullable)
ALTER TABLE properties ADD COLUMN new_field VARCHAR(255);

// Step 2: Backfill data
UPDATE properties SET new_field = ...;

// Step 3 (next deployment): Make NOT NULL
ALTER TABLE properties ALTER COLUMN new_field SET NOT NULL;

// Bad: Breaking change
ALTER TABLE properties DROP COLUMN old_field; // Old code will break!
```

**Rollback Plan:**
- Every deployment can rollback within 60 seconds
- Database migrations are versioned (can migrate up/down)
- Feature flags for risky features (disable instantly if issues)

### Environment Strategy

**Environments:**
1. **Local:** Developer laptop (Docker Compose)
2. **Development:** Shared dev environment (auto-deploys from `develop` branch)
3. **Staging:** Production-like (auto-deploys from `main` branch)
4. **Production:** Customer-facing (manual approval required)

**Environment Parity:**
- Staging = Production (same instance types, same data volume via anonymized copies)
- Catch production issues in staging before they impact customers

### Infrastructure as Code

**Tool:** Terraform

**Why:**
- Version-controlled infrastructure
- Reproducible environments (spin up staging identical to production)
- Prevent configuration drift
- Easy disaster recovery (rebuild from Terraform)

**Structure:**
```
terraform/
├── modules/
│   ├── vpc/
│   ├── rds/
│   ├── ecs/                      # ECS cluster, services, tasks
│   ├── alb/                      # Application Load Balancer
│   └── s3/
├── environments/
│   ├── staging/
│   │   └── main.tf
│   └── production/
│       └── main.tf
└── terraform.tfstate (S3 backend)
```

### Monitoring & Alerting

**Metrics (CloudWatch):**
- Application: Request rate, error rate, latency (p50, p95, p99)
- Database: CPU, connections, slow queries, replication lag
- Infrastructure: CPU, memory, disk, network

**Logging (CloudWatch Logs):**
- Structured JSON logs (easy to parse)
- Log levels: DEBUG (dev only), INFO, WARN, ERROR
- Correlation IDs (trace requests across services)

**Alerting (PagerDuty):**
- **P1 (Critical):** Production down, database unreachable → Page on-call engineer
- **P2 (High):** Error rate >5%, latency p95 >2s → Slack notification
- **P3 (Medium):** Disk >80%, memory >90% → Email notification

**Dashboards (Grafana):**
- Real-time metrics visualization
- Customer-facing status page (status.propertysaas.com)

---

## Monitoring & Observability

### Observability Pillars

**1. Metrics (What is happening):**
- Request rates, error rates, latencies
- Business metrics: Signups, active users, MRR
- Infrastructure: CPU, memory, disk

**2. Logs (Detailed events):**
- Application logs (errors, warnings, info)
- Audit logs (user actions for compliance)
- Access logs (API requests)

**3. Traces (Request flow):**
- Distributed tracing (follow request through microservices)
- Identify bottlenecks (which service is slow)

### Logging Strategy

**Structured Logging:**
```typescript
// BAD: Unstructured
console.log('User created property: ' + propertyId);

// GOOD: Structured JSON
logger.info('property.created', {
  property_id: propertyId,
  tenant_id: tenantId,
  user_id: userId,
  duration_ms: 45,
  correlation_id: req.id
});
```

**Log Levels:**
- **DEBUG:** Verbose details (development only)
- **INFO:** Normal operations (property created, invoice sent)
- **WARN:** Unexpected but recoverable (API rate limit hit, retry successful)
- **ERROR:** Errors requiring attention (failed to send email, database connection lost)

**Log Aggregation:**
- CloudWatch Logs (AWS native, automatic)
- Future (Month 12+): ELK Stack (Elasticsearch, Logstash, Kibana) for advanced search

### Application Performance Monitoring (APM)

**Tool:** Datadog or New Relic (Month 12+)

**Features:**
- **Transaction tracing:** See slow database queries in context
- **Error tracking:** Group similar errors, track resolution
- **Real user monitoring:** Actual user experience (page load times)
- **Alerting:** Intelligent alerts (anomaly detection, not just thresholds)

**Example Trace:**
```
Request: POST /api/properties
├─ Authentication: 5ms
├─ Validation: 3ms
├─ Database Query: 120ms  ⚠️ SLOW
│  └─ Query: SELECT * FROM units WHERE property_id = ...
├─ Cache Write: 2ms
└─ Total: 135ms
```

### Business Metrics Dashboard

**Key Metrics:**
- **Growth:** MRR, ARR, customer count, churn rate
- **Engagement:** DAU/MAU, feature adoption, session duration
- **Performance:** API latency, error rate, uptime
- **Operations:** Support tickets, deployment frequency, incident count

**Tools:**
- **Mixpanel / Amplitude:** Product analytics (user behavior)
- **ChartMogul:** SaaS metrics (MRR, churn, cohorts)
- **Grafana:** Infrastructure metrics (custom dashboards)

**Example Dashboard:**
```
┌─────────────────────────────────────────────────────────┐
│  MRR: €149,000  ↑ 15%     Customers: 1,000  ↑ 12%      │
│  Churn: 3.2%    ↓ 0.5%    Uptime: 99.97%   ✓           │
├─────────────────────────────────────────────────────────┤
│  API Latency (p95): 245ms                               │
│  [Graph showing latency over time]                      │
├─────────────────────────────────────────────────────────┤
│  Top Features (by usage):                               │
│  1. Invoice Generation: 12,450 uses/week               │
│  2. Owner Portal: 8,300 logins/week                    │
│  3. Document Upload: 6,200 uploads/week                │
└─────────────────────────────────────────────────────────┘
```

### Incident Management

**Process:**
1. **Detection:** Automated alerts (PagerDuty, CloudWatch)
2. **Response:** On-call engineer investigates (5-minute SLA)
3. **Mitigation:** Fix or rollback (30-minute SLA for P1)
4. **Communication:** Status page updates (every 30 minutes)
5. **Resolution:** Permanent fix deployed
6. **Post-Mortem:** Blameless analysis, action items (within 48 hours)

**Post-Mortem Template:**
```markdown
# Incident Post-Mortem: Database Outage (2025-12-15)

## Summary
Database became unresponsive at 14:32 UTC, affecting all customers.
Duration: 23 minutes. Impacted: 100% of users.

## Timeline
- 14:32: Alerts triggered (high database CPU)
- 14:35: On-call engineer paged
- 14:40: Root cause identified (long-running query from analytics)
- 14:45: Query killed, database recovered
- 14:55: Service fully restored

## Root Cause
Analytics query without LIMIT clause scanned 100M rows.
No query timeout configured.

## Action Items
1. [DONE] Add query timeout (30 seconds) - @engineer1
2. [TODO] Add LIMIT to all analytics queries - @engineer2
3. [TODO] Add slow query alerts (>10 seconds) - @engineer3
4. [TODO] Review all queries for performance - @team

## Lessons Learned
- Query timeouts are critical for multi-tenant systems
- Need better visibility into running queries
- Analytics should use read replicas (not primary)
```

---

## Technology Stack

### Backend Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Language** | TypeScript | Type safety, modern JavaScript, developer productivity |
| **Runtime** | Node.js 20 LTS | Mature, fast, huge ecosystem, AWS Lambda compatible |
| **Framework** | Express.js | Lightweight, flexible, well-documented, middleware ecosystem |
| **GraphQL** | Apollo Server | Industry standard, great TypeScript support, caching |
| **ORM** | Prisma | Type-safe, great DX, migrations, multi-database |
| **Validation** | Zod | TypeScript-first, composable, excellent errors |
| **Authentication** | jsonwebtoken + passport | Battle-tested, flexible, supports OAuth/SAML |
| **Testing** | Jest + Supertest | Fast, great mocking, coverage reports |

### Frontend Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Framework** | React 18 | Industry standard, huge ecosystem, hiring pool |
| **Language** | TypeScript | Type safety, refactoring confidence, IDE support |
| **Build Tool** | Vite | Fast dev server (instant HMR), optimized production builds |
| **State (Server)** | React Query | Caching, refetching, optimistic updates, devtools |
| **State (Client)** | Zustand | Simple, minimal boilerplate, TypeScript-friendly |
| **Routing** | React Router v6 | Declarative, nested routes, code splitting |
| **Styling** | Tailwind CSS | Utility-first, fast development, small bundle size |
| **Forms** | React Hook Form | Performance (uncontrolled), simple API, validation |
| **UI Components** | Radix UI + Custom | Accessible, unstyled (full control), composable |
| **Testing** | Vitest + Playwright | Fast (Vite-compatible), E2E for critical flows |

### Mobile Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Framework** | React Native | Code sharing with web, mature ecosystem |
| **Language** | TypeScript | Same as web (consistency) |
| **Navigation** | React Navigation | Standard for RN, deep linking, native feel |
| **State** | React Query + Zustand | Same as web (code sharing) |
| **UI** | React Native Paper | Material Design, accessible, customizable |
| **Storage** | AsyncStorage + SQLite | Offline data (SQLite), simple KV (AsyncStorage) |
| **Push** | Firebase Cloud Messaging | Free, reliable, cross-platform |
| **Analytics** | Mixpanel | Cross-platform (web + mobile), funnels, cohorts |

### Infrastructure Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Cloud** | AWS | EU data centers, mature services, compliance certifications |
| **Container** | Docker | Industry standard, reproducible builds |
| **Orchestration** | ECS Fargate | Serverless containers, no cluster management, cost-effective |
| **Database** | PostgreSQL 15 | ACID, JSON support, full-text search, proven at scale |
| **Cache** | Redis 7 | Fast, versatile (cache, sessions, pub/sub, rate limiting) |
| **Object Storage** | S3 | Durable (99.999999999%), cheap, integrates with everything |
| **CDN** | CloudFront | Global edge network, tight AWS integration |
| **DNS** | Route 53 | Health checks, latency routing, AWS integration |
| **Monitoring** | CloudWatch + Grafana | CloudWatch (native), Grafana (visualization) |
| **IaC** | Terraform | Version-controlled infrastructure, multi-cloud capable |
| **CI/CD** | GitHub Actions | Integrated with GitHub, generous free tier, flexible |

### Third-Party Services

| Service | Purpose | Rationale |
|---------|---------|-----------|
| **Stripe** | Payments, subscriptions | Industry standard, excellent API, supports EU |
| **SendGrid** | Transactional emails | Reliable, templates, analytics, GDPR-compliant |
| **Twilio** | SMS (2FA, notifications) | Reliable, global coverage, EU data centers |
| **AWS Textract** | OCR for documents | Accurate, handles German/French/Italian, pay-per-use |
| **Lokalise** | Translation management | Developer-friendly, CI/CD integration, context for translators |
| **PagerDuty** | Incident management | Reliable alerting, on-call schedules, escalation |
| **Sentry** | Error tracking | Real-time alerts, source maps, release tracking |
| **Mixpanel** | Product analytics | User behavior, funnels, retention cohorts |
| **ChartMogul** | SaaS metrics | MRR, churn, LTV, Stripe integration |

---

## Migration & Rollout Strategy

### Customer Onboarding & Data Migration

**Target Customers:** Property managers using Excel/paper/legacy software

**Migration Challenges:**
1. **Data Quality:** Inconsistent formats, missing data, errors
2. **Volume:** Average 500 units × 1000 customers = 500,000 records
3. **Downtime:** Customers can't afford days of downtime
4. **Training:** New software requires learning curve

**Migration Process:**

**Phase 1: Preparation (Week 1)**
1. Customer exports data from existing system (Excel template provided)
2. Customer uploads to our platform (secure form)
3. Automated validation (check required fields, formats)
4. Report validation errors to customer (fix and re-upload)

**Phase 2: Import (Week 2)**
1. Customer reviews mapped data in staging environment
2. Customer confirms correctness
3. One-click import to production account
4. Automated sanity checks (unit counts, balances)

**Phase 3: Parallel Run (Weeks 3-4)**
1. Customer uses both systems in parallel (safety net)
2. Verify outputs match (invoices, reports)
3. Build confidence in new system

**Phase 4: Go-Live (Week 5)**
1. Customer switches fully to our platform
2. Archive old system (keep for reference)

**Support:**
- **Self-Service:** Video tutorials, import guide, FAQ
- **Assisted Migration:** 1-hour onboarding call (Business/Enterprise tiers)
- **Professional Services:** We migrate data for customer (€500-2000, optional)

### Rollout by Market

**Germany (Months 0-12):**
- Start: Berlin, Munich, Hamburg (top 3 cities)
- Expand: Add 1-2 cities per month (Frankfurt, Cologne, Stuttgart, etc.)
- Support: German-speaking support team (initially small, scale with customers)
- Partnerships: DDIV, BVI (professional associations) for credibility

**Austria (Months 15-24):**
- Start: Vienna (44% of Austrian market)
- Expand: Salzburg, Graz (Month 18)
- Support: Same German-speaking team (no additional hires needed)
- Localization: Minimal (Austrian banking, legal templates)

**France (Months 25-30):**
- Start: Paris (largest market)
- Expand: Lyon, Marseille (Month 27)
- Support: French-speaking support team (hire 2-3 people)
- Localization: Full French translation, copropriété compliance
- Partnerships: FNAIM, UNIS (syndic associations)

**Italy (Months 28-33):**
- Start: Rome, Milan (largest markets)
- Expand: Turin, Florence, Naples (Month 30)
- Support: Italian-speaking support team (hire 2-3 people)
- Localization: Full Italian translation, condominium compliance
- Partnerships: ANACI (professional administrator association - critical)

### Feature Rollout Strategy

**MVP (Month 0-3):**
- Property & unit management
- Owner database
- Basic invoicing (manual)
- Document storage
- User management

**Enhanced (Month 3-6):**
- Automated invoicing
- Payment tracking
- Email notifications
- Assembly management
- Basic reports

**Advanced (Month 6-12):**
- DATEV integration
- Bank integrations (SEPA)
- Advanced reporting
- Mobile apps (iOS, Android)
- API access

**AI/ML (Month 12+):**
- OCR for invoices
- Predictive analytics
- Smart automation
- Recommendations

**Feature Flags:**
```typescript
// Gradual rollout of risky features
const featureFlags = {
  'ocr-invoice-extraction': {
    enabled: true,
    rollout: 10  // 10% of customers
  },
  'ai-recommendations': {
    enabled: true,
    rollout: 100,  // 100% (fully rolled out)
    tenants: ['tenant-uuid-1', 'tenant-uuid-2']  // Specific beta customers
  }
};

// Usage
if (isFeatureEnabled('ocr-invoice-extraction', tenantId)) {
  // Use OCR
} else {
  // Use manual entry
}
```

---

## Cost Optimization

### Cost Estimate (Month 12)

**Assumptions:** 1,000 customers, 500 units/customer, 2 users/customer

| Service | Configuration | Monthly Cost |
|---------|--------------|--------------|
| **Compute (ECS Fargate)** | 4 tasks × 1vCPU/2GB (avg) | $180 |
| **ECR (Container Registry)** | Image storage | $10 |
| **Database (RDS)** | db.r6g.xlarge, Multi-AZ | $800 |
| **Cache (Redis)** | cache.r6g.large | $200 |
| **Storage (S3)** | 5TB documents | $115 |
| **CDN (CloudFront)** | 10TB transfer | $850 |
| **Load Balancer** | Application LB | $30 |
| **Monitoring** | CloudWatch (no Datadog yet) | $150 |
| **Backups** | Snapshots, cross-region | $100 |
| **Other** | Secrets Manager, KMS, etc. | $80 |
| **Third-Party SaaS** | Stripe, SendGrid, Lokalise, etc. | $500 |
| **TOTAL** | | **$3,015/month** |

**Per-Customer Cost:** $3.02/month

**Revenue (assuming €149 ARPU):** €149,000/month = ~$160,000/month

**Gross Margin:** ~98% (typical SaaS)

**Cost Savings vs Kubernetes:**
- **ECS Fargate vs EKS**: $180/month vs $673/month ($73 control plane + $600 nodes) = **$493/month savings (73% reduction)**
- **Simpler Monitoring**: CloudWatch only initially (add Datadog Month 12+) = $150/month savings
- **No Kubernetes Tools**: No ArgoCD, Helm, etc. needed

**Total Monthly Savings:** **~$580/month** (~16% total cost reduction)

### Cost Optimization Strategies

**1. Reserved Instances / Savings Plans:**
- Commit to 1-year reserved instances (save 30-40%)
- Month 12: Convert to reserved instances (predictable usage)
- Estimated savings: $500/month

**2. Spot Instances:**
- Use spot instances for non-critical workloads (dev/staging, batch jobs)
- Save up to 90% on compute costs
- **Not for production** (can be terminated with 2-minute notice)

**3. Auto-Scaling:**
- Scale down during off-hours (nights, weekends)
- European customers = predictable usage patterns
- Estimated savings: 20% on compute

**4. S3 Lifecycle Policies:**
- Archive old documents to Glacier (90% cheaper)
- Example: Documents >1 year old → Glacier
- Estimated savings: $500/month at scale

**5. Database Optimization:**
- Efficient queries = smaller instances
- Vertical scaling only when necessary
- Read replicas only when bottleneck identified

**6. CDN Optimization:**
- Aggressive caching (reduce origin requests)
- Compress assets (reduce transfer)
- Use CloudFront regional pricing (cheaper in EU)

**Total Estimated Savings:** ~$1,000/month (25% reduction) by Month 24

---

## Risk Mitigation

### Technical Risks

**Risk 1: Database Bottleneck**
- **Probability:** Medium (multi-tenant shared DB)
- **Impact:** High (affects all customers)
- **Mitigation:**
  - Connection pooling (PgBouncer)
  - Query optimization (indexes, EXPLAIN ANALYZE)
  - Read replicas for reporting
  - Tenant-specific query timeouts
  - Sharding plan ready (if needed)

**Risk 2: Data Breach**
- **Probability:** Low (with proper security)
- **Impact:** Critical (regulatory fines, reputation damage)
- **Mitigation:**
  - Multi-layer security (encryption, RLS, WAF)
  - Regular pentests
  - Bug bounty program (Month 12+)
  - Cyber insurance
  - Incident response plan

**Risk 3: Integration Failures (DATEV, Banks)**
- **Probability:** Medium (external dependencies)
- **Impact:** High (blocks customer workflows)
- **Mitigation:**
  - Retry logic with exponential backoff
  - Manual fallback (export/import CSV)
  - Status page for integration health
  - SLAs with integration partners
  - Alternative integration paths (e.g., multiple bank APIs)

**Risk 4: Multi-Country Complexity**
- **Probability:** High (6 countries, different regulations)
- **Impact:** Medium (delays, quality issues)
- **Mitigation:**
  - Country-specific product managers
  - Local legal advisors
  - Beta testing with local customers
  - Phased rollout (Germany → Austria → France/Italy)
  - Shared core platform (80% reuse)

**Risk 5: Scaling Too Fast**
- **Probability:** Medium (if viral growth)
- **Impact:** Medium (performance degradation)
- **Mitigation:**
  - Auto-scaling infrastructure
  - Load testing before launch
  - Circuit breakers (graceful degradation)
  - Waitlist for new customers (if overwhelmed)
  - Hire ahead of growth curve

**Risk 6: Technical Debt Accumulation**
- **Probability:** High (fast growth = shortcuts)
- **Impact:** Medium (slows future development)
- **Mitigation:**
  - 20% time for refactoring/tech debt
  - Code reviews (quality gates)
  - Automated testing (prevent regressions)
  - Quarterly architecture reviews
  - Modular architecture (easier to refactor)

### Operational Risks

**Risk 7: Key Person Dependency**
- **Probability:** Medium (small team initially)
- **Impact:** High (bus factor = 1)
- **Mitigation:**
  - Documentation (architecture decisions, runbooks)
  - Knowledge sharing (pair programming, code reviews)
  - Redundancy in critical roles (2+ engineers know each system)
  - On-call rotation (spread knowledge)

**Risk 8: Vendor Lock-In (AWS)**
- **Probability:** Low (AWS is stable)
- **Impact:** Medium (hard to migrate)
- **Mitigation:**
  - Use portable technologies (Kubernetes, PostgreSQL, Redis)
  - Avoid AWS-specific services where possible
  - Multi-region strategy (reduce single-region dependency)
  - Terraform for reproducible infrastructure

---

## Conclusion

### Summary of Key Decisions

1. **Modular Monolith → Microservices:** Start simple, evolve as needed
2. **AWS eu-central-1 (Frankfurt):** EU data residency, German market proximity
3. **PostgreSQL + Redis:** Proven, scalable, ACID-compliant
4. **TypeScript Full-Stack:** Developer productivity, type safety
5. **React + React Native:** Code sharing, hiring pool, mature ecosystem
6. **Multi-Tenant Shared DB:** Cost-efficient, proven at scale
7. **GraphQL + REST:** Mobile efficiency + partner compatibility
8. **ECS Fargate:** Serverless containers, simple ops, cost-effective ($580/month savings vs EKS)
9. **Country-First Localization:** Competitive moat, not afterthought
10. **Security by Design:** GDPR, SOC 2, encryption, audit logs

### Success Metrics

**Month 6:**
- ✓ Platform live in Germany
- ✓ 50-100 customers onboarded
- ✓ 99.9% uptime achieved
- ✓ <200ms API latency (p95)

**Month 12:**
- ✓ 1,000 customers
- ✓ DATEV integration live
- ✓ Mobile apps launched
- ✓ SOC 2 audit in progress

**Month 24:**
- ✓ 5,000 customers (Germany + Austria)
- ✓ Multi-region architecture (Paris)
- ✓ Profitability achieved
- ✓ 99.95% uptime

**Month 36:**
- ✓ 10,000 customers across 4 countries
- ✓ Market leader in Germany
- ✓ 50+ integrations live
- ✓ AI/ML features deployed

### Next Steps

**Immediate (Weeks 1-4):**
1. Finalize technology choices (review this document with team)
2. Set up AWS account structure (multi-account strategy)
3. Create Terraform modules for core infrastructure
4. Set up GitHub org and repositories
5. Design database schema (first iteration)
6. Create system architecture diagrams (detailed)

**Month 1:**
1. Infrastructure provisioning (VPC, RDS, EKS, S3)
2. CI/CD pipeline setup
3. Monorepo structure (apps/backend, apps/frontend, packages/shared)
4. Core framework setup (Express, Apollo, Prisma)
5. Authentication & authorization implementation

**Month 2-3:**
1. Core modules development (property, financial, communication)
2. GraphQL schema design
3. Frontend scaffolding (React, routing, state)
4. Mobile apps scaffolding (React Native)
5. DATEV integration (critical path)

**Month 4-6:**
1. Feature completion (MVP scope)
2. Security hardening (pentesting, audits)
3. Performance optimization (load testing)
4. Beta customer onboarding (10-20 customers)
5. Iteration based on feedback

**Month 6:**
1. **LAUNCH** 🚀

---

**Document Prepared By:** Technical Architecture Team
**Review Cycle:** Quarterly (or when major architectural decisions needed)
**Living Document:** This architecture will evolve as we learn and scale
**Feedback:** Open to team input and improvement suggestions

---

**Appendix A: Glossary**

- **ACID:** Atomicity, Consistency, Isolation, Durability (database properties)
- **ARPU:** Average Revenue Per User
- **CDN:** Content Delivery Network
- **CI/CD:** Continuous Integration / Continuous Deployment
- **GDPR:** General Data Protection Regulation (EU privacy law)
- **HPA:** Horizontal Pod Autoscaler (Kubernetes)
- **I18n:** Internationalization
- **JWT:** JSON Web Token
- **KMS:** Key Management Service (AWS)
- **L10n:** Localization
- **MFA:** Multi-Factor Authentication
- **ORM:** Object-Relational Mapping
- **RBAC:** Role-Based Access Control
- **RLS:** Row-Level Security (PostgreSQL)
- **RTO:** Recovery Time Objective
- **RPO:** Recovery Point Objective
- **SEPA:** Single Euro Payments Area
- **SOC 2:** Service Organization Control 2 (security audit standard)
- **SRE:** Site Reliability Engineering
- **TTL:** Time To Live (cache duration)
- **VPC:** Virtual Private Cloud (AWS)
- **WAF:** Web Application Firewall

---

**Appendix B: Reference Architecture Links**

- **Shopify Modular Monolith:** https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity
- **PostgreSQL Multi-Tenancy:** https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/
- **SaaS Tenant Isolation:** https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/tenant-isolation.html
- **12-Factor App:** https://12factor.net/ (methodology for building SaaS)
- **React Query Best Practices:** https://tkdodo.eu/blog/practical-react-query
- **Kubernetes Best Practices:** https://kubernetes.io/docs/concepts/configuration/overview/

---

**End of Document**
