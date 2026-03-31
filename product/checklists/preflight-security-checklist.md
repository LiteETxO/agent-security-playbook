# Agent Pre-Flight Security Checklist
### 25-Point Review Before Going Live

**Version 1.0 | ATXO Labs Agent Security Playbook**

Run this checklist before deploying any AI agent to production. Every item should be a ✅ before launch. Any ❌ is a blocker.

---

## Category 1: Permissions & Access Control

- [ ] **1. Principle of least privilege applied.** The agent has been granted only the specific permissions required for its defined tasks — not a superset, not admin-level access for convenience. If you can't explain *why* the agent needs each permission, remove it.

- [ ] **2. Read-only credentials used where possible.** Any data source the agent only needs to read has been given read-only credentials, not read-write. Agents that read-write by default are an unnecessary risk.

- [ ] **3. Dedicated credentials created for this agent.** The agent does not share credentials with human operators, other agents, or other services. It has its own scoped API keys, service account, or role that can be revoked independently.

- [ ] **4. Credential rotation schedule is set.** Agent credentials are scheduled for automatic or manual rotation on a defined interval (at minimum quarterly, ideally monthly). The rotation process has been tested.

- [ ] **5. External outbound access is scoped.** If the agent makes outbound calls (webhooks, emails, HTTP requests), it can only contact a pre-approved whitelist of destinations. It cannot make arbitrary outbound requests unless that is its explicit function.

- [ ] **6. Action allowlist is defined.** You have an explicit list of what this agent is permitted to do. Anything not on the list is blocked by default — not just discouraged in the system prompt.

---

## Category 2: Secrets Handling

- [ ] **7. No secrets in the system prompt.** The system prompt contains no API keys, passwords, connection strings, tokens, or any credential. These are injected at runtime through environment variables or a secrets manager.

- [ ] **8. A secrets manager is in use.** Credentials are stored in a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler, 1Password Secrets, etc.) — not in code, config files, environment files checked into source control, or chat messages.

- [ ] **9. Logs are configured to redact sensitive fields.** Logging infrastructure is explicitly configured to redact field names like `password`, `token`, `key`, `secret`, and to redact strings matching common API key patterns. The full system prompt is not logged in production.

- [ ] **10. Tool responses are sanitized before entering context.** Tool call responses (API responses, database query results) are reviewed for inadvertent credential inclusion. Any response that might contain tokens, keys, or signed URLs is stripped before being passed to the LLM context.

---

## Category 3: Input Validation

- [ ] **11. System prompt includes injection guards.** The system prompt explicitly tells the agent that external content is data, not instructions, and instructs it to recognize and refuse prompt injection attempts.

- [ ] **12. Untrusted content is clearly labeled.** Inputs from external sources (emails, web pages, user-supplied documents) are explicitly framed as data in the context (e.g., wrapped in `<email_content>` tags with an instruction that this is data, not instructions).

- [ ] **13. Input size limits are enforced.** There are limits on the size of inputs the agent will process (maximum email length, maximum document size, maximum number of records per query). Oversized inputs are rejected or truncated, not silently processed in full.

- [ ] **14. File type restrictions are enforced (if applicable).** If the agent processes files, only permitted file types are accepted. Executable file types are explicitly blocked.

- [ ] **15. Path traversal protection is in place (if applicable).** If the agent handles file paths, paths containing `..`, null bytes, or other traversal patterns are rejected before being used.

---

## Category 4: Output Review

- [ ] **16. Outputs are validated before execution.** Before any agent output is acted upon (email sent, database record written, command executed), it passes through a validation step. Anomalous outputs are flagged for human review rather than executed automatically.

- [ ] **17. PII detection is in place.** A scan or review step checks agent outputs for unexpected personal data (names, emails, phone numbers, payment card details) before outputs are sent or stored. Unexpected PII triggers a flag.

- [ ] **18. Human approval gates are defined for high-stakes actions.** The list of actions requiring human approval before execution is documented and implemented. This includes at minimum: sending external emails, deleting data, and financial transactions.

- [ ] **19. Output schema validation is implemented.** If the agent is expected to produce structured output, outputs are validated against a schema. Outputs that don't conform to the expected schema are rejected.

---

## Category 5: Monitoring & Observability

- [ ] **20. Every tool call is logged.** Each time the agent calls an external tool (sends email, queries database, browses web, writes file), a log entry is created with: tool name, parameters, timestamp, session ID. These logs are retained for at least [YOUR RETENTION PERIOD: e.g., 90 days].

- [ ] **21. Anomaly alerting is configured.** Alerts are set up for: unusual action volume (more than N actions in a session), unusual destinations (new email address or webhook URL), failed authentication attempts, high error rates.

- [ ] **22. A baseline of normal behavior is established.** You have documented what "normal" looks like for this agent: typical action count per session, typical data volumes, typical destinations. Deviations from this baseline trigger review.

---

## Category 6: Incident Readiness

- [ ] **23. A kill switch is documented and tested.** You know exactly how to stop this agent immediately: which credentials to revoke, which processes to kill, which webhooks to disable. The steps are written down. The process has been rehearsed at least once.

- [ ] **24. The incident response runbook is ready.** The team has reviewed the incident response runbook (see `/product/runbooks/incident-response-runbook.md`) and understands who is responsible for each step if this agent is compromised.

- [ ] **25. The agent has been adversarially tested.** Before launch, the agent has been tested with at least the five scenarios in the Red Team Test Suite (`/product/red-team/test-suite.md`). Any failed tests have been addressed. Test results are documented.

---

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Operator | | | |
| Security Review | | | |
| Technical Lead | | | |

**Agent name/ID:** _______________
**Deployment date:** _______________
**Next security review date:** _______________

---

*This checklist should be re-run whenever the agent's capabilities, data access, or system prompt change significantly — not just at initial deployment.*

*Questions? support@atxo.me*
