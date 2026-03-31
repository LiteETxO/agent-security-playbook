# Agent Security Incident Response Runbook
### Step-by-Step Guide for When Something Goes Wrong

**Version 1.0 | ATXO Labs Agent Security Playbook**

---

## How to Use This Runbook

This runbook provides step-by-step guidance for responding to a suspected or confirmed AI agent security incident. An "incident" is any event where an agent has behaved in a way that may have compromised data, executed unauthorized actions, or been manipulated by an attacker.

**Read this before you need it.** Designate an Incident Commander role before you deploy your agents. Run a tabletop exercise using a fictional scenario. The worst time to read this document is during an actual incident.

**Keep this document accessible.** It should be reachable even if your main systems are down. Print a copy. Store it somewhere outside your primary infrastructure.

---

## Part 1: Detecting a Compromised Agent

### Signals That Something May Be Wrong

Not every anomaly is a security incident, but the following signals warrant immediate investigation:

**Behavioral anomalies:**
- Agent takes actions not observed in normal operation (new email destinations, unexpected API calls, unusual query patterns)
- Agent produces outputs inconsistent with its task (includes unexpected data, produces unusually long responses, outputs structured data it wasn't asked for)
- Agent reports receiving "special instructions" or claims its operational parameters have changed
- Agent refuses normal requests it previously handled
- Agent contacts or attempts to contact external URLs not on its authorized whitelist

**Quantitative anomalies:**
- Action volume significantly above baseline (e.g., 10x normal email sends, 100x normal database queries)
- Data transfer volume significantly above baseline
- Session duration far above normal
- Error rate spike (often indicates the agent is attempting actions it's not authorized to take)

**Explicit indicators:**
- Log entries showing attempts to access unauthorized files or paths
- Rejected authentication attempts to systems the agent doesn't normally access
- Email sends to addresses not in the authorized destination list
- Webhook calls to unrecognized URLs
- Any output containing what appears to be credential material, system prompt contents, or bulk data

**External reports:**
- A customer reports receiving an unusual email from your support agent
- A third-party service reports unusual API usage from your credentials
- Your team notices unexpected data in external systems
- A security researcher contacts you about a potential issue

### Triage: Real Incident vs. False Alarm

Before escalating fully, do a 15-minute triage:

1. **Identify the signal.** What specifically triggered concern? Is there a log entry, a report, an anomalous output?

2. **Check for a benign explanation.** Was there a system change, a new input type, or a legitimate increase in volume? Can the behavior be explained without invoking an attack?

3. **Check for corroboration.** A single anomalous data point may be noise. Multiple corroborating signals (unusual volume + unusual destination + unusual content) indicate a real incident.

4. **Apply the "if real" test.** If this signal represents a real attack, how bad could the damage be? If the answer is "very bad," err on the side of treating it as a real incident. Containment is cheap. Cleanup is expensive.

If you cannot rule out a real incident in 15 minutes, escalate.

---

## Part 2: Immediate Containment

**Timeline target: Containment within 15 minutes of incident confirmation.**

Speed matters. Every minute an agent remains active after compromise is another minute of potential damage.

### Step 1: Kill the Agent (Immediately)

Stop the agent from taking further actions. Depending on your deployment:

**Cloud-hosted agents:**
- Scale the agent deployment to 0 instances
- Disable the scheduler/trigger that invokes the agent
- Kill any running processes

**API-based agents:**
- Revoke the LLM API key the agent uses (this stops new LLM calls)
- Disable the webhook or cron job that feeds input to the agent

**Document what you did and when.** Start a running incident log immediately: every action taken, by whom, at what time.

### Step 2: Revoke Credentials

Immediately revoke every credential the compromised agent was using:

- [ ] LLM provider API key
- [ ] Database credentials
- [ ] Email service credentials
- [ ] CRM/SaaS API tokens
- [ ] File storage access credentials
- [ ] Any OAuth tokens granted to the agent
- [ ] Webhook secrets

Do not wait to assess whether credentials were actually extracted. Revoke first, assess later. Credential rotation is fast. Credential abuse is not.

After revoking, issue new credentials for any services that need to remain operational (other agents, human operators). Do not issue new credentials to the compromised agent until the incident is resolved.

### Step 3: Preserve Evidence

Before taking any cleanup actions, preserve evidence:

- [ ] Export all logs for the agent (LLM API logs, tool call logs, application logs, infrastructure logs)
- [ ] Take snapshots of any data that the agent had access to
- [ ] Export any output files or data the agent generated
- [ ] Screenshot or export the agent's current system prompt and configuration
- [ ] Preserve the current state of any databases or files the agent could access
- [ ] Note any external destinations the agent may have contacted

Store this evidence in a location separate from your main infrastructure. Do not modify it. Chain of custody matters if this becomes a legal or regulatory matter.

### Step 4: Scope Containment to Adjacent Systems

Consider what the compromised agent had access to and whether other systems need containment:

- Were credentials shared between this agent and other agents or systems? Revoke those too.
- Did the agent have access to systems that now need their own audit?
- Could the agent have injected malicious content into data stores that other systems read? (e.g., written to a shared database or document store that other agents or applications query)

If the agent could have contaminated downstream systems, those systems need to be audited before resuming normal operation.

---

## Part 3: Damage Assessment

Once the agent is contained, assess what happened.

### Reconstruct the Timeline

Build a complete timeline of the incident:

1. **When did the agent start behaving anomalously?** Review logs to find the earliest evidence of unexpected behavior. This may be before the behavior was noticed.

2. **What inputs triggered the behavior?** Find the email, document, web page, or other input that contained the injection or trigger. What did it say? How was it delivered? Who sent it?

3. **What actions did the agent take?** List every action in chronological order:
   - Every tool call
   - Every database query
   - Every email sent
   - Every file read or written
   - Every external request

4. **What data could the agent have accessed?** Even if you can't prove the agent exfiltrated data, list everything it *could have* accessed given its permissions and the timing.

5. **What data did the agent actually output or send?** Review all outputs. What was in them? Where did they go?

### Data Access Assessment Matrix

For each system the agent had access to, complete this table:

| System | Data Type | Could Agent Access? | Evidence of Access | Volume Potentially Exposed |
|--------|-----------|--------------------|--------------------|---------------------------|
| Customer DB | Names, emails, orders | Yes | [Log evidence] | [Row count] |
| Email system | Internal emails | Yes/No | | |
| File storage | [What files?] | Yes/No | | |
| [Other] | | | | |

### Output Destination Review

Review every destination the agent could have sent data to:
- Email addresses that received messages
- External URLs that received HTTP requests
- Files that were written
- APIs that were called

For each destination, determine: was this destination authorized? Is there evidence of unusual volumes or content?

---

## Part 4: Recovery Steps

### Before Restoring Operations

Do not restore the agent to production until you have:

1. **Identified the attack vector.** What was the injection or compromise method? If you haven't found it, the agent will be re-compromised immediately upon restoration.

2. **Addressed the vulnerability.** Fixed the system prompt, added validation, restricted permissions, or made whatever change prevents the same attack.

3. **Issued new credentials.** All credentials revoked during containment should be replaced with new ones. The old ones are permanently retired.

4. **Audited and cleaned up any contaminated data.** If the agent may have written malicious content to a data store, audit and clean that store before re-reading it with other agents.

5. **Tested the fix.** Run the Red Team Test Suite against the updated agent in a staging environment. Confirm the attack that succeeded no longer succeeds.

6. **Reviewed blast radius.** Confirmed that no other agents or systems were compromised.

### Restoration Sequence

1. Issue new credentials with the tightest possible scope
2. Deploy the updated agent to staging
3. Run adversarial tests in staging
4. Run normal operation tests in staging
5. Deploy to production with elevated monitoring
6. Monitor closely for 48 hours post-restoration

---

## Part 5: Post-Incident Review Template

Complete this within 5 business days of incident closure.

### Incident Summary

**Incident ID:** _______________
**Date/Time Discovered:** _______________
**Date/Time Contained:** _______________
**Date/Time Resolved:** _______________
**Incident Commander:** _______________

### What Happened

*2-3 paragraph plain-English description of the incident: what the attacker did, how the agent was compromised, what the agent did as a result.*

### Root Cause

*The specific vulnerability that was exploited. Be precise: "The agent had no injection guards in its system prompt and was given access to the full customer database with no row-level filtering, allowing the injection to exfiltrate up to [N] customer records."*

### Contributing Factors

*What made the vulnerability exploitable? Examples: excessive permissions, missing validation, no monitoring, credentials in system prompt.*

### Impact Assessment

| Category | Details |
|----------|---------|
| Data accessed | |
| Data exfiltrated (confirmed) | |
| Data exfiltrated (possible) | |
| Unauthorized actions taken | |
| Systems affected | |
| Estimated affected records/users | |

### Timeline of Events

| Time | Event |
|------|-------|
| | |

### Response Actions Taken

| Time | Action | By Whom |
|------|--------|---------|
| | | |

### What Went Well

*What aspects of the response worked? What controls limited the damage?*

### What Went Poorly

*What slowed the response? What gaps did the incident reveal?*

### Corrective Actions

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| | | | |

### Prevention Measures

*Specific changes made to prevent recurrence: system prompt updates, permission restrictions, new monitoring, architectural changes.*

---

## Part 6: Who to Notify and When

### Internal Notifications

| Who | When to Notify | How | What to Tell Them |
|-----|---------------|-----|------------------|
| Immediate team | Immediately upon confirmation | Slack/call | Brief: what happened, what's been done, what you need |
| Engineering lead | Within 30 minutes | Direct message/call | Technical details, containment status |
| Executive team | Within 2 hours if significant | Call or in-person | Business impact, customer notification needs |
| Legal/compliance | Within 2 hours if data may be involved | Email + call | What data, how much, where it may have gone |

### Customer Notifications

You may be legally required to notify affected customers. The specifics depend on your jurisdiction, the type of data involved, and your agreements with customers.

**General guidance:**
- Personal data of EU residents: GDPR requires notifying the supervisory authority within 72 hours and affected individuals without undue delay if there is high risk to their rights and freedoms.
- Personal data of California residents: CCPA/CPRA has notification requirements for data breaches.
- Healthcare data: HIPAA breach notification rules apply.
- Payment card data: PCI DSS has specific incident notification requirements.

**Work with legal counsel.** Notification obligations are fact-specific and jurisdiction-specific. Don't make notification decisions without legal review.

**When in doubt, notify.** The cost of unnecessary notification (explaining to customers that their data may have been exposed and what you're doing about it) is far lower than the cost of non-notification if you had a legal obligation.

### Regulatory Notifications

If applicable:
- [ ] Contact your legal counsel
- [ ] Identify applicable regulatory frameworks
- [ ] Submit required regulatory notifications within required timeframes
- [ ] Document all notifications made

### Third-Party Provider Notifications

If the incident involved a third-party service (LLM provider, cloud service, SaaS tool):
- Notify them — they may have security resources and incident context that helps
- Check if the incident violated their terms of service
- Determine if the incident originated from their platform or affected their other customers

---

## Appendix: Quick Reference — First 15 Minutes

```
INCIDENT DETECTED

Minute 1-3:   Confirm it's real (or triage to false alarm)
              Start incident log

Minute 3-5:   Kill the agent (scale to 0 / disable trigger)
              Notify Incident Commander

Minute 5-10:  Revoke ALL agent credentials
              Preserve all logs (export before any changes)

Minute 10-15: Assess what adjacent systems are at risk
              Scope containment to those systems if needed
              Notify leadership if significant

YOU ARE NOW IN CONTROL. BREATHE. INVESTIGATE.
```

---

*This runbook is maintained by ATXO Labs. For questions: support@atxo.me*
