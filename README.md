# Software Engineering Go Launch Checklist

A reusable production-readiness review for an AI agent. Apply it to a specific release, deployment environment, and expected workload. A code review alone cannot establish production readiness: some checks require deployed behavior, operational evidence, or a responsible person's decision.

## Use it with your AI agent

Give your coding agent this README and ask:

> Review this codebase against the Software Engineering Go Launch Checklist. Establish the release scope, evaluate every checklist ID, and produce the required evidence-backed report. Do not mark unchecked behavior as passing. Identify launch blockers and human decisions, and recommend GO, NO-GO, or NEEDS EVIDENCE. Perform a review only; do not change code or deploy.

If this helps you ship, star the repository so you can find it again. Suggestions and corrections are welcome through issues and pull requests.

## Instructions for the reviewing agent

1. **Establish scope.** Record the commit, environment, launch audience, critical user journeys, expected traffic and data volume, sensitive data, external services, and contractual commitments. Identify the launch owner. If these are unknown, report the gaps before assuming requirements.
2. **Determine applicability.** Review every item below. Conditional features are required only when the product, risk, or commitments call for them. Do not add SAML, multi-region hosting, billing, or other infrastructure just to satisfy this checklist.
3. **Gather evidence.** Inspect code, configuration, tests, dependency manifests, deployment settings, and operating procedures. Exercise critical behavior in an authorized test environment. Prefer synthetic data. Do not deploy, change production, charge real cards, send customer messages, or run disruptive tests as part of this review without authorization.
4. **Record a result for every ID:** `PASS`, `FAIL`, `UNKNOWN`, or `N/A`. Start unchecked. Use `PASS` only when all applicable parts of the item have evidence. A test file's existence, a comment, or a vendor's advertised capability is not proof that the deployed system works. `N/A` needs a reason; missing access or missing evidence is `UNKNOWN`.
5. **Separate verification from approval.** The agent can check implementation and collect evidence. Legal applicability, contractual acceptance, certifications, and risk acceptance require an accountable human or qualified reviewer. A checklist is not a certification.
6. **Deliver the report below.** Cite file paths and line numbers, commands and results, deployment/configuration references, or dated operational records. Redact secrets and customer data. Record what could not be checked.

## 1. Data design and correctness

- [ ] **D01 — Deliberate data model.** Entities, relationships, constraints, ownership, and tenant boundaries are documented and consistent with product behavior. Demonstrate that representative next-customer requirements fit without corrupting existing data.
- [ ] **D02 — Safe migrations.** Rehearse migrations against representative data volume. Verify runtime, locking behavior, compatibility during deployment, and recovery after a partial failure. Demonstrate a safe rollback or forward-repair procedure; do not assume every data migration is reversible.
- [ ] **D03 — Database capacity.** Inspect query plans and indexes for critical queries at expected scale. Verify connection-pool limits, timeouts, and behavior under concurrent traffic against database capacity.
- [ ] **D04 — Server-side validation.** Validate types, bounds, required values, and cross-field business rules at every write boundary. Invalid requests bypassing the UI are rejected without partial writes or unsafe errors.
- [ ] **D05 — Names, text, and identity inputs.** Test plus-addressed emails, apostrophes, accents, emoji, long values, and right-to-left text where supported. Storage, validation, display, search, and exports preserve intended values; normalization does not merge distinct identities.
- [ ] **D06 — Time and localization.** Test timestamps, date-only values, user timezones, daylight-saving transitions, and recurring schedules. Supported locales display dates, numbers, and translated text correctly.
- [ ] **D07 — Monetary correctness, if money is handled.** Use explicit currency and integer minor units or appropriate fixed-precision decimals. Verify rounding, currency precision, and applicable tax calculations with known examples; avoid binary floating-point arithmetic for monetary amounts.
- [ ] **D08 — Concurrency and duplicate submissions.** Test repeated clicks, concurrent writes, retries, and interrupted requests. Transactions, constraints, and idempotency prevent duplicate charges, records, or other side effects without suppressing distinct legitimate actions.
- [ ] **D09 — Large datasets and data exchange.** Verify stable pagination, sorting, and filtering at expected size. If imports/exports are offered, test large files, malformed rows, encodings, delimiters, legacy CSVs, permissions, formula injection, and useful failure reports.
- [ ] **D10 — Deletion and retention.** Verify accidental-deletion recovery where promised, including undo or soft deletion as appropriate. Enforce permanent deletion and retention policies across databases, files, caches, search indexes, logs, and vendors. Document backup expiry and how restored data is prevented from resurrecting completed deletion requests.

## 2. Identity and access

