# Skill: doc_summarize

**Version:** 1.0.0  
**Last Updated:** January 18, 2026  
**Maintained By:** Documentation Team

---

## Purpose

Summarizes technical specifications, RFCs, design documents, and other technical content into structured requirements and key insights suitable for different audiences (executives, engineers, product managers).

---

## Inputs

### Required Inputs

1. **Document Source**
   - File path (local file)
   - URL (web document)
   - Raw text content
   - Supported formats: Markdown, PDF, HTML, plain text

2. **Summary Type**
   - `requirements`: Extract functional and non-functional requirements
   - `executive`: High-level business overview
   - `technical`: Detailed technical summary
   - `comparison`: Compare with another document
   - `action_items`: Extract tasks and decisions

3. **Target Audience**
   - `executive`: Business leaders (minimal jargon)
   - `engineer`: Technical staff (detailed, specific)
   - `product`: Product managers (features, user impact)
   - `mixed`: Combined audience (layered approach)

### Optional Inputs

- **Focus Areas**: Specific sections or topics to emphasize
- **Length Target**: Desired summary length (short/medium/long)
- **Template**: Pre-defined output structure
- **Exclusions**: Sections to skip (e.g., legal disclaimers)

### Input Example

```json
{
  "source": "https://example.com/api-spec.html",
  "summary_type": "requirements",
  "target_audience": "engineer",
  "focus_areas": ["authentication", "rate_limiting", "error_handling"],
  "length": "medium"
}
```

---

## Outputs

### 1. Executive Summary
- 2-3 paragraph overview
- Key points and main conclusions
- Business implications
- Recommended actions

### 2. Functional Requirements List
Structured list of MUST/SHOULD/MAY requirements (RFC 2119 style):

```
FR-001 [MUST]: System shall authenticate users via OAuth 2.0
FR-002 [SHOULD]: API shall support pagination for list endpoints
FR-003 [MAY]: System may cache responses for up to 5 minutes
```

### 3. Non-Functional Requirements
- Performance (latency, throughput, capacity)
- Security (authentication, authorization, encryption)
- Scalability (concurrent users, data volume)
- Reliability (uptime, failover, backup)
- Maintainability (logging, monitoring, documentation)

### 4. Key Dependencies and Constraints
- External system dependencies
- Technology constraints
- Resource limitations
- Timeline constraints
- Compliance requirements

### 5. Open Questions / Ambiguities
- Unclear specifications
- Contradictory statements
- Missing information
- Assumptions that need validation

### Output Example

````markdown
# API Specification Summary

## Executive Summary

The Payment Processing API provides a RESTful interface for handling transaction requests with OAuth 2.0 authentication and rate limiting. The system is designed to handle 1000 requests/second with 99.9% uptime. Key security features include end-to-end encryption and PCI-DSS compliance. Implementation is estimated at 8-12 weeks for MVP.

## Functional Requirements

| ID | Priority | Requirement | Acceptance Criteria |
|----|----------|-------------|---------------------|
| FR-001 | MUST | OAuth 2.0 authentication | All endpoints require valid Bearer token |
| FR-002 | MUST | Process payment transactions | Support credit card, debit card, ACH |
| FR-003 | SHOULD | Idempotency support | Duplicate requests return cached response |
| FR-004 | MAY | Webhook notifications | Optional real-time event delivery |

## Non-Functional Requirements

### Performance
- Response time: < 200ms (p95)
- Throughput: 1000 req/sec sustained
- Concurrent connections: 10,000+

### Security
- TLS 1.3 required
- PCI-DSS Level 1 compliant
- API keys rotated every 90 days
- Rate limiting: 100 req/min per client

### Reliability
- Availability: 99.9% (< 44 min downtime/month)
- Data backup: Every 6 hours
- Disaster recovery: RTO 4 hours, RPO 1 hour

## Key Dependencies

- **PostgreSQL 14+**: Primary database
- **Redis 7+**: Caching and session storage
- **AWS KMS**: Key management for encryption
- **Stripe API**: Payment gateway integration

## Open Questions

1. **Retry Logic**: Specification unclear on retry behavior for failed transactions. Exponential backoff? Max attempts?

2. **Currency Support**: Document mentions "multi-currency" but doesn't specify which currencies. All ISO 4217 codes?

3. **Webhook Ordering**: No mention of event ordering guarantees. Can webhooks arrive out-of-order?

4. **Rate Limit Scope**: Per API key, per IP, or per user account?

## Recommendations

