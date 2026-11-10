# stackademic — Secure-Coding Rules

This file owns the implementation rules that keep the code secure. WHAT the system does is in `REQUIREMENTS.md`, how it looks is in `DESIGN.md`, and how it is built is in `ARCHITECTURE.md`. This file contains no threat model, no risk scores, and no application code.

IDs (`SC-`, `OD-`) are permanent. They are never renumbered or reused.

The owner answered the first interview question directly (OD-001), then asked for the remaining open decisions to be answered on their behalf. Every decision that comes from one of those delegated answers ends with *(assumed)*. An assumed decision is a proposed one the owner has accepted in bulk, not something the owner stated; the owner can change any of them.

Status words: **ACTIVE** (binding now), **BLOCKED** (cannot be implemented or verified until the linked `OD-###` is settled), `UNKNOWN` (a fact nobody has supplied), `TO BE DECIDED` (an open choice).

In every rule, **Evidence** separates three things: the project document that makes the rule apply, the public source that supports the control, and anything stricter that is a **project decision** and is not claimed from a source.

## Secure-Coding Inputs

### Documents read

- `REQUIREMENTS.md` — FR-1.1 to FR-8.2, NFR-1.1 to NFR-2.4. No open questions.
- `DESIGN.md` — including "Security-relevant interactions". No open questions.
- `ARCHITECTURE.md` — stack, components, TB-1 to TB-8, DR-1 to DR-11, AQ-1 to AQ-13. No open questions.
- Security notes supplied with this step: "private passkey-only site, nothing but a basic marketing page for the public, all the cool stuff is behind passkey auth"; users: "admin, players, tutors".
- Existing `SECURITY.md`: none. No earlier `SC-###` IDs exist to preserve.

### Confirmed components and versions (from ARCHITECTURE.md)

- Java 25 target (JDK 25 to 27); Spring Boot 4.1.x (4.1.2); Spring Framework 7.0.x (7.0.10); embedded Tomcat 11.0.x.
- Spring Security 7.1.x with `spring-security-webauthn` (webauthn4j-core 0.31.x).
- Spring Data JPA / Hibernate ORM 7.4.x; pgJDBC 42.7.x; Flyway 12.4.x; Hibernate Validator 9.1.x; Angus Mail 2.0.x; SLF4J + Logback.
- PostgreSQL 18.x in Docker; Mailpit 1.33.x in Docker; Docker 29.x / Compose v5.x (development machine only).
- React + React DOM 19.3.x; React Router 8.x; TypeScript 7.0.x; Vite 8.x (build and dev only); Node.js 24.x (build and test only, not present at run time).
- Deployment: one application process and one database on the owner's machine at `localhost` (AQ-1). TB-1 says this boundary is treated as untrusted exactly as if it were the internet, so the rules below do not relax because of `localhost`.

### Applicability decisions

Each area the step asked about, with the decision:

- **Request and resource limits** — applicable. SC-038 to SC-041.
- **Event-loop blocking** — not applicable as written: there is no Node.js server and no event loop at run time (Node is build-time only). The servlet equivalent is applicable: request threads, CPU-bound win-probability work, email sending, and long-lived event streams. SC-040, SC-041.
- **Input validation** — applicable. SC-027 to SC-031.
- **Output encoding** — applicable. SC-032 to SC-036.
- **Authentication** — applicable, passkeys only. SC-006 to SC-020, SC-057. Password storage, password policy, and password reset guidance are not applicable: there are no passwords.
- **Authorization** — applicable. SC-001 to SC-005.
- **Sessions** — applicable. SC-021 to SC-024.
- **CSRF** — applicable (cookie session, browser client). SC-025, SC-026.
- **Errors** — applicable. SC-037.
- **Logging** — applicable. SC-042 to SC-045.
- **Dependencies** — applicable. SC-051 to SC-053.
- **Runtime permissions** — the Node.js permission model is not applicable (no Node at run time). The equivalents are applicable: database role privileges, container settings, and network binding. SC-046 to SC-050.
- **Prototype pollution** — server side not applicable (Java). Client side applicable in a small way: the client's own content parser and any keyed lookups. SC-055.
- **Unsafe code execution** — applicable on both sides. SC-032 (client), SC-054 (server).
- **CORS** — no cross-origin access exists (DR-1). The control is to configure none. SC-025.
- **File upload** — not applicable: no requirement accepts files. Multipart handling is switched off (SC-038).
- **Server-side rendering, React Server Components, Server Functions** — not applicable: the client is a static bundle served by Spring Boot. The SSR sections of the React cheat sheet are not used.
- **HTML sanitization** — not applicable: no raw HTML is ever rendered (DESIGN.md, AQ-11). The sanitizer named in the React cheat sheet (DOMPurify) is an example in a guide, not an approved dependency, and is not needed.
- **Payments** — not applicable (FR-8.1, FR-8.2).
- **Guide examples that do not apply** — the Spring "Securing a Web Application" guide uses form login, an in-memory user, BCrypt, and Thymeleaf, and states Spring Boot 4.0.9. Only its `SecurityFilterChain` / `authorizeHttpRequests` pattern is used here; the rest is not part of this stack.
- **npm `--ignore-scripts`** (OWASP NPM Security) — not adopted as a rule; no confirmed fact says whether the client toolchain installs cleanly with it. Left for step 07.
- **Passkey management page** (list, rename, remove passkeys — OWASP Passkey Security) — no requirement defines one, so no rule is written. What happens to old passkeys is covered only for recovery (SC-019).
- **Signature counters** — no custom counter logic is written. Spring Security stores the counter the authenticator reports after each sign-in and the project takes the library's behaviour as it is; a counter is never a reason to lock an account (OD-013, *assumed*).

### Gaps and conflicts

- **Conflict, settled — public marketing page.** The notes say the public sees "a basic marketing page". `REQUIREMENTS.md` defines no such page and `DESIGN.md` says signed-out pages show only "Sign in" and "Create account". The owner confirmed the page exists (OD-001). SC-002 covers it. The other two documents still need it added; see Follow-ups.
- **Conflict, settled — role names.** The notes list "admin, players, tutors". The requirements define user, smart student, expert player, and administrator. Project documents win: "players" is read as user and smart student, "tutors" as expert player, "admin" as administrator. No rule uses the notes' names.
- **Conflict, settled as a recorded deviation — `Secure` cookies vs. plain HTTP on `localhost`.** OWASP Session Management calls the `Secure` attribute mandatory and recommends the `__Host-` prefix. ARCHITECTURE.md says HTTP on `localhost`. The demo stays on HTTP and the cookie is not marked `Secure`; SC-022 limits that deviation to the loopback address (OD-002, *assumed*).
- **Conflict, settled — CSRF token delivery.** Spring Security's single-page setting (`csrf.spa()`) puts the token in a script-readable cookie. OWASP CSRF Prevention says a synchronizer token should not travel in a cookie. The project uses the session-held token sent in a header, which both sources support (OD-009, *assumed*).
- **Gap, closed — WebAuthn Level 3 text.** The specification page was too large for the fetch tool; §7.1, §7.2, §13.4.3, §13.4.6, §13.4.9 and §15.1 were read from a downloaded copy. Other sections were not read and are not cited.
- **Gap, closed — Spring Security passkey behaviour not in its reference page.** Settled by reading the Spring Security `7.1.x` source (S-42); findings are under OD-013 below.
- **Gap — Spring Boot session-cookie defaults.** The properties appendix lists `server.servlet.session.cookie.*` without default values. The rules set each attribute explicitly instead of relying on defaults.
- **Gap — draft source.** The React Security Cheat Sheet is in the OWASP `cheatsheets_draft` folder. It is used as supplied; where React's own documentation covers the same point, both are cited.
- **Gap — "PostGres Security Guidance" and "Passkey Security Guidance"** were named without a URL. Used: the official PostgreSQL 18 documentation and the Docker Official Image page for PostgreSQL; the OWASP Passkey Security Cheat Sheet and W3C Web Authentication Level 3. No FIDO Alliance document was read.
- **Gap — webauthn4j.** How webauthn4j's non-strict manager treats a signature counter that does not increase was not read. No rule depends on it.

### Decisions on former open decisions

All thirteen are resolved. The IDs stay reserved so later steps can cite them.

