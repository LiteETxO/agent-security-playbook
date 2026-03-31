# Agent Security Playbook

Landing page and digital product files for [safeagent.build](https://safeagent.build).

A practitioner-grade security toolkit for operators running AI agents. Covers the OWASP LLM Top 10 threats in plain English with copy-paste templates, checklists, and runbooks.

---

## Site Structure

| File | Description |
|------|-------------|
| `index.html` | Main landing/sales page |
| `download.html` | Post-purchase download page (Full Toolkit — $197) |
| `download-core.html` | Post-purchase download page (Core PDF — $97) |
| `CNAME` | Custom domain config for GitHub Pages |

---

## Product Files

All product content lives in `/product/`. The zip (`/product/agent-security-playbook.zip`) is generated from these source files.

### The Playbook

| File | Description |
|------|-------------|
| `product/playbook.md` | Full 40-page Agent Security Playbook covering 7 threats, quick start, glossary, and resources |

### System Prompt Templates (`product/templates/`)

Ready-to-deploy system prompt security templates. Copy, customize placeholders, paste into your agent.

| File | Use Case |
|------|----------|
| `email-agent-security.md` | Email agents — injection guards, data handling rules, permission boundaries |
| `research-agent-security.md` | Research/browsing agents — indirect injection from web, data leakage prevention |
| `support-agent-security.md` | Customer support agents — social engineering, credential fishing, impersonation |
| `file-agent-security.md` | File system agents — path traversal, exfiltration, unauthorized writes |

### Checklists (`product/checklists/`)

| File | Description |
|------|-------------|
| `preflight-security-checklist.md` | 25-point pre-launch checklist (permissions, secrets, input validation, output review, monitoring, incident readiness) |

### Runbooks (`product/runbooks/`)

| File | Description |
|------|-------------|
| `incident-response-runbook.md` | Step-by-step incident response: detection, containment, assessment, recovery, post-incident review, notification guidance |

### Red Team Tests (`product/red-team/`)

| File | Description |
|------|-------------|
| `test-suite.md` | 5 adversarial test scenarios: direct injection, indirect injection via email, credential extraction, excessive agency, data exfiltration |

---

## Pricing

| Tier | Price | Stripe Link | Download Page |
|------|-------|-------------|---------------|
| Core PDF | $97 | See `index.html` | `download-core.html` |
| Full Toolkit | $197 | See `index.html` | `download.html` |

---

## Deployment

Deployed via GitHub Pages with custom domain `safeagent.build`.

To update the product zip, regenerate `product/agent-security-playbook.zip` from the `/product/` directory contents and commit.

---

## Contact

support@atxo.me
