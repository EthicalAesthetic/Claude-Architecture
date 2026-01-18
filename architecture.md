# Claude Architecture & Agent Creation Assignment

**Course/Module:** AI Systems & Agentic Workflows  
**Student:** [Your Name]  
**Date:** January 18, 2026

---

## Table of Contents

1. [Task A: The Full Stack Architecture](#task-a-the-full-stack-architecture)
2. [Task D: Claude Engine Coordination](#task-d-claude-engine-coordination)
3. [Diagrams](#diagrams)

---

## Task A: The Full Stack Architecture

### Overview

Claude-based workflows are structured as a layered system where each layer has distinct responsibilities and interfaces. This architecture enables modularity, reusability, and clear separation of concerns.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                           USER                                  │
│              (Provides Intent & Constraints)                    │
│  • Sets goals and priorities                                    │
│  • Provides context and data                                    │
│  • Defines acceptance criteria                                  │
│  • Gives feedback and refinement                                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      .claude.md                                 │
│               (Global Rules & Guidelines)                       │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ • Tone & Style Rules      • Reasoning Constraints        │  │
│  │ • Tool Usage Policy       • Safety Guardrails            │  │
│  │ • Output Format Specs     • Decision Frameworks          │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CLAUDE ENGINE                                │
│              (Orchestration & Coordination)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   PLANNER    │  │    ROUTER    │  │  VALIDATOR   │          │
│  │ • Task       │  │ • Skill      │  │ • Output     │          │
│  │   decomp     │  │   selection  │  │   checking   │          │
│  │ • Sequencing │  │ • Tool       │  │ • Quality    │          │
│  │ • Dependencies│ │   matching   │  │   assurance  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└──────────┬─────────────────┬────────────────────────────────────┘
           │                 │
           ▼                 ▼
┌──────────────────┐  ┌─────────────────────────────────────────┐
│    skill/        │  │              TOOLS                      │
│  (Modules)       │  │  ┌──────────────┐  ┌──────────────┐    │
│                  │  │  │ File System  │  │    Shell     │    │
│ • triage         │  │  └──────────────┘  └──────────────┘    │
│ • doc_summarize  │  │  ┌──────────────┐  ┌──────────────┐    │
│ • audit_crypto   │  │  │  Web Search  │  │   API Calls  │    │
│ • code_review    │  │  └──────────────┘  └──────────────┘    │
│ • report_gen     │  │  ┌──────────────┐  ┌──────────────┐    │
│                  │  │  │   Database   │  │   Artifacts  │    │
└──────────────────┘  │  └──────────────┘  └──────────────┘    │
                      └─────────────────────────────────────────┘
```

### Layer 1: .claude.md - "How to Think"

**Purpose:** Defines the global operating principles, constraints, and behaviors for Claude across all interactions.

**Contains:**
- **Tone and Style Rules:** Communication patterns, formality level, vocabulary preferences
- **Reasoning Constraints:** How to handle uncertainty, when to ask vs. assume, verification requirements
- **Tool Usage Policy:** Which tools are allowed, under what conditions, safety checks
- **Safety Guardrails:** Security boundaries, data protection, compliance requirements
- **Output Format Requirements:** Standard structures for different content types

**Example Rules:**
```
- "Always verify assumptions before proceeding with destructive operations"
- "Use concise, technical language for engineering audiences"
- "Prefer checklists for multi-step procedures"
- "Never commit secrets to files"
- "Include Summary + Action Items in all reports"
```

**Why It Matters:**
- Ensures consistent behavior across different tasks and contexts
- Acts as the "constitution" for Claude's decision-making
- Reduces need for repetitive instructions in individual prompts
- Enforces organizational standards and best practices

**Data Flow:**
- User request → .claude.md filters/shapes interpretation → Claude Engine processes

---

### Layer 2: skill/ - Specialized Capabilities

**Purpose:** Reusable, modular capabilities that encapsulate specific workflows, templates, and tool usage patterns.

**What a Skill Is:**
A skill is a self-contained module with:
- Clear inputs and outputs
- Defined algorithm/procedure
- Tool dependencies
- Error handling strategies
- Test cases for validation

**What It Stores:**
- **Templates:** Standard formats for outputs (report templates, config templates)
- **Procedures:** Step-by-step workflows (incident response procedure, code review checklist)
- **Tool Recipes:** Pre-configured tool usage patterns (shell command sequences, API call chains)
- **Domain Knowledge:** Specialized rules (security compliance checks, performance optimization patterns)

**Example Skills:**
```
skill/triage/
  ├── spec.md          # Skill specification
  ├── algorithm.py     # Core logic (if needed)
  ├── templates/       # Output templates
  └── tests/           # Test cases

skill/doc_summarize/
  ├── spec.md
  ├── extraction_rules.md
  └── templates/

skill/audit_crypto/
  ├── spec.md
  ├── patterns.json    # Known crypto patterns to detect
  └── compliance.md    # Standards reference
```

**Benefits:**
- **Reusability:** Skills can be composed into larger workflows
- **Maintainability:** Update skill logic in one place, affects all uses
- **Testability:** Each skill has defined test cases
- **Discoverability:** Catalog of available capabilities

**Data Flow:**
- Claude Engine identifies needed capability → Routes to appropriate skill → Skill executes with tools → Returns structured output

---

### Layer 3: Claude Engine - Who Executes

**Purpose:** The core orchestration system that interprets intent, plans execution, coordinates resources, and ensures quality.

**Three Key Roles:**

#### 1. Planner (Task Decomposition)
**Responsibilities:**
- Break complex requests into manageable subtasks
- Identify dependencies and ordering constraints
- Estimate effort and resources needed
- Create execution timeline

**Example:**
```
Request: "Analyze security vulnerabilities in our codebase"

Plan:
1. Scan code for common patterns (XSS, SQL injection, etc.)
2. Check dependencies for known CVEs
3. Review authentication/authorization logic
4. Test input validation
5. Generate prioritized findings report
```

#### 2. Router (Skill/Tool Selection)
**Responsibilities:**
- Match task requirements to available skills/tools
- Select optimal execution path
- Handle capability gaps
- Manage tool permissions and access

**Decision Logic:**
```
IF task_type == "code_analysis" AND language == "python":
    USE skill/code_review + tool/read_file
ELIF task_type == "data_collection":
    USE tool/web_search + tool/web_fetch
ELIF task_type == "report_generation":
    USE skill/report_gen + tool/write_file
```

#### 3. Validator (Output Checking)
**Responsibilities:**
- Verify outputs meet quality standards
- Check completeness and accuracy
- Validate against requirements
- Flag issues for correction

**Validation Checklist:**
- ✓ All required sections present?
- ✓ Data accurate and sourced?
- ✓ Format matches specification?
- ✓ No security issues (secrets exposed)?
- ✓ Meets acceptance criteria?

**Handling Ambiguity:**

**When to Make Assumptions:**
- Standard industry practices
- Common default values
- Well-established conventions

**When to Ask Questions:**
- Unclear scope or requirements
- Multiple valid interpretations
- High-risk decisions
- User-specific preferences needed

**Multi-Step Workflow Management:**

**State Tracking:**
- Maintains context across steps
- Records decisions and their rationale
- Preserves intermediate results
- Enables rollback if needed

**Checkpoints:**
- Validation points between major steps
- User confirmation for critical operations
- Progress reporting
- Error recovery points

---

### Layer 4: Tools - Hands

**Purpose:** External capabilities that enable Claude to interact with systems, data, and environments.

**Available Tools:**

| Tool | Purpose | Example Usage | Risks |
|------|---------|---------------|-------|
| `read_file` | Access file contents | Load config, analyze code | Exposure of secrets |
| `write_file` | Create/modify files | Save reports, update configs | Overwriting important data |
| `shell` | Execute commands | Run builds, tests, scans | System damage, security holes |
| `web_search` | Find information | CVE lookup, documentation | Misinformation, bias |
| `web_fetch` | Retrieve web content | Download reports, APIs | Malicious content |
| `artifacts` | Create rich outputs | Interactive visualizations | User confusion |

**How Tools Change What's Possible:**

**Without Tools:**
- Pure reasoning and text generation
- Limited to knowledge cutoff
- Cannot interact with environment
- No verification of current state

**With Tools:**
- Can access current data and systems
- Verify claims against reality
- Automate complex workflows
- Create and modify artifacts

**Risks and Mitigation:**

**Risk: Destructive Operations**
- **Mitigation:** Confirmation prompts, dry-run mode, backup verification

**Risk: Security Exposure**
- **Mitigation:** Secret scanning, access controls, audit logging

**Risk: Unintended Side Effects**
- **Mitigation:** Sandboxing, rollback capabilities, explicit scoping

**Risk: Tool Misuse**
- **Mitigation:** Permission system, rate limiting, human oversight

**Data Flow:**
- Claude Engine decides tool is needed → Validates tool permission → Executes tool call → Processes result → Validates output

---

### Layer 5: User - Director

**Purpose:** Provides strategic direction, context, constraints, and feedback to guide the system.

**User Responsibilities:**

**1. Define Intent**
- Clear goal statement
- Success criteria
- Priorities and tradeoffs
- Timeline and constraints

**2. Provide Context**
- Background information
- Relevant history
- Domain knowledge
- Organizational context

**3. Set Constraints**
- Resource limits (time, compute, cost)
- Quality requirements
- Compliance needs
- Risk tolerance

**4. Give Feedback**
- Validate outputs
- Correct misunderstandings
- Refine requirements
- Approve critical actions

**Steering Execution:**

**Continuous Alignment Loop:**
```
User Request → Claude Interprets → Proposes Plan → User Validates
     ↑                                                    ↓
     ←───────────── Feedback/Refinement ←────────────────┘
```

**Example Interaction:**
```
User: "Improve our API security"

Claude: "I'll focus on:
1. Authentication mechanisms
2. Input validation
3. Rate limiting
4. Data encryption

Does this match your priorities? Any specific concerns?"

User: "Yes, but prioritize #2 and #3 - we've had injection attempts"

Claude: "Understood. I'll start with input validation and rate limiting.
Should I also check the existing WAF rules?"

User: "Yes, and compare against OWASP Top 10"

Claude: [Proceeds with refined understanding]
```

**Data Flow:**
- User provides high-level intent → System interprets and plans → Proposes approach → User validates/refines → System executes → User evaluates results

---

## Task D: Claude Engine Coordination

### Workflow Example: Security Incident Analysis

This trace demonstrates how the Claude Engine coordinates skills, tools, and validation across an 8-step workflow.

```
════════════════════════════════════════════════════════════════
STEP 1: RECEIVE
════════════════════════════════════════════════════════════════
User Request: "Analyze our Q3 security incidents and create a 
               remediation plan"

Engine State:
  - Intent: Analysis + Planning
  - Scope: Q3 timeframe
  - Output: Remediation plan
  - Constraints: Unknown (will ask if needed)

════════════════════════════════════════════════════════════════
STEP 2: PLAN (Planner Component)
════════════════════════════════════════════════════════════════
Task Decomposition:
  1. Load Q3 incident data
  2. Parse and validate data structure
  3. Analyze patterns and categorize
  4. Prioritize by risk score
  5. Generate remediation recommendations
  6. Create timeline with dependencies
  7. Validate completeness

Dependencies Identified:
  - Step 1 must complete before Step 2
  - Steps 3-4 can run in parallel after Step 2
  - Step 5 requires Step 4 completion
  - Step 6-7 are sequential

Estimated Time: 3-5 minutes

════════════════════════════════════════════════════════════════
STEP 3: ROUTE (Router Component)
════════════════════════════════════════════════════════════════
Task: Load incident data
Selected: tool/read_file
Reason: Need to access stored incident logs

Task: Analyze patterns
Selected: skill/triage
Reason: Specialized capability for incident prioritization

Task: Generate plan
Selected: skill/report_gen
Reason: Structured output creation

════════════════════════════════════════════════════════════════
STEP 4: EXECUTE (Tool Calls)
════════════════════════════════════════════════════════════════
[tool/read_file] Reading: ./data/incidents_q3_2025.json

Result:
  - File size: 156 KB
  - Records found: 23 incidents
  - Date range: 2025-07-01 to 2025-09-30
  - Format: Valid JSON

Parsed Data Preview:
  {
    "incident_001": {
      "date": "2025-07-15",
      "type": "brute_force",
      "severity": "high",
      "affected_systems": ["auth_server_01"],
      "status": "resolved"
    },
    ...
  }

════════════════════════════════════════════════════════════════
STEP 5: VALIDATE (Validator Component)
════════════════════════════════════════════════════════════════
Data Completeness Check:
  ✓ All incidents have date field
  ✓ All incidents have type field
  ✗ 2 incidents missing severity field
  ✓ All incidents have affected_systems
  ✓ All incidents have status

Issues Found:
  - incident_007: severity = null
  - incident_019: severity = null

Decision: ASK USER
Reason: Cannot accurately prioritize without severity data

Question to User:
"I found 2 incidents (007, 019) without severity ratings:
- 007: Unauthorized API access attempt (blocked by WAF)
- 019: Suspicious file upload (quarantined)

How should I classify these? (Critical/High/Medium/Low)"

[User Response: "007 is Medium, 019 is High"]

Updated Data → Proceed

════════════════════════════════════════════════════════════════
STEP 6: ANALYZE (Skill Execution)
════════════════════════════════════════════════════════════════
[skill/triage] Processing 23 incidents...

Categorization:
  - Brute Force Attacks: 8 incidents
  - SQL Injection Attempts: 3 incidents
  - XSS Attempts: 5 incidents
  - Malware Detection: 2 incidents
  - Data Exfiltration: 2 incidents
  - Misconfiguration: 3 incidents

Risk Scoring (CVSS × Business Impact × Exploitability):
  Critical (P0): 5 incidents
    - incident_003: SQL injection in payment API (9.8)
    - incident_012: Data exfiltration attempt (9.2)
    - incident_015: RCE vulnerability exploited (9.6)
    - incident_018: Privilege escalation (9.1)
    - incident_021: Credential theft (9.4)
  
  High (P1): 8 incidents
  Medium (P2): 7 incidents
  Low (P3): 3 incidents

Systemic Issues Identified:
  1. Authentication weaknesses (10 incidents related)
  2. Input validation gaps (8 incidents related)
  3. Outdated dependencies (5 incidents related)

════════════════════════════════════════════════════════════════
STEP 7: GENERATE (Report Creation)
════════════════════════════════════════════════════════════════
[skill/report_gen] Creating remediation plan...

Output Structure:
  1. Executive Summary
  2. Incident Overview (stats, categories)
  3. Critical Findings (P0 items)
  4. Remediation Plan (prioritized, with timelines)
  5. Systemic Recommendations
  6. Resource Requirements
  7. Success Metrics

Remediation Timeline Generated:
  - P0 incidents: 24-48 hours (5 items)
  - P1 incidents: 1-2 weeks (8 items)
  - P2 incidents: 1 month (7 items)
  - P3 incidents: Backlog (3 items)

Dependencies Mapped:
  - Authentication fixes must precede access reviews
  - Input validation requires code review completion
  - Dependency updates need compatibility testing

Owners Assigned:
  - P0: Security team (immediate response)
  - P1: Dev team + Security (coordinated)
  - P2: Dev team (sprint planning)
  - P3: Tech debt backlog

════════════════════════════════════════════════════════════════
STEP 8: VALIDATE (Quality Check)
════════════════════════════════════════════════════════════════
[Validator] Checking output against quality checklist...

Required Sections:
  ✓ Summary (3 paragraphs, covers key points)
  ✓ Action Items (23 items, all prioritized)
  ✓ Timeline (realistic, dependency-aware)
  ✓ Risk Assessment (included for each P0/P1)
  ✓ Dependencies (mapped and documented)
  ✓ Resource Requirements (team allocation)
  ✓ Success Metrics (measurable criteria)

Data Quality:
  ✓ All incidents accounted for
  ✓ Risk scores properly calculated
  ✓ No missing critical information
  ✓ Sources cited (incident IDs)

Format Check:
  ✓ Markdown formatting correct
  ✓ Tables properly structured
  ✓ No security issues (no secrets exposed)
  ✓ Appropriate length (12 pages)

RESULT: PASS - Ready for delivery

════════════════════════════════════════════════════════════════
FINAL OUTPUT DELIVERED TO USER
════════════════════════════════════════════════════════════════
Total Execution Time: 4 minutes 23 seconds
Tools Used: read_file (1), web_search (3 - for CVE lookups)
Skills Used: triage, report_gen
Validation Checkpoints: 2 (data validation, output validation)
User Interactions: 1 (severity clarification)
```

### Key Coordination Patterns Demonstrated

1. **Decomposition:** Complex request broken into 7 manageable steps
2. **Routing:** Appropriate skills/tools selected for each step
3. **Validation:** Two checkpoints caught issues early
4. **Ambiguity Handling:** Asked user instead of assuming severity
5. **State Management:** Maintained context across all steps (incident data, classifications, dependencies)
6. **Quality Assurance:** Final validation ensured completeness

---

## Diagrams

### Diagram 1: Full Stack Architecture
*(See ASCII diagram at top of document)*

### Diagram 2: Data Flow Sequence

```
┌─────┐                                                     ┌─────┐
│USER │<-______________________________________________________ │USER │                                                       |
└──┬──┘                                                       | └──▲──┘                                                       |
   │                                                           │
   │ 1. Request: "Analyze incidents"                          │
   │                                                           │
   ▼                                                           │
┌──────────────┐                                              │
│  .claude.md  │  2. Apply global rules                       │
│ (Filter req) │     (tone, safety, format)                   │
└──────┬───────┘                                              │
       │                                                       │
       ▼                                                       │
┌─────────────────────────────────────────┐                   │
│          CLAUDE ENGINE                  │                   │
│  ┌─────────────────────────────────┐   │                   │
│  │ 3. PLANNER: Decompose into      │   │                   │
│  │    subtasks (load, analyze,     │   │                   │
│  │    prioritize, generate)        │   │                   │
│  └─────────────────────────────────┘   │                   │
│  ┌─────────────────────────────────┐   │                   │
│  │ 4. ROUTER: Select skills/tools  │   │                   │
│  │    (triage + read_file)         │   │                   │
│  └─────────────────────────────────┘   │                   │
└──────┬──────────────────┬───────────────┘                   │
       │                  │                                   │
       │ 5. Execute       │ 5. Execute                        │
       ▼                  ▼                                   │
┌──────────┐      ┌──────────────┐                            │
│  skill/  │      │    TOOLS     │                            │
│  triage  │      │  read_file   │                            │
└────┬─────┘      └──────┬───────┘                            │
     │                   │                                    │
     │ 6. Results        │ 6. Data                            │
     │                   │                                    │
     └────────┬──────────┘                                    │
              │                                               │
              ▼                                               │
┌─────────────────────────────────────────┐                   │
│          CLAUDE ENGINE                  │                   │
│  ┌─────────────────────────────────┐    │                   │
│  │ 7. VALIDATOR: Check quality     │    │                   │
│  │    - Completeness ✓             │   │                   │
│  │    - Format ✓                   │   │                   │
│  │    - Accuracy ✓                 │   │                   │
│  └─────────────────────────────────┘    │                   │
└──────────────────────┬──────────────────┘                   │
                       │                                      │
                       │ 8. Final Report                      │
                       └──────────────────────────────────────┘
```

### Diagram 3: Decision Flow (Ambiguity Handling)

```
                    ┌──────────────┐
                    │ Receive Task │
                    └──────┬───────┘
                           │
                           ▼
                 ┌─────────────────┐
                 │ All info clear? │
                 └────────┬────────┘
                          │
               ┌──────────┴──────────┐
               │                     │
            NO │                     │ YES
               │                     │
               ▼                     ▼
    ┌──────────────────┐   ┌─────────────────┐
    │ Is it critical?  │   │ Proceed with    │
    └────────┬─────────┘   │ execution       │
             │              └─────────────────┘
      ┌──────┴──────┐
      │             │
   YES│             │NO
      │             │
      ▼             ▼
┌──────────┐  ┌──────────────┐
│ ASK USER │  │ Use standard │
│          │  │ assumption   │
└────┬─────┘  │ + document   │
     │        └──────┬───────┘
     │               │
     └───────┬───────┘
             │
             ▼
    ┌────────────────┐
    │ Proceed with   │
    │ execution      │
    └────────────────┘
```

---

## Conclusion

The Claude architecture provides a robust framework for building agentic AI systems through:

1. **Clear separation of concerns** across five distinct layers
2. **Reusable capabilities** through the skill system
3. **Intelligent orchestration** via the Claude Engine
4. **Safe tool usage** with appropriate guardrails
5. **User-centric design** that maintains alignment with intent

This architecture enables complex workflows while maintaining clarity, safety, and maintainability.