- [ ] **I01 — Authentication and account recovery.** Exercise signup, verification, login, logout, and recovery. Reset links expire, work once, and cannot change another account. Authentication errors do not unnecessarily expose account existence.
- [ ] **I02 — MFA and recovery, where required.** Verify enrollment, challenge, removal, recovery codes, and lost-device recovery. Sensitive changes require appropriate reauthentication, and recovery does not bypass account protections.
- [ ] **I03 — Session lifecycle.** Verify idle/absolute expiry as appropriate, secure cookie or token handling, logout invalidation, and revocation of all sessions for one user. Test password changes, account suspension, and permission changes against existing sessions.
- [ ] **I04 — Authorization and tenant isolation.** Test a role/resource/action matrix at server boundaries. User A cannot read or modify user B's or another tenant's data through IDs, APIs, files, exports, caches, search, or background jobs. Default access is denied unless granted.
- [ ] **I05 — Authentication abuse controls.** Test rate limits and automated-attack defenses on login, signup, reset, and MFA endpoints. Verify legitimate recovery and avoid allowing attackers to indefinitely lock out a known account.
- [ ] **I06 — Enterprise identity, if promised.** Exercise required SSO protocols, including SAML if applicable, and provisioning/deprovisioning such as SCIM. Verify disabled employees lose effective access, including existing sessions and credentials, within the agreed time.
- [ ] **I07 — Privileged support access, if offered.** Admin tools and impersonation enforce scoped permissions, clear session boundaries, and audit records identifying both the operator and affected account. Verify impersonation cannot silently become unrestricted permanent access.

## 3. Application and infrastructure security

- [ ] **S01 — Secrets and credentials.** Scan tracked files, relevant history, build outputs, client bundles, and accessible configuration for exposed secrets. Review known AI/chat disclosures without copying secrets into the report. Revoke or rotate exposed credentials and verify the old ones no longer work; deleting the text is insufficient.
- [ ] **S02 — Dependency and build integrity.** Confirm package names, provenance, registries, and lockfiles match intended dependencies. Review vulnerability findings and unsupported versions. Verify the release builds reproducibly from the committed source and locked dependencies.
- [ ] **S03 — Injection and request defenses.** Verify parameterized database queries, context-aware output encoding, safe rich-text handling, and CSRF protection where applicable. Review relevant attack surfaces, including SSRF, command injection, path traversal, and unsafe redirects. Generic input sanitization is not a substitute for boundary-specific defenses.
- [ ] **S04 — Browser and transport configuration.** Inspect deployed HTTPS/TLS, certificate renewal, CORS allowlists, cookies, and appropriate security headers. Verify authorized browser flows work and untrusted origins do not gain access. Confirm encryption at rest and key controls for sensitive stored data.
- [ ] **S05 — File uploads, if supported.** Test authorization, file/type/size limits, safe filenames, private download access, and appropriate scanning or quarantine. Uploaded content cannot execute as application code or expose other users' files.
- [ ] **S06 — Least privilege and access reviews.** Inspect human, CI, service-account, database, cloud, and AI-agent permissions. Verify environment separation, credential expiry/revocation, and the documented access-review cadence, including quarterly sign-off where required.
- [ ] **S07 — Auditability without sensitive logging.** Verify useful actor, action, target, time, and outcome records for privileged and sensitive actions; include record reads where required. Test that logs, traces, analytics, and errors exclude passwords, tokens, card data, and prohibited personal data. Restrict access and protect audit integrity.
- [ ] **S08 — Security assessment and response.** Review threat assessment and risk-appropriate security testing, including an independent penetration test when required. Verify remediation evidence, a vulnerability reporting/triage process, and a practiced credential-revocation procedure for a stolen laptop or compromised account.

## 4. Delivery, infrastructure, and recovery

