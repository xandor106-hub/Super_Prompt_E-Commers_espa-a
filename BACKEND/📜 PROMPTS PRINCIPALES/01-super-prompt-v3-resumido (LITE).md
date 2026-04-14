# Super Prompt V3.0 — Backend Seguro + GDPR Completo (Europa)

I need to build a fully compliant, secure e-commerce backend using Supabase for a European audience. This is for learning and potential production use. Please provide complete SQL, RLS policies, Edge Functions, Stripe integration, and compliance code.

## 1. Project Context
- Business location: Europe (EU/EEA)
- Customers: European residents only initially
- Data stored: Customer names, email addresses, shipping addresses, order history, payment references (NOT full PAN)
- Estimated users: Up to 10,000 (scalable design)
- Compliance required: GDPR (full), ePrivacy Directive, PSD2 (for payments)
- Payment processor: Stripe (with Radar fraud protection, 3D Secure)
- Hosting: Supabase EU region (Frankfurt or Dublin)
- DPO status: [TO BE DETERMINED based on user response]

## 2. Database Schema (GDPR-Compliant by Design)

**Table: profiles**
- id (UUID, references auth.users, primary key)
- email (TEXT, unique, not null)
- full_name (TEXT)
- shipping_address (JSONB, encrypted at rest for sensitive fields)
- marketing_consent (BOOLEAN, default false)
- marketing_consent_given_at (TIMESTAMPTZ)
- marketing_consent_ip (INET)
- account_anonymized_at (TIMESTAMPTZ, nullable)
- created_at (TIMESTAMPTZ)
- updated_at (TIMESTAMPTZ)

**Table: consent_logs**
- id (UUID, primary key)
- user_id (UUID, references profiles.id)
- consent_type (TEXT: 'marketing', 'analytics', 'cookies_functional', 'cookies_analytics')
- status (BOOLEAN)
- version (TEXT)
- given_at (TIMESTAMPTZ)
- withdrawn_at (TIMESTAMPTZ, nullable)
- ip_address (INET)
- user_agent (TEXT)

**Table: orders**
- id (UUID, primary key)
- user_id (UUID, references profiles.id)
- stripe_payment_intent_id (TEXT, unique)
- total_amount (INTEGER, cents)
- status (TEXT: 'pending', 'paid', 'shipped', 'cancelled', 'refunded')
- shipping_address_snapshot (JSONB)
- created_at (TIMESTAMPTZ)

**Table: order_items**
- id (UUID, primary key)
- order_id (UUID, references orders.id)
- product_name (TEXT)
- quantity (INTEGER)
- unit_price_cents (INTEGER)

**Table: audit_logs**
- id (BIGSERIAL, primary key)
- user_id (UUID, nullable)
- action (TEXT)
- ip_address (INET)
- user_agent (TEXT)
- metadata (JSONB)
- created_at (TIMESTAMPTZ, default now())

**Table: data_subject_requests**
- id (UUID, primary key)
- user_id (UUID, references profiles.id)
- request_type (TEXT: 'access', 'erasure', 'rectification', 'portability')
- status (TEXT: 'received', 'processing', 'completed', 'denied')
- requested_at (TIMESTAMPTZ)
- completed_at (TIMESTAMPTZ)
- response_data_url (TEXT, nullable)
- notes (TEXT)

## 3. Row-Level Security (RLS) Policies

Enable RLS on ALL tables.

**profiles:**
- SELECT: auth.uid() = id
- UPDATE: auth.uid() = id
- DELETE: NOT ALLOWED (soft delete only)

**consent_logs:**
- SELECT: auth.uid() = user_id
- INSERT: auth.uid() = user_id
- UPDATE: NOT ALLOWED
- DELETE: NOT ALLOWED

**orders + order_items:**
- SELECT: auth.uid() = user_id
- INSERT: auth.uid() = user_id
- UPDATE: service role only
- DELETE: NOT ALLOWED

**audit_logs:**
- SELECT: admin role only
- INSERT: service role only
- UPDATE/DELETE: NOT ALLOWED

**data_subject_requests:**
- SELECT: auth.uid() = user_id OR admin role
- INSERT: auth.uid() = user_id
- UPDATE: admin role only

## 4. Authentication — PSD2 / Strong Customer Authentication (SCA)

- Email + password (bcrypt, Supabase managed)
- MFA (TOTP) — RECOMMENDED
- 3D Secure for payments above €30 (Stripe handles automatically)
- Rate limiting: 5 login attempts per 15 minutes per IP
- Session timeout: 30 minutes inactivity
- JWT expiry: 15 minutes
- Refresh token expiry: 7 days

## 5. Cookie Consent (ePrivacy Directive)

Required before dropping ANY non-essential cookies.

Edge Functions needed:
- get-consent-banner-config
- record-consent

Cookie categories:
1. Strictly necessary (no consent needed)
2. Functional (consent needed)
3. Analytics (consent needed)
4. Marketing (consent needed)

## 6. GDPR Rights Implementation

Edge Functions needed:
- request-data-export (Article 15 — Right to Access)
- request-erasure (Article 17 — Right to be Forgotten)
- request-portability (Article 20 — Data Portability)
- update-profile (Article 16 — Rectification)
- gdpr-status

## 7. Data Protection Officer (DPO) — Article 37

DPO REQUIRED if:
- Public authority
- Large-scale systematic monitoring
- Large-scale processing of special data

Decision tree for e-commerce:
- < 5,000 users: No DPO needed
- 5,000 - 50,000: Recommend DPO
- 50,000 - 250,000: DPO recommended
- > 250,000: DPO required

## 8. Breach Notification (Article 33) — 72 Hours

- Internal alert system
- Predefined notification template for local DPA
- Predefined notification template for affected users
- DPA contact list

## 9. Data Localization & Transfers

- Choose Supabase EU region (Frankfurt or Dublin)
- Ensure Stripe uses EU payment gateway
- Use Standard Contractual Clauses (SCCs) for non-EU transfers

## 10. Stripe Integration (PSD2 Compliant)

Edge Functions needed:
- create-checkout-session
- stripe-webhook

Payment security: Stripe Radar (automatically enabled)

## 11. Output Required

Please provide:
1. Complete SQL schema with all CREATE TABLE statements
2. All RLS policy CREATE POLICY statements
3. Complete TypeScript code for all Edge Functions
4. Cookie banner HTML/CSS/JS
5. Privacy policy template
6. DPA template
7. Breach notification template
8. Foxguard configuration file (.foxguardrc)
9. Environment variables list
10. Example curl commands for testing