1. **Clarify Open Questions**: Schedule review meeting with API team
2. **Security Review**: Engage security team for threat modeling
3. **Load Testing**: Validate 1000 req/sec throughput claim
4. **Documentation**: Create runbook for operations team
````

---

## Tool Usage

- **read_file**: Load local documents
- **web_fetch**: Retrieve web documents
- **web_search**: Look up unfamiliar terms or standards
- **write_file**: Save summary output

---

## Algorithm

### Step 1: Load Document Content

```python
if source.is_url():
    content = web_fetch(source)
elif source.is_file():
    content = read_file(source)
else:
    content = source  # Raw text

# Parse based on format
if format == 'pdf':
    text = extract_text_from_pdf(content)
elif format == 'html':
    text = extract_text_from_html(content)
else:
    text = content
```

### Step 2: Extract Sections

Identify document structure:

```python
sections = {
    'introduction': extract_section(text, ['intro', 'overview', 'background']),
    'scope': extract_section(text, ['scope', 'objectives', 'goals']),
    'requirements': extract_section(text, ['requirements', 'specifications', 'features']),
    'architecture': extract_section(text, ['architecture', 'design', 'structure']),
    'constraints': extract_section(text, ['constraints', 'limitations', 'assumptions']),
    'security': extract_section(text, ['security', 'authentication', 'authorization']),
    'performance': extract_section(text, ['performance', 'scalability', 'capacity'])
}
```

### Step 3: Identify RFC 2119 Keywords

```python
requirement_keywords = {
    'MUST': 'mandatory',
    'SHALL': 'mandatory',
    'REQUIRED': 'mandatory',
    'SHOULD': 'recommended',
    'RECOMMENDED': 'recommended',
    'MAY': 'optional',
    'OPTIONAL': 'optional',
    'MUST NOT': 'forbidden',
    'SHALL NOT': 'forbidden'
}

requirements = []
for sentence in text.sentences:
    for keyword in requirement_keywords:
        if keyword in sentence.upper():
            requirements.append({
                'text': sentence,
                'priority': requirement_keywords[keyword],
                'keyword': keyword
            })
```

### Step 4: Categorize Requirements

```python
functional_requirements = []
non_functional_requirements = []

for req in requirements:
    if is_functional(req):
        functional_requirements.append(req)
    else:
        # Categorize NFRs
        category = categorize_nfr(req)  # performance, security, reliability, etc.
        non_functional_requirements.append({
            'category': category,
            'requirement': req
        })
```

### Step 5: Extract Numerical Specifications

```python
specifications = extract_patterns(text, [
    r'(\d+)\s*(ms|milliseconds)',  # Latency
    r'(\d+)\s*(req/sec|requests per second)',  # Throughput
    r'(\d+)\s*%\s*(uptime|availability)',  # Reliability
    r'(\d+)\s*(GB|TB|MB)',  # Storage
    r'(\d+)\s*(users|connections)',  # Capacity
])
```

### Step 6: Flag Ambiguities

```python
ambiguities = []

# Check for vague language
vague_terms = ['soon', 'fast', 'large', 'many', 'few', 'should be', 'might']
for sentence in text.sentences:
    if any(term in sentence.lower() for term in vague_terms):
        ambiguities.append({
            'sentence': sentence,
            'issue': 'Vague specification',
            'suggestion': 'Request specific metrics or thresholds'
        })

# Check for contradictions
# (Compare statements from different sections)
contradictions = find_contradictions(text)
ambiguities.extend(contradictions)
```

### Step 7: Generate Summary

```python
summary = generate_summary(
    audience=target_audience,
    summary_type=summary_type,
    length=length_target,
    sections=sections,
    requirements={'functional': functional_requirements, 'nfr': non_functional_requirements},
    specifications=specifications,
    ambiguities=ambiguities
)
```

### Step 8: Create Acceptance Criteria

```python
for req in functional_requirements:
    req['acceptance_criteria'] = generate_acceptance_criteria(req)

# Example:
# Requirement: "System shall authenticate users via OAuth 2.0"
# Acceptance Criteria:
#   - Invalid token returns 401 Unauthorized
#   - Expired token returns 401 with appropriate error message
#   - Valid token grants access to protected resources
#   - Token expiration follows OAuth 2.0 specification
```

---

## Failure Modes & Mitigations

### Failure Mode 1: Unstructured Documents

**Symptom:** Document has no clear sections, requirements embedded in prose

**Impact:** Difficult to extract structured requirements