- **OD-001 — Public marketing page.** Yes: one static public page with no user data. *Decided by: owner.* → SC-002.
- **OD-002 — Scheme, port, and development origin.** Plain HTTP at `http://localhost:8080`, bound to the loopback address. During development only, `http://localhost:5173` (the Vite dev server) is also an allowed passkey origin, switched on by a development profile. The session cookie is not `Secure` while this holds. *(assumed)* → SC-007, SC-022.
- **OD-003 — User verification.** Required for every registration and every sign-in. *(assumed)* → SC-009.
- **OD-004 — Session lifetime.** 30 minutes idle; 8 hours total. The limited session from an email link lasts 15 minutes at most. *(assumed)* → SC-023.
- **OD-005 — Fresh passkey check.** A passkey assertion no more than 5 minutes old. *(assumed)* → SC-057.
- **OD-006 — Rate limiting.** A small in-process limiter written in the Security layer, with no new package and no new store; counts are kept in memory and reset on restart. Limits: 3 emails per address per hour; 20 email-triggering requests per source address per hour; 30 ceremony starts or link redemptions per source address per minute. *(assumed)* → SC-020.
- **OD-007 — What recovery does.** Registering a passkey through a recovery link removes every earlier passkey of the account, ends all its sessions, cancels its other unused tokens, and sends a notice to the account's address. Administrators recover the same way as everyone else. *(assumed)* → SC-019.
- **OD-008 — Confirmation-link lifetime.** 24 hours. *(assumed)* → SC-015.
- **OD-009 — CSRF token delivery.** Token held in the server session, read by the client from a token route, and sent in the `X-CSRF-TOKEN` request header. `csrf.spa()` is not used. *(assumed)* → SC-026.
- **OD-010 — Size limits.** JSON body 128 KB; email address 254 characters; display name 40; passkey label 64; quiz explanation 4,000; lesson text 20,000; chat stays 2,000 (FR-4.21). *(assumed)* → SC-039.
- **OD-011 — Database roles and passwords.** Two non-superuser roles: an owner role that runs migrations and owns the tables, and a runtime role the application uses, limited to reading and writing rows. The runtime role cannot write the administrator flag. The operator sets that flag with the owner role (TB-8). The image's superuser is used only to create the two roles when the database is first created. Passwords come from files that Git ignores; none has a default. *(assumed)* → SC-047.
- **OD-012 — Logs.** Standard output only. The application writes no log files and keeps nothing itself. *(assumed)* → SC-045.
- **OD-013 — Spring Security 7.1 passkey behaviour.** Read from the `7.1.x` source (S-42): (1) user verification defaults to `PREFERRED` for both ceremonies and is changed with `setCustomizeCreationOptions` / `setCustomizeRequestOptions` on `Webauthn4JRelyingPartyOperations`; the UV flag is enforced only when the options say `REQUIRED`. Other defaults: attestation `NONE`, resident key `REQUIRED`, random challenge, 5-minute timeout. (2) After a successful sign-in the stored signature counter is overwritten with the reported value; Spring Security adds no anomaly handling of its own. (3) `WebAuthnConfigurer.disableDefaultRegistrationPage(true)` removes the generated page. (4) A missing `FACTOR_WEBAUTHN` authority is routed by default to an entry point that redirects to `/login`; `ExceptionHandlingConfigurer.defaultDeniedHandlerForMissingAuthority` replaces it. *Decided by: source reading; the choice to add no counter logic is (assumed).* → SC-009, SC-012, SC-057.

### Follow-ups for other documents

This step may change only `SECURITY.md`. These decisions add facts the other documents do not yet state; they are not open decisions.

- `REQUIREMENTS.md` and `DESIGN.md`: the public marketing page (OD-001); the notice email after recovery and the removal of earlier passkeys (OD-007).
- `ARCHITECTURE.md`: port 8080 and the development origin (OD-002); the in-process rate limiter in the Security layer (OD-006); the two database roles (OD-011); the total session time limit (OD-004).

## Sources

All retrieved 2026-11-10.

**Guides named in the step**

- **S-01 — OWASP React Security Cheat Sheet (draft).** https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets_draft/React_Security_Cheat_Sheet.md — sections "Cross-Site Scripting (XSS) Prevention", "Sensitive Data Exposure", "Authentication and Authorization", "Other Considerations". Revision: commit `995524d4` (2026-10-03). Supports: web client.
- **S-02 — Spring guide "Securing a Web Application".** https://spring.io/guides/gs/securing-web — section "Set up Spring Security". Version: states Spring Boot 4.0.9. Supports: Security layer (filter-chain pattern only).

**Spring (official reference)**

- **S-03 — Spring Security Reference 7.1.2, "Passkeys".** https://docs.spring.io/spring-security/reference/servlet/authentication/passkeys.html — "Configuration", "Register a New Credential", "Verifying an Authentication Assertion". Supports: Security layer, Accounts.
- **S-04 — Spring Security Reference 7.1.2, "Cross Site Request Forgery (CSRF)".** https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html — "Persisting the CsrfToken", "Handling the CsrfToken", "Integrating with CSRF Protection", "CSRF Considerations", "Disable CSRF Protection". Supports: Security layer, web client.
- **S-05 — Spring Security Reference 7.1.2, "Authentication Persistence and Session Management".** https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html — "Understanding Session Fixation Attack Protection", "Configuring Concurrent Session Control", "Clearing Session Cookies on Logout". Supports: Security layer.
- **S-06 — Spring Security Reference 7.1.2, "Security HTTP Response Headers".** https://docs.spring.io/spring-security/reference/features/exploits/headers.html (defaults) and https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html (configuration). Supports: Security layer.
- **S-07 — Spring Security Reference 7.1.2, "Authorize HttpServletRequests".** https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html — "Authorizing an Endpoint", "All Dispatches Are Authorized", "Favor permitAll over ignoring". Supports: Security layer.
- **S-08 — Spring Security Reference 7.1.2, "Method Security".** https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html — enabling, `@PreAuthorize` / `@PostAuthorize`, catch-all rule advice. Supports: components that own data.
- **S-09 — Spring Security Reference 7.1.2, "Multi-Factor Authentication".** https://docs.spring.io/spring-security/reference/servlet/authentication/mfa.html — "FactorGrantedAuthority", "Selectively Requiring MFA", "Specifying a Valid Duration". Supports: Security layer (fresh passkey check).
- **S-10 — Spring Security Reference 7.1.2, "Handling Logouts".** https://docs.spring.io/spring-security/reference/servlet/authentication/logout.html. Supports: Security layer.
- **S-11 — Spring Security Reference 7.1.2, "One-Time Token Login".** https://docs.spring.io/spring-security/reference/servlet/authentication/onetimetoken.html. Read to check applicability; not used (see SC-012). Supports: none.
- **S-12 — Spring Boot 4.1.2, "Common Application Properties".** https://docs.spring.io/spring-boot/appendix/application-properties/index.html — `server.*`, `spring.web.error.*`, `spring.servlet.multipart.*`, `spring.mvc.*`, `spring.jackson.*`, `spring.jpa.*`, `logging.structured.*`. Supports: API edge, Security layer.
- **S-13 — Spring Boot 4.1.2 Reference, "Servlet Web Applications".** https://docs.spring.io/spring-boot/reference/web/servlet.html — "Error Handling", "Static Content", SameSite cookies. Supports: API edge.
- **S-14 — Spring Framework 7.0.10 Reference, "Asynchronous Requests".** https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-async.html — "SSE", "Disconnects", "Configuration". Supports: API edge (event stream).
- **S-15 — Spring Data JPA 4.1.2 Reference, "JPA Query Methods".** https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html — "Using @Query", "Using Sort", LIKE escaping. Supports: persistence code.

**PostgreSQL and containers (official)**

- **S-16 — PostgreSQL 18.6 Documentation.** §20.1 "The pg_hba.conf File" https://www.postgresql.org/docs/18/auth-pg-hba-conf.html; §20.5 "Password Authentication" https://www.postgresql.org/docs/18/auth-password.html; §5.8 "Privileges" https://www.postgresql.org/docs/18/ddl-priv.html; §5.10 "Schemas" https://www.postgresql.org/docs/18/ddl-schemas.html; §21.2 "Role Attributes" https://www.postgresql.org/docs/18/role-attributes.html; §19.3 "Connections and Authentication" https://www.postgresql.org/docs/18/runtime-config-connection.html. Supports: PostgreSQL.
- **S-17 — Docker Official Image "postgres".** https://hub.docker.com/_/postgres — "Environment Variables", "Docker Secrets", "Initialization scripts". Revision: UNKNOWN. Supports: local database container.
- **S-18 — Mailpit documentation, "Runtime options".** https://mailpit.axllent.org/docs/configuration/runtime-options/ — newest option tagged v1.33.0. Supports: local mail catcher.

**Passkeys**

- **S-19 — OWASP Passkey Security Cheat Sheet.** https://cheatsheetseries.owasp.org/cheatsheets/Passkey_Security_Cheat_Sheet.html — all sections. Revision: UNKNOWN. Supports: Security layer, Accounts.
- **S-20 — W3C Web Authentication Level 3.** https://www.w3.org/TR/webauthn-3/ — §7.1, §7.2, §13.4.3, §13.4.6, §13.4.9, §15.1. Version: W3C Recommendation, 25 August 2026. Supports: Security layer.