- [ ] **O01 — Reproducible environments.** Verify documented infrastructure and configuration can recreate the service. Development, staging, and production have understood differences, separate credentials, and safe data separation. Critical production behavior is represented in staging.
- [ ] **O02 — Release controls.** Verify CI runs required checks before release, changes receive review, and branch/merge rules preserve concurrent work. Deploy a known artifact with traceable version and configuration; show that a failed required check blocks release.
- [ ] **O03 — Safe rollout and rollback.** Rehearse a rollout, smoke test, and rollback or forward recovery. Verify schema compatibility, in-flight requests/jobs, and failure behavior. Test feature disablement where flags are used and zero-downtime behavior where promised.
- [ ] **O04 — Capacity and performance.** Load-test representative journeys and datasets against agreed latency, error-rate, and throughput targets. Verify overload behavior, resource limits, and scaling where needed. Test caching, invalidation, and CDN behavior where used, including private-data boundaries.
- [ ] **O05 — Network and service configuration.** Verify production domains, DNS, certificate-renewal monitoring, routing, storage permissions, and external-service configuration. Check that a fresh client can reach the real deployed endpoints.
- [ ] **O06 — Durable background work, if used.** Test job persistence, timeouts, bounded retries with backoff, duplicate delivery, and worker restarts. Failed jobs are inspectable and safely replayable through a dead-letter queue or equivalent. Scheduled jobs behave correctly across multiple workers and missed runs.
- [ ] **O07 — Transactional email, if used.** Verify sending-domain SPF, DKIM, and DMARC configuration, delivery of critical messages, bounce/complaint handling, and rendering in supported mail clients. Exercise expired links and provider failures.
- [ ] **O08 — Backups and disaster recovery.** Verify automated backup frequency and retention against agreed recovery-point and recovery-time targets. Restore a real backup into isolation and measure recovery. Test point-in-time recovery where required. Document dependencies, ownership, and drill cadence, including annual exercises or multi-region recovery where commitments require them.
- [ ] **O09 — Resource and cost controls.** Verify storage lifecycle cleanup, budgets, quotas, spend alerts, and a response owner. Simulate a runaway workload safely; ensure alerts arrive in time and hard limits exist where a warning alone would not contain unacceptable spend.

## 5. Monitoring and operations

- [ ] **M01 — Diagnostic visibility.** Verify searchable structured logs, useful error grouping, correlation IDs, and latency/error/throughput dashboards. Trace a representative request across services where needed. Confirm monitoring reflects the deployed release and obeys S07's logging restrictions.
- [ ] **M02 — Detection and paging.** Test external uptime checks and alerts for critical failures, latency, queue backlog, and capacity. Trigger a safe test alert and verify it reaches the responsible human; thresholds and escalation paths are actionable.
- [ ] **M03 — Incident response.** Identify launch coverage/on-call ownership and a backup. Walk through runbooks for likely incidents, customer communication or a status page, escalation, and recovery. Review how incidents produce tracked corrective actions.
- [ ] **M04 — Day-to-day support.** Demonstrate that operations/support can diagnose and resolve common issues with appropriately permissioned tools. Document escalation paths, operational procedures, and customer-facing changes in a changelog or equivalent.

## 6. Tests and user experience

- [ ] **Q01 — Meaningful automated coverage.** Run unit tests for business rules, integration tests for service boundaries, and end-to-end tests for critical journeys such as signup, recovery, and checkout. Review assertions against intended behavior; include regressions for important past bugs and realistic synthetic test data.
- [ ] **Q02 — Failure and recovery paths.** Exercise network loss, slow services, expired sessions, partial failures, and interrupted submissions. Verify actionable errors, useful empty/loading states, preserved form input, correct navigation/back behavior, and undo where the product promises it.
- [ ] **Q03 — Supported devices and connections.** Exercise supported browsers, mobile layouts, older/low-powered devices, and slow connections. Verify launch performance targets on these conditions, not only a developer's laptop.
- [ ] **Q04 — Accessibility.** Test keyboard access, focus order, labels, contrast, error announcements, and screen-reader use on critical journeys. Combine automated checks with manual testing against the agreed accessibility target and contractual requirements.
- [ ] **Q05 — Communication preferences.** If notifications are offered, verify preferences are respected across channels and distinguish optional notifications from necessary account/security messages. Check notification content and links for privacy and correctness.

## 7. Billing — if the product takes payments

- [ ] **B01 — Payment and subscription lifecycle.** In a sandbox, exercise purchases, renewals, upgrades, downgrades, cancellation, proration, failed-payment retries, and dunning. Test duplicate and out-of-order provider events; entitlements match payment state. Apply D07 and D08.
- [ ] **B02 — Invoices, taxes, and adjustments.** Verify invoice details, PO numbers where needed, applicable sales tax/VAT behavior, credits, full/partial refunds, and supported custom/annual contract terms against approved examples.
- [ ] **B03 — Metering and reconciliation.** Reconcile representative usage through metering, customer-visible totals, invoices, payment-provider records, and finance reports. Test rounding boundaries, corrections, and delayed/duplicate events.
- [ ] **B04 — Payment-data scope.** Document payment-data flows and obtain an appropriate PCI-scope assessment. Verify card data stays within the approved payment handling design and does not leak into application databases, logs, analytics, or support tools.

## 8. APIs and integrations — if exposed or consumed

- [ ] **P01 — API contract and customer credentials.** Verify documented schemas, compatibility/versioning policy, safe errors, and scoped API-key creation, storage, rotation, and revocation. Test rate limits against both abuse and legitimate integration throughput. Provide a sandbox when promised or needed for safe integration testing.
- [ ] **P02 — Incoming and outgoing webhooks.** Verify incoming signatures and replay protections before processing. Test duplicate/out-of-order delivery. For outgoing webhooks, verify authentication/signing, bounded retries, delivery history, failure visibility, and safe replay. Apply D08 to side effects.
- [ ] **P03 — External-service resilience.** Exercise OAuth refresh/revocation, timeouts, outages, malformed or changed responses, and rate-limit handling. Verify bounded retries, useful fallback/degradation, contract tests, and an owner for provider changes.

