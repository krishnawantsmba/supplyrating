# VenIQ – Enterprise Supplier Intelligence & Risk Management Platform

A complete, production-ready full-stack B2B SaaS application for enterprise supplier evaluation and risk management.

---

## Quick Start

### Prerequisites
- Node.js 18+
- PostgreSQL 14+

### 1. Database Setup

```sql
CREATE DATABASE veniq_db;
CREATE USER veniq_user WITH ENCRYPTED PASSWORD 'veniq_password';
GRANT ALL PRIVILEGES ON DATABASE veniq_db TO veniq_user;
GRANT CREATE ON SCHEMA public TO veniq_user;
```

### 2. Backend Configuration

Copy `backend/env.example` to `backend/.env` and update values.

```bash
cd backend
npm install
npm run migrate      # Creates all tables
npm run seed         # Loads 20 demo suppliers + 4 demo users
npm run dev          # Starts API on port 3001
```

### 3. Frontend

```bash
cd frontend
npm install
npm run dev          # Starts on port 5173
```

Open `http://localhost:5173`

---

## Demo Credentials (after seeding)

| Role | Email | Password |
|---|---|---|
| Organisation Admin | admin@demo.veniq.com | Demo@123 |
| Procurement Manager | manager@demo.veniq.com | Demo@123 |
| SCM Employee | alice@demo.veniq.com | Demo@123 |

Or click **"Try Demo"** on the login page.

---

## Architecture

```
veniq/
├── backend/                    # Node.js + Express + TypeScript
│   ├── src/
│   │   ├── config/            # Environment configuration
│   │   ├── db/
│   │   │   ├── pool.ts        # PostgreSQL connection pool
│   │   │   ├── migrate.ts     # Database migrations (18 tables)
│   │   │   └── seed.ts        # Demo data generator
│   │   ├── middleware/
│   │   │   ├── auth.ts        # JWT authentication + RBAC
│   │   │   └── helpers.ts     # Audit logging + error handling
│   │   ├── routes/            # REST API endpoints
│   │   │   ├── auth.ts        # Registration, login, demo access
│   │   │   ├── suppliers.ts   # Supplier CRUD
│   │   │   ├── evaluations.ts # VPS calculation engine API
│   │   │   ├── purchaseOrders.ts
│   │   │   ├── qualityInspections.ts
│   │   │   ├── assessments.ts
│   │   │   ├── financialRecords.ts
│   │   │   ├── environmental.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── riskAlerts.ts
│   │   │   ├── comparison.ts
│   │   │   └── organisation.ts
│   │   ├── services/
│   │   │   └── scoringEngine.ts  # ← Core six-dimensional VPS engine
│   │   └── __tests__/
│   │       └── scoringEngine.test.ts  # 43 unit tests
│   └── package.json
│
└── frontend/                   # React + TypeScript + Tailwind CSS
    ├── src/
    │   ├── components/
    │   │   ├── Layout.tsx     # Persistent sidebar + topbar
    │   │   └── ui/            # Reusable UI components
    │   ├── pages/             # All 10 application modules
    │   ├── services/api.ts    # Axios API layer
    │   ├── store/authStore.ts # Zustand auth state
    │   └── types/index.ts     # Shared TypeScript interfaces
    └── package.json
```

---

## Database Schema (18 tables)

1. **organisations** – Multi-tenant root entity
2. **licences** – Subscription tiers and expiry
3. **roles** – org_admin, procurement_manager, scm_employee
4. **users** – Organisation-scoped user accounts
5. **org_settings** – Configurable scoring parameters
6. **suppliers** – Supplier directory
7. **products** – Products per supplier
8. **purchase_orders** – Procurement records with Incoterms
9. **quality_inspections** – Defect records with severity weights
10. **employee_assessments** – Responsiveness ratings (1–5 scale)
11. **financial_records** – Balance sheet and P&L data
12. **env_certifications** – ISO 14001, EMAS etc.
13. **compliance_incidents** – Environmental violations
14. **supplier_evaluations** – Immutable VPS snapshots
15. **risk_alerts** – Auto-generated risk flags with actions
16. **corrective_actions** – Action tracking
17. **audit_logs** – Full audit trail
18. **uploaded_documents** – Document metadata

---

## Scoring Formula

```
VPS = 0.25 × Reliability
    + 0.25 × Quality
    + 0.15 × Responsiveness
    + 0.10 × Location
    + 0.15 × Financial Condition
    + 0.10 × Environmental Compliance
```

| Score | Classification |
|---|---|
| 85–100 | Preferred Supplier |
| 70–84 | Approved Supplier |
| 50–69 | Conditional Supplier |
| < 50 | High-Risk Supplier |

If any dimension is missing, weights are renormalised to the sum of available dimensions and the score is marked **Provisional**. Suppliers cannot be classified as Preferred or Approved on a provisional score.

---

## Tests

```bash
cd backend
npm test
```

**43 unit tests** covering all 6 scoring dimensions, VPS calculation, classification, weight renormalisation, edge cases (zero units, missing data, zero liabilities, negative equity, expired certs) and risk alert generation.

---

## Demo Data

After running `npm run seed`, the database contains:

- **Organisation:** Apex Manufacturing Ltd. (Birmingham, UK)
- **20 demo suppliers** across diverse categories and countries
- **Varied performance profiles:**
  - Excellent delivery + quality (Precision Steel, Midlands Rubber)
  - Poor delivery performance (Guangzhou Metal, Chilean Copper)
  - Poor quality (Karachi Steel, Mumbai Auto Parts)
  - Financial distress (negative equity, operating losses)
  - Environmental violations (critical incidents)
  - No financial data (Scottish Alloys – intentional gap)
  - Excellent pricing but quality issues
  - Geographic concentration risk (>5000 km suppliers)
- **Purchase orders:** 6–12 per supplier with realistic delay patterns
- **Quality inspections:** 4–10 per supplier
- **Employee assessments:** 3 per supplier from different assessors
- **Financial records:** For 19/20 suppliers (1 intentional gap)
- **Environmental data:** Certifications and incidents

All demo records are tagged `is_demo = true`.

---

## Security

- JWT authentication with configurable expiry
- bcrypt password hashing (cost factor 12)
- Tenant isolation: all queries include `organisation_id`
- Database-level: unique constraints prevent cross-tenant data leaks
- Role-based access control enforced at API middleware level
- Financial records restricted to authorised roles
- Rate limiting on authentication endpoints
- Helmet.js security headers
- Audit logging for all significant actions

---

## Limitations

The following require additional infrastructure not available in this environment:

1. **Email notifications** – SMTP not configured. Use a service like Resend or SendGrid.
2. **File uploads** – Disk-based storage. For production, replace with S3/Blob storage.
3. **Real payment processing** – Licence management is demo-only. Integrate Stripe for production.
4. **PDF report generation** – CSV export is implemented. PDF requires additional library (e.g., jsPDF or Puppeteer).
5. **Geographic routing** – Location scoring uses straight-line Haversine distance. For road routing, integrate a maps API.
6. **WebSocket notifications** – Risk alerts are polled. Real-time requires Socket.io or SSE.
