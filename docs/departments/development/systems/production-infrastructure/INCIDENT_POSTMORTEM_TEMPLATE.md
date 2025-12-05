# Incident Postmortem Template

**Purpose**: Use this template for documenting incidents and investigations. Keep it concise but comprehensive.

---

## Incident Information

**Date/Time**: [When did the incident occur?]
**Duration**: [How long did the incident last?]
**Severity**: [P0 - Critical | P1 - High | P2 - Medium | P3 - Low]
**Status**: [Resolved | In Progress | Monitoring]
**Author**: [Who wrote this postmortem?]
**Reviewers**: [Who reviewed this?]

---

## Executive Summary

[2-3 sentence overview of what happened, the impact, and the resolution]

**Impact**:
- [Number of users affected]
- [Services impacted]
- [Data loss/corruption: Yes/No]
- [Financial impact if applicable]

**Root Cause**: [One sentence description of what caused the incident]

---

## Timeline

**All times in [timezone]**

| Time | Event | Action Taken |
|------|-------|--------------|
| HH:MM | [What happened] | [What was done] |
| HH:MM | [Detection/Alert] | [Who was notified] |
| HH:MM | [Investigation began] | [What was checked] |
| HH:MM | [Root cause identified] | [What was found] |
| HH:MM | [Fix implemented] | [What was changed] |
| HH:MM | [Incident resolved] | [Service restored] |
| HH:MM | [Monitoring period] | [Verified stability] |

**Total Time to Detect**: [Time from incident start to detection]
**Total Time to Resolve**: [Time from detection to resolution]
**Total Impact Duration**: [Total time users were affected]

---

## What Happened

### Background
[Provide context about the system/feature involved. What was the normal state before the incident?]

### Incident Description
[Detailed description of what went wrong. Include symptoms, error messages, and observable behavior.]

### Detection
[How was the incident discovered? Alert? User report? Monitoring?]

---

## Investigation

### Initial Hypothesis
[What did we initially think was wrong?]

### Investigation Steps
1. [First thing checked]
   - **Finding**: [What was discovered]
   - **Conclusion**: [What this told us]

2. [Second thing checked]
   - **Finding**: [What was discovered]
   - **Conclusion**: [What this told us]

3. [Continue as needed...]

### Key Evidence
- [Log entries, metrics, screenshots, or other evidence that helped identify the issue]
- [Include specific error messages, stack traces, or data points]

### Root Cause Analysis
[Detailed explanation of the actual root cause. Use the "5 Whys" technique if helpful:]

1. **Why did [symptom] happen?** Because [reason]
2. **Why did [reason] happen?** Because [deeper reason]
3. **Why did [deeper reason] happen?** Because [root cause]

**Final Root Cause**: [Clear statement of the fundamental issue]

---

## Resolution

### Immediate Fix
[What was done to restore service immediately?]

```bash
# Include relevant commands or code changes
[example command or configuration]
```

### Verification
[How did we verify the fix worked?]
- [Metric/test 1]
- [Metric/test 2]
- [User verification]

### Rollback Plan
[If the fix didn't work, what was the rollback plan? Was it needed?]

---

## Prevention & Follow-up

### Permanent Fix
[What needs to be done to permanently solve this? Is the immediate fix sufficient?]

**Status**: [Completed | Scheduled | Pending]
**Timeline**: [When will this be done?]
**Owner**: [Who is responsible?]

### Action Items

| Action | Owner | Priority | Due Date | Status |
|--------|-------|----------|----------|--------|
| [Specific task to prevent recurrence] | [Name] | [P0-P3] | [Date] | [Done/In Progress/Pending] |
| [Improve monitoring/alerting] | [Name] | [P0-P3] | [Date] | [Done/In Progress/Pending] |
| [Update documentation] | [Name] | [P0-P3] | [Date] | [Done/In Progress/Pending] |
| [Add tests/validation] | [Name] | [P0-P3] | [Date] | [Done/In Progress/Pending] |

---

## Lessons Learned

### What Went Well
- [Things that worked during incident response]
- [Effective tools, processes, or communication]

### What Could Be Improved
- [Detection could have been faster if...]
- [Response could have been better if...]
- [Communication could be improved by...]

### Key Takeaways
1. [Important lesson learned]
2. [Important lesson learned]
3. [Important lesson learned]

---

## Template Usage Notes

### When to Write a Postmortem
- **Always**: P0 (Critical) and P1 (High) incidents
- **Usually**: P2 (Medium) incidents that affected users
- **Sometimes**: P3 (Low) incidents if there are important lessons
- **Always**: Any incident that could have been prevented with better processes

### Tips for Writing Good Postmortems
1. **Be blameless**: Focus on systems and processes, not individuals
2. **Be specific**: Include exact times, commands, and error messages
3. **Be honest**: Document mistakes and what went wrong
4. **Be actionable**: Every "What Could Be Improved" should have an action item
5. **Be concise**: Aim for 2-3 pages maximum
6. **Write quickly**: Complete within 2-3 days of incident resolution

### Severity Definitions

| Level | Definition | Example | Response Time |
|-------|------------|---------|---------------|
| **P0 - Critical** | Complete service outage or data loss | "Website down for all users" | Immediate (5 min) |
| **P1 - High** | Major functionality broken, significant user impact | "Login broken, payments failing" | 30 minutes |
| **P2 - Medium** | Partial functionality degraded, some users affected | "Search slow, some pages timing out" | 4 hours |
| **P3 - Low** | Minor issue, minimal user impact | "Typo on page, deprecated warning in logs" | Next business day |

### Review Process
1. **Author** writes first draft within 24-48 hours
2. **Technical reviewer** validates technical accuracy
3. **Team** reviews action items in next team meeting
4. **Manager** approves and archives
5. **Follow-up** check action items after 30 days

---

**Delete this section when creating actual postmortem**
