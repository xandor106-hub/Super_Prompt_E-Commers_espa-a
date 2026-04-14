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
- marketing_consent (BOOLEAN, default false)  ← GDPR Article 7
- marketing_consent_given_at (TIMESTAMPTZ)
- marketing_consent_ip (INET)
- account_anonymized_at (TIMESTAMPTZ, nullable)  ← For right to erasure
- created_at (TIMESTAMPTZ)
- updated_at (TIMESTAMPTZ)

**Table: consent_logs** (GDPR Article 7 — prove consent was given)
- id (UUID, primary key)
- user_id (UUID, references profiles.id)
- consent_type (TEXT: 'marketing', 'analytics', 'cookies_functional', 'cookies_analytics')
- status (BOOLEAN)
- version (TEXT)  ← Which version of your consent text they agreed to
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
- shipping_address_snapshot (JSONB)  ← Snapshot at time of order (GDPR: order history must persist even if profile anonymized)
- created_at (TIMESTAMPTZ)

**Table: order_items**
- id (UUID, primary key)
- order_id (UUID, references orders.id)
- product_name (TEXT)
- quantity (INTEGER)
- unit_price_cents (INTEGER)

**Table: audit_logs** (immutable — GDPR Article 5: accountability)
- id (BIGSERIAL, primary key)
- user_id (UUID, nullable)  ← Nullable if user deleted
- action (TEXT)
- ip_address (INET)
- user_agent (TEXT)
- metadata (JSONB)
- created_at (TIMESTAMPTZ, default now())

**Table: data_subject_requests** (GDPR Articles 15-22 — track requests)
- id (UUID, primary key)
- user_id (UUID, references profiles.id)
- request_type (TEXT: 'access', 'erasure', 'rectification', 'portability')
- status (TEXT: 'received', 'processing', 'completed', 'denied')
- requested_at (TIMESTAMPTZ)
- completed_at (TIMESTAMPTZ)
- response_data_url (TEXT, nullable)  ← Temporary download link
- notes (TEXT)

## 3. Row-Level Security (RLS) Policies — GDPR Compliant

**Enable RLS on ALL tables**

**profiles:**
- SELECT: auth.uid() = id (users see only themselves)
- UPDATE: auth.uid() = id
- DELETE: NOT ALLOWED (soft delete only via anonymization)

**consent_logs:**
- SELECT: auth.uid() = user_id (users see their own consents)
- INSERT: auth.uid() = user_id (system inserts on their behalf)
- UPDATE: NOT ALLOWED (immutable record of consent)
- DELETE: NOT ALLOWED (must keep proof of consent)

**orders + order_items:**
- SELECT: auth.uid() = user_id
- INSERT: auth.uid() = user_id (only when creating order)
- UPDATE: service role only (status updates)
- DELETE: NOT ALLOWED (legal retention for tax purposes)

**audit_logs:**
- SELECT: admin role only (create is_admin check using custom claims)
- INSERT: service role only
- UPDATE/DELETE: NOT ALLOWED (immutable by design)

**data_subject_requests:**
- SELECT: auth.uid() = user_id OR admin role
- INSERT: auth.uid() = user_id
- UPDATE: admin role only

## 4. Authentication — PSD2 / Strong Customer Authentication (SCA)

- Email + password (bcrypt, Supabase managed)
- MFA (TOTP) — RECOMMENDED but not forced for all users
- 3D Secure for payments above €30 (Stripe handles automatically)
- Rate limiting: 5 login attempts per 15 minutes per IP
- Session timeout: 30 minutes inactivity
- JWT expiry: 15 minutes
- Refresh token expiry: 7 days

## 5. Cookie Consent (ePrivacy Directive)

**Required before dropping ANY non-essential cookies:**

Edge Function: get-consent-banner-config
- Returns current consent version and categories
- No cookies set until user interacts

Edge Function: record-consent
- Stores user consent in consent_logs table
- Records timestamp, version, IP, user agent
- Returns 204 No Content

**Cookie categories:**
1. Strictly necessary (no consent needed — checkout, auth, security)
2. Functional (consent needed — preferences, language)
3. Analytics (consent needed — PostHog, Plausible, etc.)
4. Marketing (consent needed — email lists, retargeting)

## 6. GDPR Rights Implementation

**Edge Function: request-data-export (Article 15 — Right to Access)**
- Creates entry in data_subject_requests
- Compiles all user data from: profiles, orders, order_items, consent_logs
- Generates JSON file to Supabase Storage
- Returns download URL (expires in 15 minutes)
- Logs request to audit_logs
- Completion target: 30 days (log deadline in metadata)

**Edge Function: request-erasure (Article 17 — Right to be Forgotten)**
- Creates entry in data_subject_requests
- Anonymizes profile: email → deleted_[timestamp]@anon.local, full_name → NULL, shipping_address → NULL
- Sets account_anonymized_at = now()
- Does NOT delete orders (legal retention for tax — 7 years in most EU countries)
- Does NOT delete consent_logs (legal proof)
- Triggers auth user deletion or deactivation
- Completion target: 30 days

