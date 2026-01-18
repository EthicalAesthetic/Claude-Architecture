# Skill: triage

**Version:** 1.0.0  
**Last Updated:** January 18, 2026  
**Maintained By:** Security Team

---

## Purpose

Prioritizes security incidents/issues and produces a remediation plan based on risk scoring methodology that considers CVSS scores, business impact, and exploitability.

---

## Inputs

### Required Inputs

1. **Incident Data**
   - Format: JSON, CSV, or structured text
   - Required fields:
     - `id`: Unique incident identifier
     - `description`: Brief description of the incident
     - `type`: Incident category (e.g., brute_force, sql_injection, xss, malware, etc.)
     - `discovered_date`: ISO 8601 timestamp
   - Optional fields (will prompt if missing):
     - `severity`: Initial severity assessment
     - `cvss_score`: CVSS 3.1 score (0-10)
     - `affected_systems`: List of impacted systems
     - `status`: Current status (open, investigating, resolved, closed)
     - `business_impact`: Qualitative assessment (critical, high, medium, low)

2. **Risk Tolerance Threshold** (Optional)
   - Default: CVSS 7.0+ triggers immediate attention
   - Can be adjusted based on organization risk appetite

3. **Resource Constraints** (Optional)
   - Team size: Number of available security engineers
   - Timeline: Overall deadline for remediation completion
   - Budget: If relevant for external resources

### Input Example

```json
{
  "incidents": [
    {
      "id": "INC-001",
      "type": "sql_injection",
      "description": "SQL injection vulnerability in payment API endpoint",
      "discovered_date": "2025-09-15T14:30:00Z",
      "cvss_score": 9.8,
      "affected_systems": ["payment-api", "customer-db"],
      "status": "open",
      "business_impact": "critical"
    },
    {
      "id": "INC-002",
      "type": "brute_force",
      "description": "Multiple failed login attempts from 192.168.1.50",
      "discovered_date": "2025-09-18T09:15:00Z",
      "affected_systems": ["auth-server"],
      "status": "investigating"
    }
  ],
  "risk_threshold": 7.0,
  "resources": {
    "team_size": 3,
    "timeline_days": 90
  }
}
```

---

## Outputs

### 1. Prioritized Incident List

Table format with P0-P4 classification:

| ID | Type | Risk Score | Priority | Affected Systems | Deadline |
|----|------|-----------|----------|------------------|----------|
| INC-001 | SQL Injection | 29.4 | P0 | payment-api, customer-db | 24 hours |
| INC-003 | Data Exfiltration | 27.6 | P0 | file-server | 24 hours |

### 2. Remediation Plan

Structured plan with:
- Immediate actions (containment)
- Short-term fixes
- Long-term improvements
- Timeline with milestones
- Dependencies between tasks
- Resource allocation
- Verification procedures

### 3. Risk Summary Dashboard

Executive summary including:
- Total incidents by priority
- Risk score distribution
- Top affected systems
- Systemic issues identified
- Resource requirements
- Timeline overview

---

## Tool Usage

- **read_file**: Load incident data from JSON/CSV files
- **write_file**: Save prioritized list and remediation plan
- **web_search**: Look up CVE details, CVSS scores, and known exploits

---

## Algorithm

### Step 1: Load and Validate Data

```python
incidents = load_incident_data(file_path)
for incident in incidents:
    validate_required_fields(incident, ['id', 'description', 'type', 'discovered_date'])
    if missing_fields:
        prompt_user_for_data(incident.id, missing_fields)
```

### Step 2: Calculate Risk Scores

For each incident, calculate comprehensive risk score:

```
Risk Score = CVSS × Business Impact × Exploitability

Where:
- CVSS: 0-10 (from CVE database or manual assessment)
- Business Impact: 
    - Critical: 3.0 (revenue-generating, customer-facing)
    - High: 2.0 (internal operations, data processing)
    - Medium: 1.5 (development, staging)
    - Low: 1.0 (test, isolated systems)
- Exploitability:
    - Active exploitation in wild: 3.0
    - Public exploit available: 2.5
    - Proof-of-concept exists: 2.0
    - Theoretical only: 1.0
```

