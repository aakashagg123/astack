---
name: secure-coding-practices
description: >-
  Enforce and audit secure coding practices across all software tasks. Use whenever
  the user asks to review code for security, audit APIs or modules, write secure code, assess specs
  for security gaps, or asks about authentication (OTP/MFA), session management, input validation,
  access control, cryptography, error handling, logging, data protection, DB security, file handling,
  or memory safety. Also trigger for fintech risks (CIBIL API, PAN/Aadhaar, bureau data, UPI,
  OCEN/TReDS, RBI/DPDP Act), AI/LLM security (prompt injection, RAG leakage, model output
  validation), and API security (JWT, OAuth 2.0, rate limiting, webhook validation). Trigger on
  "security review", "OWASP", "pentest", "vulnerability", "compliance audit", "RBI audit", "DPDP",
  or any backend logic, API endpoint, DB query, or LLM integration request. Proactively apply when
  user data, credentials, sessions, or bureau calls are involved — don't wait to be asked.
---

# Secure Coding Practices Skill

A structured security review and code generation skill based on OWASP-aligned secure coding principles. Covers the full stack: input handling → output encoding → auth → sessions → access control → crypto → logging → data → comms → system config → DB → file management → memory → general practices.

---

## How to use this skill

**For code review requests**: Map the code against the relevant checklist domain(s) below. Flag each violation with severity (Critical / High / Medium / Low) and a fix recommendation.

**For code generation requests**: Write code with all relevant checklist items baked in. Do not leave security as an afterthought.

**For spec/design reviews**: Identify which checklist domains apply, flag gaps, and recommend controls before implementation begins.