**Edge Function: request-portability (Article 20 — Data Portability)**
- Same as export but returns machine-readable JSON (CSV optional)
- Includes data provided by user + observed data

**Edge Function: update-profile (Article 16 — Rectification)**
- Standard update endpoint with validation
- Logs change to audit_logs

**Edge Function: gdpr-status**
- Allows user to see status of their pending requests
- Returns all data_subject_requests for that user

## 7. Data Protection Impact Assessment (DPIA) — Article 35

**Trigger conditions for a DPIA:**
- ✅ Systematic monitoring of public areas? NO
- ✅ Large-scale processing of special data? NO (unless health/financial)
- ✅ Processing data to determine access to services? NO
- ⚠️ Using new technologies? YES (AI assistance for coding)

**DPIA required if:**
- You process biometric, health, or criminal record data
- You monitor publicly accessible areas
- You use automated decision-making with legal effects

**For a standard e-commerce site with 10,000 users:**
- DPIA is RECOMMENDED but not strictly mandatory
- Document your data flows and mitigations regardless

## 8. Data Protection Officer (DPO) — Article 37

**DPO is REQUIRED if:**
- ❌ Public authority (you are not)
- ✅ Large-scale systematic monitoring? (10,000+ users = borderline, argueable)
- ✅ Large-scale processing of special data? (Payment data = special? No, not Article 9)
- ⚠️ Core activities involve large-scale monitoring? (E-commerce analytics? Possibly)

**Decision tree for your e-commerce:**

| Users | Payment data only | Payment + analytics | Payment + analytics + marketing |
|-------|------------------|---------------------|--------------------------------|
| < 5,000 | No DPO needed | No DPO needed | No DPO needed (recommend privacy advisor) |
| 5,000 - 50,000 | No DPO needed | Recommend DPO | Recommend DPO |
| 50,000 - 250,000 | Recommend DPO | DPO recommended | DPO required (arguably) |
| > 250,000 | DPO required | DPO required | DPO required |

**If DPO not required, you still need:**
- A person responsible for GDPR inquiries (can be you)
- Documented process for handling requests
- Privacy policy with contact point

**Edge Function: contact-dpo**
- Creates secure ticket for privacy inquiries
- Only accessible to admin users
- Logs to audit_logs

## 9. Data Processing Agreements (DPA)

**Must have DPA with:**
- Supabase (they provide — select EU hosting)
- Stripe (they provide — EU data processing addendum)
- Any analytics provider (Plausible, Simple Analytics — both GDPR-compliant)
- Any email marketing tool (if used with consent)

**DPA checklist:**
- [ ] Signed by both parties
- [ ] Specifies data categories processed
- [ ] Specifies processing purposes
- [ ] Includes Standard Contractual Clauses (SCCs) for non-EU transfers
- [ ] Includes subprocessor list
- [ ] Includes breach notification obligations
- [ ] Includes audit rights

## 10. Breach Notification (Article 33) — 72 Hours

**Implementation requirements:**
- Internal alert system: If audit_logs shows >100 failed logins in 5 minutes → alert
- Predefined notification template for local DPA
- Predefined notification template for affected users (if high risk)
- DPA contact list (for your country: AEPD for Spain, ICO for UK, CNIL for France, etc.)

**Edge Function: simulate-breach-notification (admin only)**
- Triggers test notification workflow
- Does NOT send real notifications
- Logs to audit_logs for compliance proof

**Actual breach response:**
1. Detect (via monitoring or Foxguard secrets scan)
2. Contain (rotate keys, isolate systems)
3. Assess (what data? how many users?)
4. Notify DPA within 72 hours
5. Notify users if high risk
6. Document everything

## 11. Data Localization & Transfers

**Requirements:**
- Choose Supabase EU region (Frankfurt or Dublin)
- Ensure Stripe uses EU payment gateway (automatic for EU customers)
- Do NOT use US-hosted analytics without SCCs
- Do NOT backup to US regions without appropriate safeguards

**Transfer mechanisms for non-EU:**
- Standard Contractual Clauses (SCCs) — approved by European Commission
- Binding Corporate Rules (BCRs) — only for multinationals
- Adequacy decision — countries approved by EU (very few)

## 12. Stripe Integration (PSD2 Compliant)

**Edge Function: create-checkout-session**
- Creates Stripe Checkout Session
- Enables 3D Secure automatically for transactions > €30
- Stores stripe_payment_intent_id in orders
- Returns session URL

**Edge Function: stripe-webhook**
- Verifies webhook signature (critical for security)
- On payment_intent.succeeded: updates order status to 'paid'
- On payment_intent.payment_failed: logs failure
- On chargeback: updates order status to 'disputed'
- Logs all webhook events to audit_logs

**Payment security (Stripe Radar):**
- Automatically enabled
- Evaluates >1,000 transaction characteristics
- Blocks or requests 3D Secure for suspicious transactions

## 13. Security Scanning (Foxguard + Manual)

**Pre-deployment checks:**
```bash
npx foxguard . --severity high
npx foxguard secrets .
npm audit