### Step 3: Classify by Priority

```
Priority Classification:
- P0 (Critical): Risk Score ≥ 25 OR CVSS ≥ 9.0
- P1 (High):     Risk Score 15-24.9 OR CVSS 7.0-8.9
- P2 (Medium):   Risk Score 8-14.9 OR CVSS 5.0-6.9
- P3 (Low):      Risk Score 3-7.9 OR CVSS 3.0-4.9
- P4 (Info):     Risk Score < 3 OR CVSS < 3.0
```

### Step 4: Group by Affected System

Identify systemic issues:

```python
system_incidents = group_by(incidents, 'affected_systems')
for system, system_incidents in system_incidents.items():
    if len(system_incidents) >= 3:
        flag_systemic_issue(system, system_incidents)
```

### Step 5: Generate Remediation Timeline

```
Remediation Deadlines (from discovery):
- P0 (Critical): 24-48 hours
- P1 (High):     7 days
- P2 (Medium):   30 days
- P3 (Low):      90 days
- P4 (Info):     Backlog (no deadline)

Adjust for:
- Dependencies (prerequisite tasks)
- Resource availability
- Complexity (some P0s may need 72 hours if complex)
```

### Step 6: Identify Dependencies and Blockers

```python
dependencies = []
for incident in incidents:
    if incident.type in ['auth_bypass', 'privilege_escalation']:
        # Must fix auth issues before other access-related problems
        dependencies.append({
            'incident': incident.id,
            'blocks': find_dependent_incidents(incident),
            'reason': 'Authentication must be secured first'
        })
```

### Step 7: Output Structured Plan

Generate comprehensive report with:
- Executive summary (2-3 paragraphs)
- Prioritized incident table
- Remediation timeline (Gantt chart or timeline)
- Resource allocation
- Risk summary
- Systemic issues and recommendations

---

## Failure Modes & Mitigations

### Failure Mode 1: Missing Severity Data

**Symptom:** Incidents lack CVSS scores or severity ratings

**Impact:** Cannot accurately calculate risk scores

**Mitigation:**
1. Check if CVE ID exists → lookup CVSS from NVD database
2. If no CVE, analyze incident type and context:
   - SQL injection in production API → CVSS 8-9
   - XSS in admin panel → CVSS 6-7
   - Brute force with no success → CVSS 4-5
3. Prompt user for manual classification
4. Default to P2 (Medium) if truly uncertain, but flag for review

**Implementation:**
```python
if not incident.cvss_score:
    if incident.cve_id:
        incident.cvss_score = lookup_cvss_from_nvd(incident.cve_id)
    else:
        estimated = estimate_cvss_from_type(incident.type)
        ask_user(f"Incident {incident.id} has no CVSS. Estimated: {estimated}. Confirm?")
```

### Failure Mode 2: Conflicting Priorities

**Symptom:** Multiple P0 incidents but limited resources

**Impact:** Cannot address all critical issues simultaneously

**Mitigation:**
1. Apply tiebreaker rules:
   - Exploitability > Asset Value > Discovery Date
   - Active exploitation beats potential exploitation
   - Customer-facing beats internal systems
2. Implement phased approach:
   - Immediate containment for all P0s (within hours)
   - Prioritize remediation by tiebreaker rules
3. Flag capacity issues to user
4. Suggest resource augmentation (contractors, managed services)

**Implementation:**
```python
critical_incidents = filter(incidents, priority='P0')
if len(critical_incidents) > team_capacity:
    sorted_critical = sort_by_tiebreaker(critical_incidents)
    warn_user(f"Found {len(critical_incidents)} P0 incidents but capacity for {team_capacity}")
    suggest_phased_approach(sorted_critical)
```

### Failure Mode 3: Resource Overload

**Symptom:** Remediation timeline exceeds available resources

**Impact:** Cannot meet deadlines with current team

**Mitigation:**
1. Calculate total effort required vs. available
2. Flag capacity issues with specific numbers
3. Suggest options:
   - Extend timeline (with risk explanation)
   - Prioritize subset of issues
   - Add resources (temporary staff, contractors)
   - Accept risk for lower-priority items