## 9. AI features — if the product uses models or agents

- [ ] **A01 — Model data boundaries.** Map prompts, retrieved content, outputs, logs, and provider retention. Verify only approved data reaches approved providers/regions under the applicable agreements. Redaction and access controls match actual data flows, including debugging tools.
- [ ] **A02 — Prompt injection and tool authority.** Test hostile instructions in user input, files, retrieved content, and tool output. Treat model output as untrusted. Enforce permissions, tenant isolation, destination restrictions, and tool scopes outside the model; instructions alone are not a security boundary.
- [ ] **A03 — Human approval for consequential actions.** Verify the product requires appropriate approval before irreversible or high-impact AI actions. Bind approval to the specific action and parameters; test that retries, changed arguments, or injected text cannot bypass it.
- [ ] **A04 — Quality and model changes.** Run representative evaluations and safety/abuse cases against defined acceptance thresholds. Record model and prompt versions, regression results, and a controlled upgrade/rollback path. Document ownership and a migration plan for provider deprecation.
- [ ] **A05 — AI reliability and cost.** Test provider outages, rate limits, timeouts, and the promised fallback. Enforce per-customer and global spend/token/tool-call limits, including loop termination. Verify debugging records remain useful while respecting A01 and S07.

## 10. Privacy, contracts, and company ownership

- [ ] **G01 — Obligations and truthful disclosures.** Have the responsible owner identify applicable privacy, security, accessibility, tax, and sector-specific obligations and customer commitments. Compare terms, privacy policy, security questionnaires, and marketing claims with implementation evidence. Record unresolved questions for qualified review.
- [ ] **G02 — Regulated or contracted data, if applicable.** Obtain appropriate review of HIPAA/BAA requirements, GDPR rights handling, residency commitments, and vendor/subprocessor arrangements, including AI providers. Verify the approved controls across storage, processing, support access, logs, and recovery. Use D10 for deletion behavior; do not assume immediate erasure of every backup is the required or implemented policy.
- [ ] **G03 — Assurance commitments, if applicable.** If SOC 2 or another formal assurance is promised, inspect its actual scope, status, policies, evidence, and required independent assessment. A completed checklist or planned audit does not establish certification or an issued report.
- [ ] **G04 — Company control and continuity.** Verify company ownership/control of domains, repositories, hosting, payment, email, and vendor accounts. Test recovery and access handoff. At least one backup person can deploy, revoke access, and restore service using the documentation.

## 11. Maintainability and handoff

- [ ] **H01 — Understandable implementation.** Review critical code for understandable structure, clear boundaries, abandoned attempts/dead code, and unnecessary duplication. Split oversized components when their responsibilities obstruct review or safe changes; file length alone is not a launch criterion.
- [ ] **H02 — Reproducible onboarding and decisions.** Have a fresh environment follow the README through setup, configuration, tests, and local startup within the team's target time. Document consequential architecture decisions and their reasons, plus deployment, recovery, and maintenance procedures for the next owner.
- [ ] **H03 — Maintenance ownership.** Assign owners and a cadence for dependency review/upgrades, framework support deadlines, vulnerability remediation, access reviews, recovery drills, and recurring verification. Verify these are tracked work with responsible people.

## Required review output

Create a report with these fields:

```text
Release / commit:
Environment / deployed version:
Review date / reviewer:
Launch scope and critical journeys:
Expected workload and performance/recovery targets:
Launch owner:

ID | Status | Evidence / reproduction | Gap and impact | Owner / next action
[One row for every checklist ID, including explained N/A results.]

Blockers:
Nonblocking follow-ups, with owners and deadlines:
Unknowns / unavailable access / tests not run:
Human reviews or approvals still required:
Accepted exceptions: ID, impact, named approver, mitigation, expiry
Recommendation: GO / NO-GO / NEEDS EVIDENCE
```

**Decision rule:** Recommend **NO-GO** for a verified unresolved blocker: for example, broken critical journeys, unauthorized access, exposed active secrets, unacceptable data-loss risk, or an unmet mandatory commitment. Recommend **NEEDS EVIDENCE** when required behavior or approvals cannot be verified. Recommend **GO** only when applicable requirements pass or the responsible owner explicitly accepts a permissible, bounded exception. An exception cannot waive a binding legal obligation. Low-risk improvements may become owned follow-ups; the agent cannot accept risk on the owner's behalf. A GO recommendation does not itself authorize deployment.
