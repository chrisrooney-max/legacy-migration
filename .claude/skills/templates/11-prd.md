# Discovery Synthesis: [System Name] — Rebuild PRD

**Synthesis Date**: YYYY-MM-DD
**Iteration**: [YYYY-MM-iteration-name]
**Research Period**: [Analysis start date] - [Analysis end date]

---

## Executive Summary

[2-3 sentence summary of what the system does, the key pain points found in the codebase, and the top recommendation for rebuilding it]

---

## Research Overview

**Documents Analysed**: 10 spec documents + migration advisor report
**Codebase Path**: [path analysed]
**Other Sources**: context/migration-inputs.md

### Analysis Coverage

| # | Document | Key Focus |
|---|----------|-----------|
| 1 | system-overview-spec.md | System purpose and capabilities |
| 2 | architecture-spec.md | Components and dependencies |
| 3 | code-structure-spec.md | Repo layout and patterns |
| 4 | data-model-spec.md | Entities and storage |
| 5 | integrations-spec.md | External systems |
| 6 | operations-spec.md | Build and deploy |
| 7 | change-risk-spec.md | Technical debt and coupling |
| 8 | testing-spec.md | Test strategy and gaps |
| 9 | security-spec.md | Auth and compliance |
| 10 | business-rules-spec.md | Core rules and workflows |

---

## Key Themes

### Theme 1: [Theme Name]

**Summary**: [One sentence description]

**Evidence**:
- system-overview-spec.md: "[Finding]"
- architecture-spec.md: "[Finding]"
- change-risk-spec.md: "[Finding]"

**Impact**: [High/Medium/Low]

**User Need**: [What a rebuilt system needs to address based on this theme]

---

### Theme 2: [Theme Name]

**Summary**: [One sentence description]

**Evidence**:
- [spec doc]: "[Finding]"
- [spec doc]: "[Finding]"

**Impact**: [High/Medium/Low]

**User Need**: [What a rebuilt system needs to address based on this theme]

---

## Pain Points (Ranked)

| Rank | Pain Point | Severity | Source Document | Affected Area |
|------|------------|----------|-----------------|---------------|
| 1 | [Pain point] | High | change-risk-spec.md | [Module/area] |
| 2 | [Pain point] | Medium | testing-spec.md | [Module/area] |
| 3 | [Pain point] | Medium | security-spec.md | [Module/area] |

---

## User Needs

### Must Address
1. **[Need]**: [Description] - Evidence: [Source doc + section]
2. **[Need]**: [Description] - Evidence: [Source doc + section]

### Should Address
1. **[Need]**: [Description] - Evidence: [Source doc + section]

### Could Address
1. **[Need]**: [Description] - Evidence: [Source doc + section]

---

## Proposed Features

### Feature 1: [Name]

**User Story**: As a [persona], I want [capability] so that [benefit]

**Addresses**:
- Theme: [Theme name]
- Pain Points: [Pain point numbers]
- User Needs: [Need numbers]

**Estimated Effort**: [S/M/L/XL]

**Priority**: [Critical / High / Medium / Low]

---

### Feature 2: [Name]

**User Story**: As a [persona], I want [capability] so that [benefit]

**Addresses**:
- Theme: [Theme name]
- Pain Points: [Pain point numbers]
- User Needs: [Need numbers]

**Estimated Effort**: [S/M/L/XL]

**Priority**: [Critical / High / Medium / Low]

---

## Cross-Iteration References

**Related Previous Analysis**:
- [Prior run / version]: [Relevant finding or feature]

**Potential Conflicts**:
- [Any conflicts with previous analysis decisions]

---

## Recommendations

### Immediate Actions
1. [Recommendation grounded in analysis findings]
2. [Recommendation grounded in analysis findings]

### Future Considerations
1. [Recommendation for later iterations]

---

## Open Questions

- [ ] [Question requiring further research or stakeholder input]
- [ ] [Question for business validation]

---

## Appendix

### Source Documents
- analysis-output/V{n}/system-overview/system-overview-spec.md
- analysis-output/V{n}/architecture/architecture-spec.md
- analysis-output/V{n}/code-structure/code-structure-spec.md
- analysis-output/V{n}/data-model/data-model-spec.md
- analysis-output/V{n}/integrations/integrations-spec.md
- analysis-output/V{n}/operations/operations-spec.md
- analysis-output/V{n}/change-risk/change-risk-spec.md
- analysis-output/V{n}/testing/testing-spec.md
- analysis-output/V{n}/security/security-spec.md
- analysis-output/V{n}/business-rules/business-rules-spec.md
- final-docs/V{n}/rewrite-vs-refactor.md

### Methodology Notes
[Notes about analysis approach, codebase access limitations, or areas where evidence was thin]