**Mitigation:**
1. Use NLP techniques to identify requirement-like sentences:
   - Look for modal verbs (must, shall, should, may)
   - Identify imperative statements
   - Find conditional logic (if-then patterns)
2. Extract by topic clustering:
   - Group related sentences
   - Infer logical sections
3. Flag for manual review if confidence is low
4. Ask user to identify key sections

**Implementation:**
```python
if not has_clear_sections(document):
    # Use NLP-based extraction
    requirements = extract_via_nlp(document)
    warn_user("Document lacks clear structure. Extraction confidence: 70%")
    prompt_user("Please review extracted requirements for accuracy")
```

### Failure Mode 2: Ambiguous Specifications

**Symptom:** Requirements use vague language or lack measurable criteria

**Impact:** Cannot create clear acceptance criteria

**Mitigation:**
1. Flag ambiguous statements prominently
2. Suggest specific questions to resolve ambiguity
3. Provide examples of better specifications
4. Don't make assumptions - ask for clarification

**Example:**
```markdown
⚠️ AMBIGUOUS REQUIREMENT: "System should be fast"

Issues:
- "Fast" is subjective and unmeasurable
- No baseline for comparison
- No specific metric defined

Suggested Clarifications:
1. What is the acceptable response time?
   Example: "API responses must be < 200ms at p95"
2. Under what load conditions?
   Example: "With 1000 concurrent users"
3. For which operations?
   Example: "For GET requests to /api/users endpoint"
```

### Failure Mode 3: Very Large Documents

**Symptom:** Document exceeds processing capacity (100+ pages)

**Impact:** Cannot process entire document at once

**Mitigation:**
1. Chunk document into logical sections
2. Process each section independently
3. Merge summaries with deduplication
4. Create hierarchical summary:
   - Top-level: Ultra-brief overview
   - Mid-level: Section summaries
   - Detail-level: Full section analysis

**Implementation:**
```python
if document.page_count > 50:
    sections = chunk_by_headings(document)
    summaries = []
    for section in sections:
        summaries.append(summarize_section(section))
    
    final_summary = merge_summaries(summaries, deduplicate=True)
    create_hierarchical_view(final_summary)
```

### Failure Mode 4: Technical Jargon / Unknown Terms

**Symptom:** Document contains unfamiliar acronyms or domain-specific terminology

**Impact:** May misinterpret requirements

**Mitigation:**
1. Identify unknown terms and acronyms
2. Use web_search to look up definitions
3. Maintain glossary in output
4. Flag terms that couldn't be resolved
5. Ask user for clarification on critical unknowns

**Implementation:**
```python
unknown_terms = identify_acronyms_and_jargon(document)
glossary = {}

for term in unknown_terms:
    definition = web_search(f"{term} definition technical")
    if definition:
        glossary[term] = definition
    else:
        flag_for_user_clarification(term)
```

### Failure Mode 5: Contradictory Requirements

**Symptom:** Different sections specify conflicting behavior

**Impact:** Cannot create coherent requirements list

**Mitigation:**
1. Detect contradictions through semantic analysis
2. Flag prominently with side-by-side comparison
3. Don't choose - present both and ask user
4. Document which sections contain conflicts

**Example:**
```markdown
⚠️ CONTRADICTION DETECTED

Section 2.3: "All API requests must use JSON format"
Section 4.1: "The system shall support XML and JSON formats"

These statements conflict. Please clarify:
1. Is XML support required or not?
2. If XML is supported, is it optional for all endpoints?
3. Should this be documented as an evolution (JSON-only initially, XML added later)?
```

---

## Test Cases

### Test Case 1: API Specification (OpenAPI 3.0)

**Input:**
```yaml
# payment-api-spec.yaml (50 pages, OpenAPI 3.0 format)
openapi: 3.0.0
info:
  title: Payment Processing API
  version: 2.1.0
paths:
  /v1/payments:
    post:
      summary: Create payment
      security:
        - bearerAuth: []
      parameters:
        - name: idempotency-key
          in: header
          required: true
      responses:
        '201':
          description: Payment created
        '401':
          description: Unauthorized
```