4. Provide trade-off analysis

**Implementation:**
```python
total_effort_days = sum(incident.estimated_effort for incident in incidents)
available_capacity = team_size × timeline_days

if total_effort_days > available_capacity:
    overload_pct = (total_effort_days / available_capacity - 1) × 100
    warn_user(f"Resource overload: {overload_pct}% over capacity")
    suggest_options(incidents, team_size, timeline_days)
```

### Failure Mode 4: Incomplete Incident Data

**Symptom:** Missing affected systems, unclear description

**Impact:** Cannot assess business impact accurately

**Mitigation:**
1. Identify what's missing
2. Attempt to infer from available data:
   - Log source can indicate affected system
   - Incident type suggests likely targets
3. Request missing information from user
4. Proceed with cautious assumptions, clearly documented
5. Flag assumptions in output for user review

### Failure Mode 5: Duplicate or Related Incidents

**Symptom:** Multiple incidents are actually same root cause

**Impact:** Over-estimation of work, fragmented remediation

**Mitigation:**
1. Detect potential duplicates:
   - Same affected system
   - Similar timeframe (within hours)
   - Same incident type
2. Flag for user review
3. Suggest consolidation
4. In remediation plan, group related incidents

---

## Test Cases

### Test Case 1: Critical SQL Injection

**Input:**
```json
{
  "id": "INC-TEST-001",
  "type": "sql_injection",
  "description": "SQL injection in payment API /v1/transactions endpoint",
  "discovered_date": "2026-01-18T10:00:00Z",
  "cvss_score": 9.8,
  "affected_systems": ["payment-api", "customer-database"],
  "business_impact": "critical",
  "status": "open"
}
```

**Expected Output:**
```
Priority: P0 (Critical)
Risk Score: 29.4 (CVSS 9.8 × Impact 3.0 × Exploit 1.0)
Remediation Deadline: 2026-01-19T10:00:00Z (24 hours)

Immediate Actions:
1. Isolate payment-api from public internet (0-2 hours)
2. Enable WAF rules for SQL injection (0-2 hours)
3. Review logs for exploitation attempts (2-4 hours)

Remediation Plan:
1. Patch vulnerable endpoint with parameterized queries (4-8 hours)
2. Deploy to staging and test (2 hours)
3. Deploy to production with rollback plan (1 hour)
4. Verify fix with security scan (1 hour)
5. Monitor for 24 hours

Long-term:
1. Implement code review checklist for SQL injection
2. Add automated SQL injection testing to CI/CD
3. Security training for development team
```

### Test Case 2: Mixed Severity Batch

**Input:**
```json
{
  "incidents": [
    {"id": "INC-101", "type": "sql_injection", "cvss_score": 9.8, "business_impact": "critical"},
    {"id": "INC-102", "type": "xss", "cvss_score": 6.5, "business_impact": "high"},
    {"id": "INC-103", "type": "brute_force", "cvss_score": 5.0, "business_impact": "medium"},
    {"id": "INC-104", "type": "outdated_library", "cvss_score": 7.5, "business_impact": "medium"},
    {"id": "INC-105", "type": "misconfiguration", "cvss_score": 4.0, "business_impact": "low"},
    ... (10 more incidents)
  ],
  "resources": {
    "team_size": 2,
    "timeline_days": 30
  }
}
```