**Output format**: Always structure findings as:
- **Domain** (e.g., Input Validation)
- **Finding** (what's wrong or missing)
- **Severity** (Critical / High / Medium / Low)
- **Fix** (concrete, implementable recommendation)

---

## Checklist Domains

### 1. Input Validation
- Validate all input **server-side** on a trusted system — never client-side alone
- Classify all data sources as trusted or untrusted; validate untrusted sources exhaustively
- Use a **centralised validation routine** — no ad hoc checks scattered across the codebase
- Specify and enforce character sets (UTF-8); encode input to a common set before validating
- Use **allow-lists**, not deny-lists, for expected data types
- Validate: data type, range, length, format
- Reject on any validation failure — do not sanitise and silently proceed
- Validate after UTF-8 decoding if extended character sets are in use
- Validate data from HTTP redirects
- Verify protocol headers contain only ASCII characters
- Use canonicalisation to defeat obfuscation attacks
- If hazardous input must be accepted, implement compensating controls

### 2. Output Encoding
- Encode all output **server-side** on a trusted system
- Use a tested, standard encoding routine per output type (HTML, JS, SQL, LDAP, OS)
- Specify character sets (UTF-8) for all outputs
- Contextually encode all untrusted data returned to the client
- Sanitise untrusted output used in SQL, XML, LDAP, and OS commands

### 3. Authentication & Password Management
- Enforce authentication on all non-public pages/resources
- All auth controls on a trusted system; use centralised, standard auth services
- Segregate auth logic from requested resources; use redirection to a central auth control
- Auth controls must **fail securely**
- Store credentials using **cryptographically strong one-way salted hashes** (server-side); use bcrypt/argon2, never MD5/SHA1
- Complete all input before validating credentials — never reveal which field failed
- Use HTTP POST for credential transmission; only transmit over encrypted connections
- Enforce password complexity, length, age, history, and expiry policies
- Obscure password input on screen; disable "remember me" for password fields
- Enforce account lockout after N failed attempts; implement exponential backoff
- Use the same security level for reset/change as for initial authentication
- Send reset emails only to pre-registered addresses with short-lived tokens (≤15 min TTL)
- Force password change on first use of temporary credentials
- Notify users on password reset; report last login at next successful login
- Require re-authentication before critical operations (loan disbursals, limit changes, KYC updates)

**OTP / Mobile Auth (India fintech context)**
- Generate OTPs server-side using CSPRNG; minimum 6 digits
- OTP TTL: ≤5 minutes for financial transactions; ≤10 minutes for login
- Invalidate OTP immediately after first successful use — no reuse window
- Enforce OTP attempt limits (max 3–5 attempts before cooldown/lockout)
- Never log OTP values; mask in all audit trails
- Do not send OTP over email for high-value transactions — SMS or TOTP only
- Validate OTP server-side; never accept client-claimed OTP verification

**MFA**
- Mandate MFA for: admin portals, loan officer dashboards, credit decisioning systems, any RBI-regulated function
- Support TOTP (RFC 6238) as minimum; prefer hardware tokens or push auth for privileged roles
- MFA bypass flows (account recovery) must be at least as secure as primary MFA
- Audit any third-party auth libraries for malicious or vulnerable code

### 4. Session Management
- Use server/framework session controls only — no custom session schemes
- Generate session IDs server-side using cryptographically random algorithms
- Set domain and path for session cookies to the minimum necessary scope
- Logout must fully terminate the session; be available on all protected pages
- Enforce inactivity timeout (as short as business need allows)
- Prohibit persistent logins; enforce periodic session termination even when active
- Issue a new session ID after login; invalidate old session ID
- Generate new session ID on re-authentication and on HTTP→HTTPS transition
- Prevent concurrent logins with the same user ID
- Never expose session IDs in URLs, error messages, or logs
- Set `Secure` flag on cookies over TLS; set `HttpOnly` unless client-side script access is required
- Consistently use HTTPS — no HTTP→HTTPS switching mid-session
- For sensitive operations, supplement with per-session or per-request random tokens

### 5. Access Control
- Use server-side session objects for all authorisation decisions
- Single site-wide authorisation component; fail securely on error
- Deny all access if security config is unavailable
- Enforce authorisation on **every request**, including server-side script calls
- Segregate privileged logic from general application code
- Restrict access to: URLs, functions, direct object references, services, data, config, and attributes
- Server-side and presentation-layer access rules must match
- If client-side state is used, encrypt and integrity-check it server-side
- Rate-limit transactions per user/device to deter automated attacks
- Never use the `Referer` header as the sole authorisation check
- Re-validate authorisation periodically for long-lived sessions
- Audit accounts; disable unused accounts; support session termination when auth ceases
- Service/integration accounts must have least privilege

### 6. Cryptographic Practices
- All cryptographic functions protecting secrets must run on a trusted system
- Cryptographic modules must fail securely
- Use **FIPS 140-2 compliant** (or equivalent) modules
- All random values (numbers, filenames, GUIDs, strings) via the module's approved RNG
- Establish a key management policy and process

### 7. Error Handling & Logging
- Error responses must not disclose: system details, session IDs, account info, stack traces
- Use generic error messages and custom error pages
- Application handles its own errors — do not rely on server default error pages
- Free allocated memory on error conditions
- Security control errors default to deny access
- All logging on a trusted system; log both successes and failures of security events
- Log: validation failures, auth attempts/failures, access control failures, state tampering, invalid/expired session tokens, system exceptions, admin actions, TLS failures, crypto failures
- Do not log sensitive data (passwords, session IDs, unnecessary system details)
- Prevent log injection — sanitise untrusted data before logging
- Restrict log access to authorised individuals; use a central logging routine
- Validate log integrity with cryptographic hash functions

### 8. Data Protection
- Apply least privilege — users access only what their role requires
- Encrypt highly sensitive stored data (auth verification data, PII) even server-side; use AES-256 at rest
- Purge cached/temporary sensitive files as soon as no longer needed
- Never store passwords, connection strings, or secrets in plaintext on the client
- Remove backend comments and unnecessary documentation from production code
- Never include sensitive data in HTTP GET parameters
- Disable autocomplete on sensitive forms (auth, PII)
- Disable client-side caching on sensitive pages
- Implement data retention/removal policies; align with DPDP Act 2023 consent and purpose-limitation requirements

**Sensitive Financial & Identity Data (India fintech context)**
- **PAN**: Never log in plaintext; mask as `XXXXX1234X` in UI/logs; store encrypted; transmit only over TLS
- **Aadhaar**: Do not store full Aadhaar numbers — store only VID or masked last 4 digits; comply with UIDAI tokenisation mandate
- **Bank account / IFSC**: Encrypt at rest; mask in logs and UI; never echo in API responses beyond last 4 digits
- **CIBIL / Bureau scores and reports**: Treat as Sensitive Personal Data under DPDP Act; restrict access to credit team roles only; log every access with purpose code
- **Loan application data**: Classify all fields (PII, financial, behavioural); apply field-level encryption for Critical fields
- **GST / ITR data**: Obtained under specific consent; do not repurpose beyond stated loan underwriting use; delete post-decision if not required for audit
- Data localisation: All customer financial data must reside on servers within India (RBI mandate for payment data; follow for credit data by extension)
- Implement explicit consent capture (timestamp, purpose, version) before collecting any Sensitive Personal Data — required under DPDP Act 2023

### 9. Communication Security
- Encrypt all sensitive data in transit using TLS
- TLS certificates must be valid, correctly domain-matched, and non-expired
- Failed TLS connections must **not** fall back to plaintext
- Use TLS for all authenticated content and all connections to external systems involving sensitive data
- Single, correctly configured TLS implementation
- Filter sensitive parameters from `Referer` header when linking externally

### 10. System Configuration
- Keep servers, frameworks, and components on the latest patched version
- Disable directory listings; remove unnecessary functionality, test code, and default files
- Run web server/process accounts with least privilege
- Define and restrict supported HTTP methods
- Remove OS/server/framework version info from HTTP response headers
- Isolate dev environments from production; implement change control for code

### 11. Database Security
- Use **strongly typed parameterised queries** — no string concatenation for queries
- Validate input and encode output before any DB command; reject on failure
- Application connects with least-privilege DB credentials
- Never hardcode connection strings; store encrypted in a trusted config system
- Use stored procedures to abstract data access where possible
- Close DB connections as soon as possible
- Remove default DB passwords, unnecessary functionality, default schemas, and default accounts
- Use separate credentials per trust level (user, read-only, admin)

### 12. File Management
- Never pass user-supplied data directly to dynamic include functions
- Require authentication before file uploads
- Allow only business-required file types; validate by file header, not extension
- Store uploaded files outside the web root; disable execution on upload directories
- Never pass user data to dynamic redirects or file path functions
- Use allow-lists for allowed file names and types
- Never send absolute file paths to the client; use index values mapped to pre-defined paths
- Scan uploaded files for viruses and malware

### 13. Memory Management
- Validate all input/output for untrusted data before buffer use
- Check buffer sizes; handle NULL termination; protect against overflow in loops
- Truncate input strings to reasonable lengths before passing downstream
- Explicitly close resources — do not rely solely on garbage collection
- Use non-executable stacks where available; avoid known vulnerable functions
- Free allocated memory at all exit points; overwrite sensitive memory before freeing

### 14. General Coding Practices
- Use tested, managed code for common tasks; avoid new unmanaged code
- Never issue OS commands directly from the application; use task-specific APIs
- Use checksums/hashes to verify integrity of libraries, executables, and config files
- Use locking/synchronisation to prevent race conditions on shared resources
- Explicitly initialise all variables
- Raise privileges as late as possible; drop them as soon as possible
- Never pass user-supplied data to dynamic execution functions
- Audit all third-party libraries for business necessity and safe functionality
- Implement safe, encrypted update channels

### 15. API Security
- Authenticate every API endpoint — no unauthenticated endpoints for financial operations
- Use **OAuth 2.0 with short-lived access tokens** (≤15 min) + refresh token rotation; never long-lived static tokens
- Validate **JWT** on every request: signature, `exp`, `iss`, `aud`, `alg` (reject `alg: none`); use RS256 or ES256, never HS256 with a weak secret
- Enforce **rate limiting** per user, per IP, and per endpoint; use sliding window; return `429` with `Retry-After`
- Validate webhook payloads using HMAC-SHA256 signatures; reject unsigned or unverifiable payloads immediately
- Never expose internal IDs in API responses — use opaque tokens or UUIDs
- Enforce strict schema validation on all request bodies (type, format, length, required fields)
- Return only the minimum data needed — no over-fetching; strip internal fields before response
- Implement API versioning; deprecate old versions with sunset headers, not silent removal
- Log all API calls with: timestamp, caller identity, endpoint, response code, latency — but never log request bodies containing PII or credentials
- Use `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options`, and `Strict-Transport-Security` headers on all responses
- For third-party APIs (CIBIL, CERSAI, GST): implement circuit breakers, timeouts (≤10s), and fallback states — treat 5xx as untrusted input

### 16. Fintech-Specific Risks (India Stack)
- **CIBIL / Bureau API calls**
  - Never cache raw bureau reports beyond the session — treat as ephemeral sensitive data
  - Log every bureau pull with: loan application ID, purpose code, timestamp, response code; alert on error codes 9005 / 51%+ failure rates
  - Validate bureau API responses before passing to decisioning logic — do not trust malformed or partial responses
  - Implement retry limits (max 2 retries with backoff) to avoid double-billing on bureau pulls
  - Store only derived scores/fields post-decision; do not persist raw XML/JSON bureau payload in application DB

- **PAN / Aadhaar / KYC**
  - Use DigiLocker / UIDAI APIs for KYC verification — never ask users to upload Aadhaar PDF directly without redaction
  - Validate PAN format (regex + checksum) before sending to NSDL/ITD APIs
  - Store Aadhaar verification status (boolean) + VID only — not the raw number
  - Implement consent audit trail: who fetched KYC data, when, for which application ID

- **UPI / Payment flows**
  - Validate UPI VPA format and bank response codes before updating loan disbursal status
  - Never store UPI PIN, OTP, or transaction passcodes — these must not even transit your servers
  - Idempotency keys mandatory on all payment/disbursal API calls to prevent double disbursals

- **OCEN / TReDS / SCF instruments**
  - Validate claim amounts against invoice data before triggering funding flows
  - Enforce maker-checker on any instrument creation above configurable thresholds
  - Log all state transitions in financing lifecycle with actor identity and timestamp

- **General RBI / Regulatory**
  - Implement audit logs that meet RBI IS Audit requirements: tamper-evident, retained ≥3 years, role-restricted access
  - Fair Practices Code: do not store or use data for purposes beyond stated consent (DPDP Act 2023 §6)
  - Report security incidents to RBI CSITE/CERT-In within mandated timelines

### 17. AI / LLM Security
- **Prompt Injection**
  - Never pass raw user input directly into LLM prompts — sanitise and structure all user-provided content
  - Use role-separated prompt architecture: system prompt (trusted) vs. user content (untrusted) — never merge
  - Validate LLM outputs before using them in downstream logic (parsing, DB writes, API calls)
  - Implement output length limits and format validation on all LLM responses

- **RAG Pipeline Data Leakage**
  - Enforce document-level access control before embedding — never index documents the querying user cannot access
  - Sanitise retrieved chunks before injecting into prompts — strip metadata fields containing internal IDs, confidential scores, or PII
  - Implement retrieval audit logs: which documents were retrieved for which query, by which user
  - Do not store raw LLM conversation history containing PII or financial data in vector stores

- **Model Output Validation**
  - Never use LLM-generated output directly in: SQL queries, OS commands, file paths, or API calls — always validate/parameterise
  - For credit decisioning AI: human-in-the-loop mandatory on any decision above configurable exposure thresholds
  - Flag and reject LLM outputs that contain structured data (JSON, SQL) before schema validation
  - Log model inputs and outputs for audit where LLM is used in regulated workflows (credit, KYC, fraud)

- **Model & Infrastructure Security**
  - API keys for LLM providers (Gemini, OpenAI) must be stored in secrets manager — never in code or env files committed to VCS
  - Implement per-user token budgets to prevent resource exhaustion attacks
  - Monitor for anomalous prompt patterns (unusually long inputs, jailbreak signatures) and rate-limit aggressively
  - Do not fine-tune models on customer PII without explicit anonymisation and DPDP Act 2023 compliance

---

## Severity Classification Guide

| Severity | Criteria |
|---|---|
| **Critical** | Direct path to RCE, auth bypass, credential theft, SQLi, or data exfiltration |
| **High** | Significant exposure: session hijack, IDOR, missing access control, plaintext secrets |
| **Medium** | Indirect risk: info disclosure, weak crypto, missing rate-limit, poor error handling |
| **Low** | Defence-in-depth gaps: missing HttpOnly flag, verbose headers, missing log entries |

---

## Quick Reference: Most Common High-Impact Violations

1. String-concatenated SQL queries → SQLi (Critical)
2. Client-side-only input validation → bypass (Critical)
3. Hardcoded credentials / connection strings (Critical)
4. Session ID in URL (High)
5. Missing re-authentication before critical ops (High)
6. Plaintext password storage or transmission (Critical)
7. Generic `catch` blocks swallowing errors silently (Medium)
8. Unrestricted file upload by extension only (High)
9. Missing rate limiting on auth endpoints (High)
10. Verbose error messages exposing stack traces or system info (Medium)
11. Raw user input injected into LLM prompt without sanitisation (Critical)
12. JWT `alg: none` accepted or HS256 with weak secret (Critical)
13. Full Aadhaar number stored in DB (Critical — UIDAI violation)
14. CIBIL bureau response cached beyond session or raw payload persisted (High)
15. OTP reuse window not invalidated after first use (High)
16. LLM API key committed to VCS or stored in env file (Critical)
17. No webhook HMAC signature validation (High)
18. RAG pipeline retrieving documents user has no access to (Critical)