**OWASP Cheat Sheet Series** (https://cheatsheetseries.owasp.org/cheatsheets/ — revisions UNKNOWN; the series is unversioned)

- **S-21 — Session Management.** `Session_Management_Cheat_Sheet.html`. Supports: Security layer.
- **S-22 — Cross-Site Request Forgery Prevention.** `Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html`. Supports: Security layer, web client.
- **S-23 — Authorization.** `Authorization_Cheat_Sheet.html`. Supports: Security layer, all components.
- **S-24 — Forgot Password.** `Forgot_Password_Cheat_Sheet.html`. Supports: Accounts, Email gateway.
- **S-25 — Email Validation and Verification.** `Email_Validation_and_Verification_Cheat_Sheet.html`. Supports: Accounts.
- **S-26 — Input Validation.** `Input_Validation_Cheat_Sheet.html`. Supports: API edge, components.
- **S-27 — Bean Validation.** `Bean_Validation_Cheat_Sheet.html`. Supports: API edge.
- **S-28 — Mass Assignment.** `Mass_Assignment_Cheat_Sheet.html`. Supports: API edge.
- **S-29 — Query Parameterization.** `Query_Parameterization_Cheat_Sheet.html`. Supports: persistence code.
- **S-30 — Content Security Policy.** `Content_Security_Policy_Cheat_Sheet.html`. Supports: Security layer, web client.
- **S-31 — HTTP Security Response Headers.** `HTTP_Headers_Cheat_Sheet.html`. Supports: Security layer.
- **S-32 — Error Handling.** `Error_Handling_Cheat_Sheet.html`. Supports: API edge.
- **S-33 — Logging.** `Logging_Cheat_Sheet.html`. Supports: all server components.
- **S-34 — Denial of Service.** `Denial_of_Service_Cheat_Sheet.html`. Supports: API edge, Win probability, Tutoring.
- **S-35 — Deserialization.** `Deserialization_Cheat_Sheet.html`. Supports: server code.
- **S-36 — Prototype Pollution Prevention.** `Prototype_Pollution_Prevention_Cheat_Sheet.html`. Supports: web client.
- **S-37 — Docker Security.** `Docker_Security_Cheat_Sheet.html`. Supports: local containers.
- **S-38 — NPM Security.** `NPM_Security_Cheat_Sheet.html`. Supports: client build.
- **S-39 — Vulnerable Dependency Management.** `Vulnerable_Dependency_Management_Cheat_Sheet.html`. Supports: both builds.

**Spring Security source**

- **S-42 — Spring Security source, branch `7.1.x`.** https://github.com/spring-projects/spring-security/tree/7.1.x — `config/.../configurers/WebAuthnConfigurer.java`, `config/.../configurers/ExceptionHandlingConfigurer.java`, `webauthn/.../management/Webauthn4JRelyingPartyOperations.java`. Revision: branch head on the retrieval date; commit UNKNOWN. Supports: Security layer (settles OD-013).

**Client tooling (official)**

- **S-40 — React reference, "Common components".** https://react.dev/reference/react-dom/components/common — "Dangerously setting the inner HTML". Version: describes React 19. Supports: web client.
- **S-41 — Vite 8.3.7 documentation.** "Env Variables and Modes" https://vite.dev/guide/env-and-mode; "Server Options" https://vite.dev/config/server-options; "Build Options" https://vite.dev/config/build-options. Supports: client build and dev server.

## Secure-Coding Rules

### Authorization

**SC-001 — Deny by default at the route level**
- **Status:** ACTIVE
- **Rule:** Every route MUST require a full signed-in session unless it is on a short explicit public list, and the filter chain MUST end with a rule that refuses anything unmatched. The public list is: the built client files, account creation, recovery request, email-link redemption, passkey sign-in, the CSRF token route (SC-026), and the marketing page (SC-002). Static files MUST be permitted through the filter chain, not excluded from it.
- **Enforcement:** Security layer — the single `SecurityFilterChain`.
- **Evidence:** FR-6.1, TB-1. S-07 "Authorizing an Endpoint" (deny unmatched requests) and "Favor permitAll over ignoring"; S-02 (filter-chain pattern); S-23 "Deny by Default".
- **Verification:** Pass: a test lists every mapped route and each is either on the public list or returns 401 without a session. Negative: a request to a path that no rule names returns 401 or 403, never 200.

**SC-002 — Public marketing page**
- **Status:** ACTIVE
- **Rule:** The marketing page MUST be static files served at one exact path on the public list of SC-001. It MUST NOT read the session, call any API route, show any user or account data, or load anything from another origin. It carries the same response headers as every other page (SC-035).
- **Enforcement:** Security layer (public list); web client.
- **Evidence:** OD-001 (owner). S-23 "Enforce Authorization on Static Resources"; S-07 "Favor permitAll over ignoring".
- **Verification:** Pass: a signed-out visitor gets the page with the SC-035 headers. Negative: loading the page causes no request other than its own static files; the page's content is byte-identical for a signed-out visitor and a signed-in administrator.

**SC-003 — Decide from stored state on every request**
- **Status:** ACTIVE
- **Rule:** The server MUST work out the caller's roles (administrator, expert player, tutor access) and active flag from stored state on every request. It MUST NOT accept a role, account ID, or relationship from a request body, header, query string, or the client's own state, and MUST NOT rely on a hidden or disabled control.
- **Enforcement:** Security layer for roles on routes; Accounts, Progression, and Tutoring as the owners of the stored state (DR-11).
- **Evidence:** DR-10, DR-11, TB-1, FR-7.8. S-23 "Validate Permissions on Every Request", "Perform Checks in the Right Location"; S-01 "Authentication and Authorization".
- **Verification:** Pass: after an administrator removes expert status (FR-7.6), that user's next tutor-only request is refused without signing in again. Negative: a request that adds `"role":"ADMIN"` or another user's account ID to its body gets no extra access.

**SC-004 — Check ownership and relationship for every object**
- **Status:** ACTIVE
- **Rule:** Every read, write, and event-stream subscription that names an object (quiz answer, practice hand, exam attempt, chat, live game, tutor selection) MUST be checked in the owning component against the signed-in account: "is this mine?" or "is this my current tutor or student?". The account MUST come from the session, never from the request.
- **Enforcement:** The owning component (Quiz, Simulation, Progression, Bluffing sessions, Tutoring), with method-level checks where they fit.
- **Evidence:** FR-1.14, FR-4.19, FR-4.20, TB-4, DR-4. S-23 "Ensure Lookup IDs Are Not Accessible", "Prefer ABAC and ReBAC over RBAC"; S-08 (`@PreAuthorize` / `@PostAuthorize`, catch-all rule advice).
- **Verification:** Pass: a tutor can read their current student's quiz results. Negative: user A requesting user B's quiz answer, chat, or live game by ID is refused; a tutor is refused the results of a student who has switched tutors (FR-4.19); a refusal looks the same whether or not the ID exists.

**SC-005 — No web path to the administrator flag**
- **Status:** ACTIVE
- **Rule:** No request-handling code MAY write the administrator flag. Request bodies MUST bind to purpose-built input types that contain only the fields the caller may set; stored entities MUST NOT be bound from, or serialized to, HTTP.
- **Enforcement:** API edge (input types); Accounts (no write path); operator action in the database only (TB-8, AQ-6).
- **Evidence:** FR-7.2, FR-7.8, TB-8. S-28 (bind to a dedicated object; do not rely on a block-list).
- **Verification:** Pass: a search of the server code finds no assignment of the administrator flag outside migrations and tests. Negative: sending `admin`, `isAdmin`, `active`, or `expert` fields on account creation or any update leaves the stored values unchanged.

### Passkeys

**SC-006 — Use the framework's passkey support; write no protocol code**
- **Status:** ACTIVE
- **Rule:** Passkey registration and sign-in MUST be verified only by Spring Security's WebAuthn support. Project code MUST NOT parse or verify attestation objects, assertions, client data, CBOR, or COSE keys. The browser side MUST use the platform `navigator.credentials` API with no helper library. Every ceremony MUST have automated tests for both success and failure.
- **Enforcement:** Security layer; web client.
- **Evidence:** ARCHITECTURE.md Stack (server packages; "Deliberately not added"), TB-2. S-19 "Use a Maintained Library"; S-03 "Required Dependencies".
- **Verification:** Pass: sign-up and sign-in tests succeed with a virtual authenticator. Negative: an assertion with one altered byte in the signature, challenge, or client data is rejected and creates no session.

**SC-007 — Fixed relying-party ID and exact origin list**
- **Status:** ACTIVE
- **Rule:** The relying-party ID MUST be the fixed value `localhost`. The allowed origins MUST be an exact list in server configuration: `http://localhost:8080` only, plus `http://localhost:5173` only when the development profile is active. Neither value MAY be derived from a request header, and no pattern or wildcard origin is allowed. The same values MUST be used for registration and sign-in.
- **Enforcement:** Security layer — `webAuthn` configuration (`rpId`, `allowedOrigins`).
- **Evidence:** AQ-1, TB-2, OD-002 *(assumed)*. S-03 "Configuration"; S-19 "Choose the RP ID Carefully"; S-20 §13.4.9.
- **Verification:** Pass: a ceremony from `http://localhost:8080` succeeds. Negative: the same ceremony with client data naming `http://localhost:9999`, `http://127.0.0.1:8080`, or `https://localhost:8080` is rejected; without the development profile, `http://localhost:5173` is rejected; changing the `Host` header changes nothing.

**SC-008 — Server-held, single-use challenges**
- **Status:** ACTIVE
- **Rule:** Every ceremony MUST use a fresh challenge generated by the framework on the server and held in the server-side session. A challenge MUST be accepted once, only for the ceremony type and session it was issued for, and MUST be discarded on any outcome. The default challenge handling MUST NOT be replaced with a store the client controls.
- **Enforcement:** Security layer.
- **Evidence:** TB-2. S-20 §13.4.3 (random, server-generated, at least 16 bytes, stored until the operation completes); S-19 "Generate and Bind Registration Options", "Generate and Bind Authentication Options"; S-03 (options persisted in the `HttpSession` by default).
- **Verification:** Pass: a normal ceremony completes. Negative: replaying a completed response is rejected; a response to a challenge issued in a different session is rejected; a registration response sent to the sign-in endpoint is rejected.

**SC-009 — User verification is required**
- **Status:** ACTIVE
- **Rule:** Both the registration options and the sign-in options MUST set user verification to `required`, so that the framework checks the returned UV flag on the server. The framework default (`preferred`) MUST NOT be left in place.
- **Enforcement:** Security layer — a `WebAuthnRelyingPartyOperations` bean with both option customizers set.
- **Evidence:** FR-6.2 (passkey is the only sign-in factor), OD-003 *(assumed)*. S-19 "Components and Terms" (request UV and verify the UV flag); S-20 §7.1 step 16, §7.2; S-42 (default is `PREFERRED`; the flag is enforced only when the options say `REQUIRED`; `setCustomizeCreationOptions`, `setCustomizeRequestOptions`).
- **Verification:** Pass: both options responses contain `"userVerification":"required"` and a ceremony with a verifying authenticator succeeds. Negative: an otherwise valid registration and an otherwise valid assertion from a virtual authenticator with user verification off are both rejected.

**SC-010 — Passkeys are registered only in the limited session, for the server-chosen account**
- **Status:** ACTIVE
- **Rule:** The passkey-registration routes MUST be reachable only from the limited session opened by a redeemed email link, and that session MUST be able to do nothing else. A full signed-in session MUST be refused on those routes. The account that receives the passkey MUST come from the server-side session, never from the request. The user-supplied passkey label MUST be length-limited and treated as plain text.
- **Enforcement:** Security layer (route rule and limited-session authority); Accounts.
- **Evidence:** AQ-3, TB-3, FR-6.4, FR-6.7. S-03 "Register a New Credential" (options are for the currently authenticated user); S-19 "Establish an Authorized Account Context". Project decision: full sessions are refused because no requirement lets a signed-in user add a passkey.
- **Verification:** Pass: a limited session registers one passkey and the account becomes usable. Negative: a limited session calling any other protected route gets 403; a full session calling a registration route gets 403; a request naming another account still registers against the session's account.

**SC-011 — Durable, unique credential records with an opaque user handle**
- **Status:** ACTIVE
- **Rule:** Passkey records MUST be stored in PostgreSQL through Accounts, never in the framework's in-memory default. The credential ID MUST carry a database uniqueness constraint across all accounts. The WebAuthn user handle MUST be a random value that contains no email address or display name and is never reassigned. Raw credential IDs MUST NOT appear in the UI, URLs, or logs.
- **Enforcement:** Accounts (implements the framework's credential and user-entity repository interfaces); PostgreSQL constraint.
- **Evidence:** DR-7, ARCHITECTURE.md Components (Accounts owns passkeys). S-03 "JDBC & Custom Persistence" (default is in-memory); S-19 "Verify the Registration Response", "Privacy and Operations"; S-20 §7.1 steps 26 to 27.
- **Verification:** Pass: a passkey still signs in after the application restarts. Negative: inserting a second record with an existing credential ID fails at the database; the stored user handle does not contain or decode to the email address.

**SC-012 — Passkey is the only way in**
- **Status:** ACTIVE
- **Rule:** The filter chain MUST NOT enable form login, HTTP Basic, remember-me, or one-time-token login, and MUST NOT serve any framework-generated login or registration page. An email link MUST lead only to the limited session of SC-010, never to a full session. A failed or cancelled passkey ceremony MUST NOT fall back to any other method. Sign-in MUST NOT ask for an email address or username, and the sign-in options response MUST be identical for every caller. Attestation MUST NOT be required.
- **Enforcement:** Security layer.
- **Evidence:** FR-6.2, FR-6.3, AQ-3, DESIGN.md "Error messages". S-19 "Use Safe Failure Behavior", "Minimize Attestation", "Generate and Bind Authentication Options"; S-11 (one-time-token login signs the user in fully, so it is not used). S-42: `disableDefaultRegistrationPage(true)` removes the generated registration page, and attestation already defaults to `NONE`.
- **Verification:** Pass: sign-in works with a discoverable passkey and no typed identifier. Negative: `POST /login`, `POST /ott/generate`, and a request with a Basic `Authorization` header create no session; `GET /login` and `GET /webauthn/register` return no framework page; redeeming an email link and then calling a quiz route is refused.

**SC-013 — A full session only after every check passes**
- **Status:** ACTIVE
- **Rule:** A full session MUST be created only after the framework has verified the assertion and the account is confirmed and active. The session ID MUST change at that moment. Every sign-in failure MUST return the same response whatever the cause.
- **Enforcement:** Security layer, asking Accounts.
- **Evidence:** FR-6.3, FR-7.7, ARCHITECTURE.md flow 2. S-19 "Verify the Authentication Response" (create the session only after all checks; rotate the session identifier); S-05 "Understanding Session Fixation Attack Protection"; S-21 "Session ID Regeneration".
- **Verification:** Pass: the session cookie value before sign-in differs from the value after. Negative: a valid passkey for a deactivated or unconfirmed account gets the same failure response as a bad signature, and no session.

### Email links and recovery

**SC-014 — Email-link tokens**
- **Status:** ACTIVE
- **Rule:** Confirmation and recovery tokens MUST be generated with `SecureRandom`, be tied to one account and one purpose, and work once. Consuming a token MUST be a single atomic database step so two simultaneous uses cannot both succeed. A recovery token MUST stop working 30 minutes after it is sent. Issuing a new token MUST cancel earlier unused tokens of the same purpose for that account.
- **Enforcement:** Accounts.
- **Evidence:** FR-6.6 to FR-6.8, TB-3. S-24 "General Security Practices" (random, long enough, linked to a user, single-use, expiring, stored securely); S-25 "Email Ownership Verification". Project decisions, not from a source: at least 32 random bytes; only a SHA-256 hash of the token is stored; newer tokens cancel older ones.
- **Verification:** Pass: a fresh token opens the limited session once. Negative: a second use fails; a recovery token fails at 30 minutes and 1 second; two parallel redemptions give exactly one success; a recovery token is refused on the confirmation path; the database holds no value equal to a token that was emailed.

**SC-015 — Confirmation-link lifetime**
- **Status:** ACTIVE
- **Rule:** A confirmation link MUST stop working 24 hours after it is sent.
- **Enforcement:** Accounts.
- **Evidence:** FR-6.6 gives no lifetime; OD-008 *(assumed)*. S-24 and S-25 (tokens are time-limited).
- **Verification:** Pass: a confirmation token works at 23 hours 59 minutes. Negative: the same token is refused at 24 hours and 1 second, with the same response as an unknown token.

**SC-016 — Links are redeemed by POST and leave no trace**
- **Status:** ACTIVE
- **Rule:** Opening an emailed link with GET MUST NOT consume the token or change any state; the client MUST send the token in a CSRF-protected POST. The client MUST remove the token from the address bar before rendering anything else, and the page MUST be served with `Referrer-Policy: no-referrer`. Tokens and full link URLs MUST NOT be logged or kept in router state or web storage.
- **Enforcement:** Web client (link landing screen); API edge; Accounts.
- **Evidence:** TB-3. S-22 (no state change on GET); S-24 "URL Tokens" (`no-referrer`); S-25 (never log tokens or full URLs); S-01 "Keep Sensitive Data Out of URLs".
- **Verification:** Pass: following a link and confirming registers a passkey. Negative: fetching the link URL with GET twice leaves the token usable; after the landing screen loads, `location.href` no longer contains the token; no log line contains the token.

**SC-017 — Sign-up and recovery never reveal whether an account exists**
- **Status:** ACTIVE
- **Rule:** The account-creation and recovery-request routes MUST return the same status, body, and headers whether or not the email address has an account. The email MUST be handed off and sent after the response is produced, on a bounded in-process executor, so the response time does not depend on it. A failure to send MUST NOT change the response.
- **Enforcement:** Accounts; Email gateway.
- **Evidence:** FR-6.5, FR-6.9, DESIGN.md "Sign-up with an email already in use", ARCHITECTURE.md conflict 3. S-24 "Forgot Password Request" (same message, consistent time); S-25 "Anti-Enumeration Controls".
- **Verification:** Pass: both cases show "Check your email to finish" / the fixed recovery message. Negative: byte-for-byte comparison of the two responses shows no difference; with the mail catcher stopped, the response is still the same.

**SC-018 — Links and emails are built from configuration**
- **Status:** ACTIVE
- **Rule:** The base address in every emailed link MUST come from server configuration, never from the `Host` or any forwarded header. Email bodies MUST be plain text. User-supplied values MUST NOT be placed in any email header; the recipient address MUST be the stored, validated address.
- **Enforcement:** Email gateway; Accounts.
- **Evidence:** TB-7, DR-8. S-24 "URL Tokens" (do not build the link from the Host header). Project decision: plain-text bodies and no user values in headers.
- **Verification:** Pass: the emailed link starts with the configured base address. Negative: a sign-up request sent with a forged `Host` header still produces a link with the configured address; a display name containing a line break does not add a header to the email.

**SC-019 — Recovery replaces every earlier passkey and ends every session**
- **Status:** ACTIVE
- **Rule:** When a passkey is registered through a recovery link, the server MUST, in the same transaction, remove every earlier passkey of that account and cancel its other unused tokens, and MUST end every session of that account. It MUST then send a plain-text notice to the account's confirmed address. The person MUST sign in with the new passkey afterwards; recovery itself MUST NOT produce a full session. The same path applies to administrators.
- **Enforcement:** Accounts; Security layer.
- **Evidence:** FR-6.7, OD-007 *(assumed)*. S-19 "Account Recovery and Bootstrap" (after recovery rotate sessions, review or revoke credentials, notify); S-24 "User Resets Password" (do not sign in automatically; invalidate sessions; notify).
- **Verification:** Pass: after recovery the new passkey signs in and a notice email is in the mail catcher. Negative: the old passkey is refused; a session that was open before recovery gets 401 on its next request; a second recovery token issued earlier no longer works.

**SC-020 — Rate limits on sign-up, recovery, sign-in, and link redemption**
- **Status:** ACTIVE
- **Rule:** The server MUST enforce these limits in process: at most 3 emails to one address per hour; at most 20 email-triggering requests from one source address per hour; at most 30 passkey ceremony starts or link redemptions from one source address per minute. Over an email limit, the response MUST stay identical to the normal one (SC-017) and no email is sent. Over the other limits, the response MUST be 429. The source address MUST be the connection's address, never a forwarded header. No limit MAY lock or change an account. The limiter's memory use MUST be bounded.
- **Enforcement:** Security layer (in-process limiter, no new package).
- **Evidence:** OD-006 *(assumed)*. S-24 "Forgot Password Request" (rate-limit to stop flooding), "Account Lockout" (do not lock accounts); S-19 "Use Safe Failure Behavior". Project decision: the numbers, the in-memory design, and ignoring forwarded headers (there is no proxy).
- **Verification:** Pass: 3 recovery requests for one address in an hour send 3 emails. Negative: the 4th sends none and its response is byte-identical to the first; the 31st ceremony start in a minute gets 429; a forged `X-Forwarded-For` header does not reset a count; after the limits are hit, the account still signs in with its passkey.

### Sessions

**SC-021 — Session cookie attributes that do not depend on HTTPS**
- **Status:** ACTIVE
- **Rule:** The session cookie MUST be `HttpOnly`, `SameSite=Strict`, host-only (no `Domain`), `Path=/`, and non-persistent, and each of these MUST be set explicitly in configuration. Session tracking MUST be cookie-only: a session ID MUST never appear in a URL. The cookie name MUST NOT be the container default. Sessions MUST NOT be written to disk.
- **Enforcement:** Application configuration (`server.servlet.session.*`); Security layer.
- **Evidence:** AQ-9. S-21 "Cookie Attributes", "Session ID Exchange", "Session ID Properties"; S-12 (`server.servlet.session.cookie.*`, `tracking-modes`, `persistent`); S-13 (SameSite). Project decision: `Strict` rather than `Lax`.
- **Verification:** Pass: the `Set-Cookie` header at sign-in shows `HttpOnly; SameSite=Strict; Path=/`, no `Domain`, no `Max-Age`/`Expires`. Negative: no response or redirect ever contains `;jsessionid=`; a request carrying the session ID as a URL parameter is treated as signed out.

**SC-022 — Plain HTTP is allowed only on the loopback address**
- **Status:** ACTIVE
- **Rule:** The application MUST be served only at `http://localhost:8080`, bound to the loopback address (SC-048). Only while that holds, the session cookie is not marked `Secure`; this is a recorded deviation from the source. The application MUST refuse to start if its bind address is not a loopback address while the cookie is not `Secure`. If it is ever served over HTTPS or on another host, the cookie MUST be `Secure` and named with the `__Host-` prefix, and the origin list (SC-007) MUST be HTTPS only.
- **Enforcement:** Application configuration; a start-up check in the Security layer.
- **Evidence:** TB-1, AQ-1, OD-002 *(assumed)*. S-21 "Cookie Attributes" (`Secure` is mandatory; use `__Host-`) — the demo deviates from this on the loopback address only; S-06 (HSTS is added only on HTTPS responses); S-22 "Using Cookies with Host Prefixes".
- **Verification:** Pass: the application starts with `server.address` set to the loopback address. Negative: starting with `server.address=0.0.0.0` and a non-`Secure` cookie fails at start-up.

**SC-023 — Session lifetime**
- **Status:** ACTIVE
- **Rule:** A full session MUST end after 30 minutes without a request and, whatever the activity, 8 hours after sign-in. The limited session from an email link MUST end 15 minutes after it opens. All three MUST be enforced on the server. An open event stream or its heartbeat MUST NOT count as activity.
- **Enforcement:** Application configuration (`server.servlet.session.timeout`); Security layer (total-time check on each request).
- **Evidence:** AQ-9, OD-004 *(assumed)*. S-21 "Session Expiration Timeouts" (idle and absolute timeouts, enforced server-side); S-12 (`server.servlet.session.timeout`, default `30m`). Project decision: the three values and the stream exclusion; the total-time check is project code because no built-in setting for it was found in the sources read.
- **Verification:** Pass: a request at 29 minutes idle succeeds. Negative: with the clock advanced, a request at 31 minutes idle gets 401; a session kept active gets 401 after 8 hours; a limited session gets 401 after 15 minutes; a session with only an open stream still expires at 30 minutes.

**SC-024 — Sign-out and forced session end**
- **Status:** ACTIVE
- **Rule:** Sign-out MUST be a CSRF-protected POST handled by the framework's logout support. It MUST invalidate the server-side session, send `Clear-Site-Data` for cookies, and return a status code with no redirect. When an account is deactivated or deleted, every session and open event stream of that account MUST end at once, and a deactivated account's next request MUST be refused even if a session somehow remains.
- **Enforcement:** Security layer; Accounts announces deactivation and deletion (DR-6).
- **Evidence:** FR-6.10, FR-7.7, DESIGN.md "Signed-in state", ARCHITECTURE.md flow 9. S-10 (default logout steps, `HttpStatusReturningLogoutSuccessHandler`, `Clear-Site-Data`, do not write a custom endpoint); S-21 "Manual Logout".
- **Verification:** Pass: after sign-out the old cookie value is refused. Negative: `GET /logout` does not end the session; a POST to sign-out without a CSRF token is refused; a user deactivated while signed in gets 401 on the next request and their event stream closes.

### CSRF and cross-origin

**SC-025 — CSRF protection on every state change; no cross-origin access**
- **Status:** ACTIVE
- **Rule:** The framework's CSRF protection MUST stay on for every unsafe request, including the passkey routes, link redemption, and sign-out; it MUST NOT be disabled or given exemptions. No GET, HEAD, or OPTIONS request MAY change state. The server MUST NOT configure CORS in any form (no CORS filter, no `@CrossOrigin`), and no response MAY carry `Access-Control-Allow-Origin`. The client MUST call only its own origin.
- **Enforcement:** Security layer; API edge; web client (`fetch` wrapper).
- **Evidence:** TB-1, DR-1, DR-9. S-04 (on by default; "Login", "Logout"; passkey routes need the token per S-03); S-22 (use the framework's protection; no state change on GET; restrict CORS); S-31 (`Access-Control-Allow-Origin`).
- **Verification:** Pass: a POST with a valid token succeeds. Negative: the same POST with no token, and with a wrong token, returns 403 on every unsafe route, including `/webauthn/authenticate/options` and sign-out; a request with a foreign `Origin` header gets no `Access-Control-Allow-Origin` header back; no route answers a state-changing GET.

**SC-026 — How the client gets the CSRF token**
- **Status:** ACTIVE
- **Rule:** The CSRF token MUST be kept in the server session (the framework default repository and request handler), handed to the client only by a same-origin GET token route, and sent back in the `X-CSRF-TOKEN` request header. It MUST NOT be placed in a cookie or a URL, and `csrf.spa()` MUST NOT be used. The client MUST fetch the token again after sign-in, sign-out, and link redemption, and MUST keep it in memory only.
- **Enforcement:** Security layer (token route); web client (`fetch` wrapper).
- **Evidence:** OD-009 *(assumed)*. S-04 "Using the HttpSessionCsrfTokenRepository", "Using the XorCsrfTokenRequestAttributeHandler", "Mobile Applications" (token route; fetch again after sign-in and sign-out); S-22 "Transmitting CSRF Tokens in Synchronized Patterns" (header, not cookie or URL); S-03 (the passkey examples send `X-CSRF-TOKEN`).
- **Verification:** Pass: a POST carrying the token from the token route succeeds. Negative: no response sets an `XSRF-TOKEN` cookie; a token fetched before sign-in is refused after sign-in; a token sent as a query parameter only is refused; the token is absent from `localStorage` and `sessionStorage`.

### Input validation

**SC-027 — Declared validation on every request**
- **Status:** ACTIVE
- **Rule:** Every request body, path value, and query value MUST bind to a purpose-built input type with declared constraints, and validation MUST be triggered at the API edge before any component runs. Every text field MUST have an explicit maximum length. Unknown JSON properties MUST be rejected. Validation in the client is for convenience only and MUST be repeated on the server.
- **Enforcement:** API edge (Bean Validation, Jackson settings).
- **Evidence:** ARCHITECTURE.md Components (API edge), FR-1.8, FR-4.21, FR-6.5. S-26 (allow-list, server-side validation); S-27 (declared constraints, `@Valid`, cascading); S-28; S-12 (`spring.jackson.deserialization.*`). Project decision: reject unknown properties.
- **Verification:** Pass: a well-formed request is accepted. Negative: a missing field, a wrong type, an over-length string, and an extra unknown property each return 400 and reach no component; a chat message of 0 or 2,001 characters is refused with the reason (FR-4.21).

**SC-028 — Business rules are checked by the owning component**
- **Status:** ACTIVE
- **Rule:** After shape validation, the owning component MUST check meaning against stored state: the move is one of the offered choices, the quiz has no answer yet, the bet size is legal for the rules and the stack, the table size is 2 to 9, the retake wait has passed, the caller is the student and not the tutor in a live game. The server MUST recompute legality itself and MUST NOT trust a list of legal moves, a pot, a stack, or a result sent by the client.
- **Enforcement:** Quiz, Simulation, Progression, Tutoring, using Poker rules.
- **Evidence:** FR-1.6, FR-1.8, FR-1.9, FR-2.2, FR-2.6, FR-2.7, FR-4.13, FR-4.16, FR-4.23, DR-10. S-26 (semantic validation).
- **Verification:** Pass: a legal move is applied. Negative: a second answer to the same quiz is refused and the first stays; a bet of one chip more than the stack is refused; a table size of 1 or 10 is refused; a tutor's move in a live game is refused; a retake inside 24 hours is refused.

**SC-029 — Email address and display name**
- **Status:** ACTIVE
- **Rule:** Email addresses MUST be validated with the validation library's email constraint plus a length limit, compared in one documented canonical form everywhere (sign-up, uniqueness, recovery), and backed by a database uniqueness constraint on that form. Display names MUST be length-limited and stored as entered; they MUST NOT be used as an identifier or in any authorization decision.
- **Enforcement:** API edge; Accounts; PostgreSQL constraint.
- **Evidence:** FR-6.4, FR-6.5, FR-6.11. S-25 "Email Canonicalization", "Email Format Validation", "Case Sensitivity"; S-26 (use a maintained validation library). The maximum lengths are in SC-039.
- **Verification:** Pass: a valid new address creates an unconfirmed account. Negative: a malformed address is refused; the same address in different letter case does not create a second account; two users may share a display name without either gaining the other's access.

**SC-030 — Bounded pattern matching**
- **Status:** ACTIVE
- **Rule:** Any regular expression applied to user-supplied text MUST run on length-limited input, match the whole value, and avoid nested or overlapping repetition. The parser for the administrator content notation (AQ-11) MUST run in time proportional to its input.
- **Enforcement:** API edge; web client content parser.
- **Evidence:** AQ-11. S-26 (bound length before matching; avoid excessive backtracking).
- **Verification:** Pass: normal content parses. Negative: a maximum-length input of repeated `*`, `-`, and card-like characters is parsed within a fixed small time budget in a test.

**SC-031 — Parameterized database access only**
- **Status:** ACTIVE
- **Rule:** All database access MUST go through Spring Data JPA with bound parameters. Query text MUST NOT be built by joining strings with any request-derived value. Sort and filter fields MUST be chosen from a fixed list in code; `JpaSort.unsafe` MUST NOT be used. Any `LIKE` search on user text MUST escape wildcards.
- **Enforcement:** Persistence code in each owning component (DR-8).
- **Evidence:** AQ-8, DR-8. S-29 (parameterized queries); S-15 (parameter binding, `JpaSort.unsafe`, LIKE escaping).
- **Verification:** Pass: repository tests run against real PostgreSQL. Negative: a display name or chat message containing quotes, semicolons, and `%` is stored and returned unchanged and alters no other row; a request for an unlisted sort field is refused; a code search finds no query string concatenation and no `JpaSort.unsafe`.

### Output and rendering

**SC-032 — No raw markup and no dynamic code in the client**
- **Status:** ACTIVE
- **Rule:** Client code MUST NOT use `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `eval`, `new Function`, or a string argument to `setTimeout` / `setInterval`. No HTML-rendering or sanitizing library MAY be added.
- **Enforcement:** Web client; checked in the build by a source search (no linter is in the stack).
- **Evidence:** DESIGN.md "User-supplied content", ARCHITECTURE.md "Deliberately not added". S-01 "Avoid Unsafe HTML Injection", "Avoid Direct DOM Manipulation", "Avoid Dynamic Code Execution"; S-40.
- **Verification:** Pass: the client test command includes a check that finds none of these names in the client source. Negative: adding any one of them to a source file makes that check fail.

**SC-033 — User text is always plain text**
- **Status:** ACTIVE
- **Rule:** Display names, chat messages, and passkey labels MUST be rendered only as React text children. No user- or server-supplied value MAY be used as an `href`, `src`, `action`, `style`, or event-handler value, and no object from a server response MAY be spread into element props. Links in chat MUST NOT be clickable. System notices MUST be produced by client code from a fixed event type, never from message text.
- **Enforcement:** Web client.
- **Evidence:** DESIGN.md "User-supplied content", "Chat"; TB-4. S-01 "Validate URLs Before Rendering", "Avoid Prop Injection via Spread Syntax".
- **Verification:** Pass: a normal message shows as typed. Negative: a chat message and a display name containing `<img src=x onerror=…>`, `javascript:` text, and a URL each render as literal characters, create no element or link, and run nothing; a chat message that copies the wording of a system notice is still shown in a chat bubble.

**SC-034 — Administrator content uses the fixed notation only**
- **Status:** ACTIVE
- **Rule:** Explanations and lessons MUST be stored as plain text and turned into React elements only by the client's own parser, which produces paragraphs, bold, lists, and card components and shows everything else as literal text. The parser MUST NOT produce links, images, or attributes from content.
- **Enforcement:** Web client parser; owning component stores text only.
- **Evidence:** AQ-11, TB-5, DESIGN.md "User-supplied content". S-01 "Avoid Unsafe HTML Injection" (render as text wherever possible).
- **Verification:** Pass: `**bold**`, `- item`, and `Ad` render as bold, a list item, and a card. Negative: content containing `<script>`, `<a href>`, and `[text](url)` renders as those literal characters.

**SC-035 — Content Security Policy and response headers**
- **Status:** ACTIVE
- **Rule:** Every response MUST carry, as HTTP headers: a Content Security Policy of `default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self'; font-src 'self'; connect-src 'self'; form-action 'self'; base-uri 'none'; frame-ancestors 'none'`; `Referrer-Policy: no-referrer`; `Permissions-Policy` allowing only `publickey-credentials-get=(self), publickey-credentials-create=(self)`; and `Cross-Origin-Opener-Policy: same-origin`. Spring Security's default headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, no-store cache headers) MUST stay on. The policy MUST NOT contain `'unsafe-inline'` or `'unsafe-eval'`, so the client MUST NOT use inline scripts or inline `style` attributes that the policy would block.
- **Enforcement:** Security layer (`headers` configuration); web client build (no inline script).
- **Evidence:** DESIGN.md (self-hosted font, no third parties), DR-1. S-06 (defaults; CSP, Referrer-Policy, Permissions-Policy are not on by default); S-30 (header delivery, tight `default-src 'none'` form, avoid `unsafe-*`); S-31; S-01 "Other Considerations". Project decision: the exact policy string, `no-referrer` site-wide, and the passkey-only Permissions-Policy.
- **Verification:** Pass: the application works end to end with the policy enforced and the browser console shows no violation. Negative: a response to any route, including static files and errors, that lacks one of these headers fails the header test; a test page that injects an inline script is blocked by the browser.

**SC-036 — Send only what the user may see**
- **Status:** ACTIVE
- **Rule:** Responses and stream events MUST be built from purpose-built output types that list their fields. They MUST NOT contain other players' private cards before showdown, undealt cards, accepted moves or explanations before the answer is locked, win probability before a quiz answer, other users' email addresses, credential data, or internal fields. The client MUST NOT keep session or account secrets in `localStorage`, `sessionStorage`, router state, or URLs.
- **Enforcement:** Quiz, Simulation, Tutoring, Accounts (output types); web client.
- **Evidence:** DR-10, FR-1.11, FR-2.8, FR-2.11, FR-3.4, FR-4.24, FR-6.11. S-01 "Minimize Sensitive Data in Component State and Props", "Keep Sensitive Data Out of URLs"; S-21 "Client-Side Storage".
- **Verification:** Pass: after locking an answer the response includes the explanation and win probability. Negative: the raw JSON of a quiz hand before answering contains no accepted move, explanation, or win probability; the raw JSON of a running simulation contains only the caller's own private cards; the tutor list contains no email address.

### Errors

**SC-037 — Generic errors**
- **Status:** ACTIVE
- **Rule:** One global handler MUST turn every unhandled failure into a fixed, generic response with the right status code (4xx for caller errors, 5xx only for server faults). Responses MUST NOT contain stack traces, exception names, SQL, internal IDs, which security check failed, or whether another account exists. `spring.web.error.include-stacktrace`, `include-message`, and `include-binding-errors` MUST be `never`, `include-exception` MUST be `false`, and the whitelabel page MUST be off. Full detail goes to the server log only.
- **Enforcement:** API edge (`@RestControllerAdvice`); application configuration.
- **Evidence:** DESIGN.md "Error messages". S-32 (global handler, generic message, log detail server-side, status codes); S-12 (`spring.web.error.*` names and defaults); S-23 "Exit Safely When Checks Fail".
- **Verification:** Pass: a forced server fault returns the generic 500 body and a log entry with the detail. Negative: malformed JSON, a wrong content type, an unknown route, a constraint violation, and a forced exception each return a body with no class name, stack frame, or SQL text.

### Request and resource limits

**SC-038 — Request limits**
- **Status:** ACTIVE
- **Rule:** Multipart handling MUST be switched off (`spring.servlet.multipart.enabled=false`). Tomcat's header, form, parameter-count, and connection limits MUST NOT be raised above their Spring Boot defaults. A maximum JSON body size MUST be configured (the framework default is unlimited) and Jackson's nesting-depth limit MUST NOT be raised.
- **Enforcement:** Application configuration; API edge.
- **Evidence:** No requirement accepts files. S-12 (`spring.servlet.multipart.enabled` default `true`; `server.max-http-request-header-size` `8KB`; `server.tomcat.max-http-form-post-size` `2MB`; `server.tomcat.max-parameter-count` `1000`; `spring.jackson.factory.constraints.read.max-document-length` default `-1`, `max-nesting-depth` `500`); S-34 "Input validation". The size number is in SC-039.
- **Verification:** Pass: normal requests succeed. Negative: a multipart request is refused; a JSON body one byte over the configured maximum is refused with a 4xx and no component runs; a deeply nested JSON body is refused.

**SC-039 — Size numbers**
- **Status:** ACTIVE
- **Rule:** The server MUST enforce these maximums: JSON request body 128 KB; email address 254 characters; display name 40; passkey label 64; quiz explanation 4,000; lesson text 20,000; chat message 2,000. Display name, passkey label, explanation, and lesson text MUST also be non-blank.
- **Enforcement:** API edge (declared constraints); application configuration (body size).
- **Evidence:** FR-4.21 (chat), FR-7.4, OD-010 *(assumed)*. S-26; S-34 "Input validation".
- **Verification:** Pass: values at exactly each maximum are accepted. Negative: a value one character over each maximum returns 400 and is not stored; a body of 128 KB plus one byte is refused before binding.

**SC-040 — Bounded computation and no slow work on request threads**
- **Status:** ACTIVE
- **Rule:** Win-probability work MUST have a hard cap on iterations and on time, sized from server-side values only (table size 2 to 9), and the number of calculations running at once MUST be capped. No request value MAY set a loop count, sample size, thread count, or timeout. Email sending MUST run on a bounded executor with a bounded queue, never on the request thread. Outbound SMTP MUST have connect and read timeouts.
- **Enforcement:** Win probability; Simulation; Email gateway; application configuration.
- **Evidence:** NFR-2.2, NFR-2.3, AQ-4, ARCHITECTURE.md conflict 3. S-34 "Software Design Concepts" (avoid CPU-heavy operations; do not let input control resource allocation); S-14 (the default async executor is not suitable under load).
- **Verification:** Pass: a 9-player pre-flop calculation returns within 2 seconds. Negative: a request that tries to supply a sample size or table size of 10 is refused; with the mail catcher unreachable, sign-up still responds promptly and the executor queue does not grow without bound.

**SC-041 — Event streams**
- **Status:** ACTIVE
- **Rule:** An event stream MUST be opened only by a full session, and each event MUST be delivered only after re-checking that the recipient is a current participant. Each session MAY hold one stream; opening another closes the first. Every stream MUST have an explicit finite timeout and a periodic heartbeat, and its emitter MUST be removed on completion, timeout, and error. The stream MUST accept nothing from the client. Database connections MUST NOT be held for the life of a stream: `spring.jpa.open-in-view` MUST be `false`.
- **Enforcement:** API edge (stream endpoint); Tutoring.
- **Evidence:** AQ-7, DR-9, FR-4.20, FR-4.26, flow 7. S-14 "SSE", "Disconnects" (heartbeats; errors on send after disconnect), "Configuration"; S-12 (`spring.jpa.open-in-view` default `true`, binds an EntityManager for the whole request; `spring.mvc.async.request-timeout`); S-34. Project decision: one stream per session and `open-in-view=false`.
- **Verification:** Pass: both participants receive a chat message within 2 seconds (NFR-2.4). Negative: a signed-out request for the stream gets 401; a student who switches tutor stops receiving the old pairing's events at once; opening 50 streams from one session leaves one open and the connection pool unaffected; after a client disconnects, its emitter is gone from the registry.

### Logging

**SC-042 — Security events are logged**
- **Status:** ACTIVE
- **Rule:** The server MUST log, with time, event type, outcome, and the internal account ID when known: sign-in success and failure, passkey registration, email-link issue and redemption (success and failure), recovery requests, sign-out, session end by deactivation or deletion, authorization refusals, CSRF refusals, validation failures, every administrator action with its target, and account deletion.
- **Enforcement:** Security layer; Accounts; Administration; API edge.
- **Evidence:** ARCHITECTURE.md Inputs ("what to log … is owned by SECURITY.md"). S-33 "Which events to log", "Event attributes"; S-19 "Credential Management" (audit enrollment and use); S-21 "Logging and Monitoring".
- **Verification:** Pass: each listed event produces one log entry in a test. Negative: a refused administrator request by a non-administrator produces an authorization-refusal entry naming the caller's account ID and the route.

**SC-043 — What never goes in a log**
- **Status:** ACTIVE
- **Rule:** Logs MUST NOT contain session IDs, CSRF tokens, email-link tokens or full link URLs, WebAuthn challenges, credential IDs, attestation or assertion payloads, database or mail passwords, connection strings with credentials, email addresses, display names, or chat message text. Accounts are referred to by internal ID. `spring.jpa.show-sql` MUST be `false` and request bodies MUST NOT be logged.
- **Enforcement:** All server components; logging configuration.
- **Evidence:** FR-6.10, FR-6.11, FR-4.20. S-33 "Data to exclude"; S-19 "Use Safe Failure Behavior" (never log challenges, credential responses, session IDs); S-25 (never log tokens or URLs); S-12 (`spring.jpa.show-sql` default `false`). Project decision: email addresses, display names, and chat text are excluded so that account deletion (FR-6.10) does not leave them behind in logs.
- **Verification:** Pass: logs from a full sign-up, sign-in, chat, and recovery run exist. Negative: searching those logs for the test email address, the display name, the chat text, the emailed token, and the session cookie value finds nothing.

**SC-044 — Log entries cannot be forged**
- **Status:** ACTIVE
- **Rule:** Application logs MUST be written in a structured format whose encoder escapes line breaks and delimiters (Spring Boot structured logging via `logging.structured.format.console`). Log calls MUST pass values as parameters, never by joining strings, and any user-influenced value MUST be length-limited before logging.
- **Enforcement:** Logging configuration; all server components.
- **Evidence:** ARCHITECTURE.md Inputs (SLF4J with Logback, no extra package). S-33 "Event collection", "Log Injection" (parameterized logging alone does not stop CR/LF injection; use a structured encoder); S-12 (`logging.structured.format.console`).
- **Verification:** Pass: each log entry is one valid structured record. Negative: a request whose path and header values contain CR/LF and a fake log line produces exactly one record, with the line breaks escaped.

**SC-045 — Log destination and retention**
- **Status:** ACTIVE
- **Rule:** The application MUST write its logs to standard output only. It MUST NOT configure a log file, a network log destination, or any third-party log service.
- **Enforcement:** Logging configuration.
- **Evidence:** AQ-1 (one machine, demo), DR-1, OD-012 *(assumed)*. S-33 "Protection" (restrict access to logs; care before sending to third parties).
- **Verification:** Pass: log records appear on standard output. Negative: a configuration test fails if `logging.file.name`, `logging.file.path`, or a non-console appender is set; running the application creates no log file.

### Database, containers, and secrets

**SC-046 — Database authentication**
- **Status:** ACTIVE
- **Rule:** Every network connection to PostgreSQL MUST authenticate with `scram-sha-256`. The `trust`, `password`, and `md5` methods MUST NOT be configured for host connections, and `POSTGRES_HOST_AUTH_METHOD` MUST NOT be set to `trust`.
- **Enforcement:** Docker Compose file; PostgreSQL configuration.
- **Evidence:** TB-6, AQ-13. S-16 §20.1 and §20.5 (`trust` lets anyone log in as any user; `md5` is deprecated; `scram-sha-256` is the most secure method provided); §19.3 (`password_encryption` default `scram-sha-256`); S-17 (`POSTGRES_HOST_AUTH_METHOD`).
- **Verification:** Pass: the application connects with its password. Negative: a connection from the host with a wrong or empty password is refused; `pg_hba_file_rules` shows no `trust`, `password`, or `md5` rule for `host` records.

**SC-047 — Database roles and privileges**
- **Status:** ACTIVE
- **Rule:** The application MUST connect as a runtime role that is not a superuser, owns no objects, has none of `CREATEDB`, `CREATEROLE`, `REPLICATION`, `BYPASSRLS`, and holds only `SELECT`, `INSERT`, `UPDATE`, `DELETE` on the application's tables and `USAGE` on its sequences. The runtime role MUST NOT have `INSERT` or `UPDATE` on the administrator flag column. Migrations MUST run as a separate non-superuser owner role (`spring.flyway.user`), which is also the role the operator uses to set the administrator flag. The image's superuser MUST be used only to create these two roles when the database is first created. `CREATE` on the `public` schema MUST NOT be granted to `PUBLIC`.
- **Enforcement:** PostgreSQL roles and grants (database initialization script and migrations); application and Flyway configuration.
- **Evidence:** TB-6, TB-8, AQ-6, FR-7.2, OD-011 *(assumed)*. S-16 §21.2 (do most work as a role that is not a superuser), §5.8 (privileges, including on table columns), §5.10 (schema usage patterns); S-17 (`POSTGRES_USER` creates a superuser; initialization scripts; `*_FILE` secrets); S-12 (`spring.flyway.user`, `spring.flyway.password`). Project decision: the two-role split and the column-level block on the administrator flag.
- **Verification:** Pass: the application runs every feature as the runtime role. Negative, as the runtime role: `CREATE TABLE`, `DROP TABLE`, `ALTER TABLE`, and an `UPDATE` of the administrator flag each fail with a permission error; `pg_roles` shows the runtime and owner roles with `rolsuper`, `rolcreatedb`, `rolcreaterole`, `rolreplication`, and `rolbypassrls` all false.

**SC-048 — Local services listen on the loopback address only**
- **Status:** ACTIVE
- **Rule:** The application MUST bind to the loopback address (`server.address`). Every port published by Docker Compose (PostgreSQL, Mailpit SMTP, Mailpit web UI) MUST be published on `127.0.0.1` only. Mailpit's relay and "release" features MUST stay off so no email can leave the machine. The Vite dev server MUST keep `server.host` at `localhost` and MUST NOT set `server.allowedHosts` or `server.cors` to `true`.
- **Enforcement:** Application configuration; Docker Compose file; Vite configuration.
- **Evidence:** AQ-1, AQ-2, TB-7 ("must listen on `localhost` only"). S-18 (Mailpit listens on `0.0.0.0` by default; relay options); S-37 Rule #5a (bind published ports to `127.0.0.1`); S-16 §19.3 (`listen_addresses`); S-12 (`server.address`); S-41 "Server Options" (warnings on `allowedHosts` and `cors`).
- **Verification:** Pass: the application, database, and mail catcher are reachable at `127.0.0.1`. Negative: from another address of the same machine (its LAN address), connections to the application port, 5432, 1025, 8025, and the Vite port are refused.

**SC-049 — No secrets in the repository or the client bundle**
- **Status:** ACTIVE
- **Rule:** Passwords, tokens, and keys MUST NOT be committed, baked into an image, or given a default value in committed configuration; they MUST be supplied at run time from a file or environment that Git ignores. No secret MAY be placed in a `VITE_`-prefixed variable, and `envPrefix` MUST NOT be widened. Local `.env*` files holding secrets MUST be listed in `.gitignore`.
- **Enforcement:** Repository (`.gitignore`); application and Compose configuration; Vite configuration.
- **Evidence:** TB-6. S-01 "Do Not Expose Secrets Through Environment Variables"; S-41 "Env Variables and Modes"; S-37 Rule #12; S-17 "Docker Secrets".
- **Verification:** Pass: the application starts with secrets supplied from an ignored file. Negative: starting without the database password fails at start-up rather than falling back to a default; a search of the built client bundle and of the Git history for the test password finds nothing.

**SC-050 — Container settings**
- **Status:** ACTIVE
- **Rule:** Container images MUST be pinned to a specific version tag, never `latest`. No container MAY run privileged, mount the Docker socket, or add capabilities.
- **Enforcement:** Docker Compose file.
- **Evidence:** ARCHITECTURE.md Stack (PostgreSQL 18.x, Mailpit 1.33.x). S-37 Rules #1, #3, #9 (pinned image versions).
- **Verification:** Pass: `docker compose config` shows version-tagged images. Negative: a check over the Compose file fails if it finds `:latest`, an untagged image, `privileged: true`, `cap_add`, or `docker.sock`.

### Dependencies and code execution

**SC-051 — Only the listed dependencies**
- **Status:** ACTIVE
- **Rule:** The server and client MUST use only the packages listed in ARCHITECTURE.md. A package that appears in a guide or cheat sheet is not approved by that fact; adding any package requires changing ARCHITECTURE.md first. Server versions MUST come from the Spring Boot dependency management, not hand-pinned overrides.
- **Enforcement:** `pom.xml`; `package.json`; review.
- **Evidence:** ARCHITECTURE.md Stack ("Deliberately not added"). S-38 (verify packages before installing; typosquatting); S-39.
- **Verification:** Pass: the direct dependencies in `pom.xml` and `package.json` equal the lists in ARCHITECTURE.md. Negative: a check fails when a direct dependency not on those lists is present.

**SC-052 — Locked, repeatable client installs**
- **Status:** ACTIVE
- **Rule:** The client lockfile MUST be committed and every install in a build or check MUST use `npm ci`.
- **Enforcement:** Client build commands (ARCHITECTURE.md Toolchain).
- **Evidence:** ARCHITECTURE.md Toolchain. S-38 "Enforce the lockfile".
- **Verification:** Pass: `npm ci` succeeds on a clean checkout. Negative: `npm ci` fails when `package.json` and the lockfile disagree.

**SC-053 — Dependency audits**
- **Status:** ACTIVE
- **Rule:** Before any change is merged, `./mvnw org.owasp:dependency-check-maven:13.0.0:check`, `npm audit`, and `npm audit signatures` MUST have been run on that change. A known vulnerability MUST be fixed by upgrading, or recorded with a written reason and a suppression limited to that one CVE.
- **Enforcement:** Toolchain commands, run by the person merging (no CI system is defined in ARCHITECTURE.md).
- **Evidence:** AQ-12, ARCHITECTURE.md Toolchain. S-39 (automate from the start; suppress per CVE, not per dependency; document decisions).
- **Verification:** Pass: all three commands exit without unsuppressed findings. Negative: a suppression that names a whole dependency rather than a CVE, or has no written reason, fails review.

**SC-054 — No unsafe code execution or deserialization on the server**
- **Status:** ACTIVE
- **Rule:** Server code MUST NOT deserialize Java-native serialized data from any request, MUST NOT enable polymorphic type handling in JSON binding, and MUST NOT evaluate expressions, scripts, or class names built from request data. It MUST NOT start operating-system processes. Authorization expressions MUST be fixed text in code.
- **Enforcement:** All server components; JSON configuration.
- **Evidence:** ARCHITECTURE.md (JSON over HTTP only). S-35 (avoid native serialization; JSON binding is safe as long as polymorphism is not used). Project decision: no process execution and no expression evaluation of request data.
- **Verification:** Pass: JSON requests bind normally. Negative: a request with content type `application/x-java-serialized-object` is refused; a JSON body carrying a type-selector property (`@class`, `@type`) is refused as an unknown property (SC-027); a code search finds no `ObjectInputStream`, `Runtime.exec`, `ProcessBuilder`, or script-engine use.

**SC-055 — No prototype pollution paths in the client**
- **Status:** ACTIVE
- **Rule:** Client code MUST NOT copy or merge keys from server responses or user text into plain objects by computed key. Lookups keyed by content (for example card codes in the content parser) MUST use `Map`/`Set` or an object with no prototype.
- **Enforcement:** Web client.
- **Evidence:** AQ-11. S-36 (use `Map`/`Set`; create objects with no prototype).
- **Verification:** Pass: card codes resolve through the lookup. Negative: content and JSON containing the keys `__proto__`, `constructor`, and `prototype` are handled as ordinary text or rejected, and `({}).polluted` remains undefined afterwards in a test.

**SC-056 — Cryptographic randomness**
- **Status:** ACTIVE
- **Rule:** Card dealing, email-link tokens, and user handles MUST use `java.security.SecureRandom`. `java.util.Random`, `Math.random`, and time-based seeds MUST NOT be used for any of them. The server's deck and seed MUST never be sent to the client.
- **Enforcement:** Poker rules (dealing source passed in); Accounts; Simulation.
- **Evidence:** FR-2.5, FR-2.11, ARCHITECTURE.md ("Random dealing uses the JDK's `SecureRandom`"), DR-10. S-24 "General Security Practices"; S-20 §13.4.3.
- **Verification:** Pass: dealing tests use an injected `SecureRandom`. Negative: a code search finds no `java.util.Random` or `Math.random` in dealing, token, or handle code; simulation responses contain no deck order or seed.

### Privileged actions

**SC-057 — Fresh passkey check before privileged actions**
- **Status:** ACTIVE
- **Rule:** Account deletion and every administrator action MUST be refused unless the session holds a passkey assertion verified no more than 5 minutes earlier, checked on the server by the route's authorization rule. A refusal for this reason MUST be a 403 with a fixed code that tells the client to run a new passkey ceremony; it MUST NOT be a redirect to a login page. Nothing sent by the client MAY state when the last assertion happened.
- **Enforcement:** Security layer — a factor rule requiring the WebAuthn authority within 5 minutes on the privileged routes, and a missing-authority handler that returns the status.
- **Evidence:** DESIGN.md "Destructive and privileged actions" *(assumed)*; flows 8 and 9; OD-005 *(assumed)*. S-09 "FactorGrantedAuthority", "Selectively Requiring MFA", "Specifying a Valid Duration"; S-42 (the default handler for a missing `FACTOR_WEBAUTHN` authority redirects to `/login`; `defaultDeniedHandlerForMissingAuthority` replaces it); S-19 "Credential Management"; S-21 "Reauthentication for Risk Events".
- **Verification:** Pass: an administrator action 4 minutes after sign-in succeeds; after a refusal, a new assertion makes the same action succeed. Negative: with the clock advanced 6 minutes, account deletion and each administrator route return 403 with the fixed code and change nothing; the response is not a 3xx.

## Open Security Decisions

None.

OD-001 to OD-013 are resolved and recorded under [Decisions on former open decisions](#decisions-on-former-open-decisions). OD-001 was answered by the owner. OD-002 to OD-012 were answered on the owner's behalf at their request and are marked *(assumed)*. OD-013 was settled by reading the Spring Security source. No item is deferred to steps 07 or 09; the notes under [Follow-ups for other documents](#follow-ups-for-other-documents) list facts those steps should carry into the other files.