**Expected Output:**
```markdown
# Payment Processing API - Requirements Summary

## Executive Summary
RESTful API for payment processing with OAuth 2.0 authentication and idempotency support. Designed for high-volume transaction processing with PCI-DSS compliance.

## Functional Requirements

FR-001 [MUST]: OAuth 2.0 bearer token authentication required for all endpoints
FR-002 [MUST]: Idempotency keys required for all mutating operations
FR-003 [MUST]: Support payment methods: credit card, debit card, ACH
FR-004 [SHOULD]: Return payment status in real-time
FR-005 [MAY]: Support webhook notifications for payment events

## Non-Functional Requirements

### Security
- TLS 1.3 required for all connections
- PCI-DSS Level 1 compliance mandatory
- API keys must be rotated every 90 days

### Performance
- P95 response time: < 200ms
- Throughput: 1000 req/sec sustained
- Peak capacity: 5000 req/sec for 5 minutes

### API Design
- RESTful design following OpenAPI 3.0
- JSON request/response format
- Standard HTTP status codes
- Comprehensive error messages

## Dependencies
- PostgreSQL 14+ for transaction storage
- Redis for idempotency key caching
- Stripe API for payment gateway
- AWS KMS for encryption key management

## Open Questions
1. Token expiration time not specified - default to 1 hour?
2. Webhook retry policy unclear - exponential backoff?
3. Multi-currency support mentioned but currencies not listed
```

### Test Case 2: Architecture RFC (50-page Technical Document)

**Input:**
```markdown
# RFC-2024-001: Microservices Migration Plan

## 1. Introduction
This RFC proposes migrating our monolithic application to microservices architecture...

## 2. Current State
- Monolithic Rails application
- PostgreSQL database
- 500k daily active users
- 2TB of data

## 3. Proposed Architecture
...
(48 more pages of technical details)
```

**Expected Output:**
```markdown
# Microservices Migration - Executive Summary

## Overview
Proposal to migrate from Rails monolith to microservices architecture over 18 months. Estimated cost: $2.4M. Expected benefits: 10x scalability, 50% reduction in deployment time, improved fault isolation.

## Functional Requirements

FR-001 [MUST]: Decompose monolith into 12 domain-bounded services
FR-002 [MUST]: Implement API gateway for request routing
FR-003 [MUST]: Migrate data to service-specific databases
FR-004 [SHOULD]: Support gradual rollout with feature flags
FR-005 [MUST]: Maintain backward compatibility during migration

## Phased Approach

### Phase 1: Foundation (Months 1-3)
- Set up Kubernetes infrastructure
- Implement API gateway and service mesh
- Create CI/CD pipelines

### Phase 2: Extract Services (Months 4-12)
- Extract 4 services per quarter
- Migrate data incrementally
- Dual-run old and new systems

### Phase 3: Decommission (Months 13-18)
- Route all traffic to new services
- Decommission monolith
- Performance tuning and optimization

## Risks & Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Data migration failures | High | Medium | Comprehensive testing, rollback plan |
| Performance degradation | High | Low | Load testing, canary deployments |
| Team capability gap | Medium | High | Training program, external consultants |

## Open Questions

1. **Service Boundaries**: Document uses different decomposition strategies in different sections (domain-driven vs. technical). Which takes precedence?

2. **Data Consistency**: How to handle distributed transactions? Saga pattern mentioned but not detailed.

3. **Timeline**: Is 18 months realistic given team size of 15 engineers?

4. **Cost Breakdown**: $2.4M total but no line-item budget provided.

## Recommendations

1. **Clarify Open Questions**: Schedule architecture review board meeting
2. **Proof of Concept**: Extract one low-risk service first (months 1-2)
3. **Team Training**: Invest in Kubernetes and microservices training
4. **External Review**: Consider architectural consultant for validation
```

---

## Performance Characteristics

- **Processing Time**: 
  - Small docs (< 10 pages): 5-15 seconds
  - Medium docs (10-50 pages): 30-90 seconds
  - Large docs (50+ pages): 2-5 minutes (chunked processing)
- **Memory Usage**: Proportional to document size, ~2MB per page
- **Scalability**: Can process multiple documents in parallel

---

## Usage Examples

### Example 1: CLI Usage

```bash
claude skill/doc_summarize/execute \
  --source api-spec.yaml \
  --type requirements \
  --audience engineer \
  --output requirements.md
```

### Example 2: Programmatic Usage

```python
from skills import doc_summarize

result = doc_summarize.execute(
    source="https://example.com/rfc-123.html",
    summary_type="executive",
    target_audience="mixed",
    focus_areas=["security", "performance"]
)

print(result.executive_summary)
print(result.requirements)
```

### Example 3: Batch Processing

```bash
# Summarize all docs in a directory
for file in docs/*.md; do
  claude skill/doc_summarize/execute \
    --source "$file" \
    --type requirements \
    --audience engineer \
    --output "summaries/$(basename $file)"
done
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-18 | Initial release |

---

**End of Skill Specification**