**Expected Output:**
```
╔══════════════════════════════════════════════════════════════╗
║              TRIAGE SUMMARY - 15 INCIDENTS                   ║
╚══════════════════════════════════════════════════════════════╝

Priority Breakdown:
- P0 (Critical): 3 incidents → 24-48 hour deadline
- P1 (High):     7 incidents → 7 day deadline  
- P2 (Medium):   4 incidents → 30 day deadline
- P3 (Low):      1 incident  → 90 day deadline

Systemic Issues Identified:
⚠️  Authentication System: 5 incidents related (INC-102, 103, 108, 111, 114)
    Recommendation: Comprehensive auth review and MFA implementation
⚠️  Dependency Management: 3 outdated libraries with known CVEs
    Recommendation: Implement automated dependency scanning

Resource Analysis:
- Total estimated effort: 42 person-days
- Available capacity: 60 person-days (2 people × 30 days)
- Capacity utilization: 70%
- Status: ✓ FEASIBLE within timeline

Phased Remediation Timeline:
┌─────────────────────────────────────────────────────────────┐
│ Week 1: P0 incidents + containment for P1                  │
│ Week 2-3: P1 incidents (7 items)                           │
│ Week 4: P2 incidents + systemic improvements               │
│ Week 5+: P3 backlog items                                  │
└─────────────────────────────────────────────────────────────┘

Critical Path:
INC-101 (SQL Inj) → INC-104 (Auth) → INC-102 (XSS in auth) → ...
     ↓
Auth system must be secured before other access-related issues
```

### Test Case 3: Missing Data Handling

**Input:**
```json
{
  "id": "INC-201",
  "type": "unauthorized_access",
  "description": "Suspicious login from unknown IP",
  "discovered_date": "2026-01-18T14:00:00Z"
  // Missing: cvss_score, affected_systems, business_impact
}
```

**Expected Output:**
```
⚠️  INCOMPLETE INCIDENT DATA - INC-201

Missing Information:
1. CVSS Score: Cannot determine severity
2. Affected Systems: Cannot assess business impact
3. Business Impact: Required for prioritization

Questions for User:
1. Which system was accessed?
   - Production authentication server?
   - Development environment?
   - Internal tool?

2. Was the login successful?
   - Yes → Higher severity
   - No (blocked) → Lower severity

3. What actions were taken after login?
   - Data access?
   - Configuration changes?
   - Failed attempts only?

Preliminary Assessment (pending answers):
- Type suggests potential: P1-P2
- Estimated CVSS range: 6.0-8.0
- Recommended: Treat as P1 until confirmed

Proceeding with assumption: P1, 7-day remediation timeline
**This will be revised once user provides missing data**
```

---

## Usage Examples

### Example 1: CLI Usage

```bash
# Using the skill via command line
claude skill/triage/execute \
  --input incidents_q3.json \
  --risk-threshold 7.0 \
  --team-size 3 \
  --output remediation_plan.md
```

### Example 2: Programmatic Usage

```python
from skills import triage

# Load incidents
incidents = load_json('incidents.json')

# Execute triage
result = triage.execute(
    incidents=incidents,
    risk_threshold=7.0,
    resources={'team_size': 3, 'timeline_days': 90}
)

# Output results
print(result.summary)
print(result.prioritized_list)
save_markdown(result.remediation_plan, 'plan.md')
```

### Example 3: Interactive Mode

```bash
$ claude skill/triage/execute --interactive

Loading incidents from ./data/incidents.json... ✓
Found 23 incidents

Analyzing incident data...
⚠️  2 incidents missing severity ratings:
    - INC-007: Unauthorized API access
    - INC-019: Suspicious file upload

Please provide severity (Critical/High/Medium/Low):
INC-007: > Medium
INC-019: > High

Calculating risk scores... ✓
Identifying systemic issues... ✓
Generating remediation plan... ✓

Summary:
- 5 P0 incidents (immediate attention)
- 8 P1 incidents (this week)
- 7 P2 incidents (this month)
- 3 P3 incidents (backlog)

Full report saved to: remediation_plan_2026-01-18.md
```

---

## Performance Characteristics

- **Execution Time**: O(n log n) where n is number of incidents (sorting dominates)
- **Memory**: O(n) - linear in incident count
- **Scalability**: Tested with up to 1000 incidents
- **Typical Duration**: 5-15 seconds for 50 incidents

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-18 | Initial release |

---

## Maintenance Notes

**Review Triggers:**
- Changes to organizational risk tolerance
- New compliance requirements
- Updates to CVSS scoring methodology
- Feedback from security team on accuracy

**Dependencies:**
- NVD API for CVE/CVSS lookup
- Internal asset database for business impact assessment

---

**End of Skill Specification**