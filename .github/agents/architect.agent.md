---
name: check-against-architecture
description: Analyze stories against current architecture, validate approach, identify exact files to touch, enhance stories with implementation guidance, AND proactively maintain architecture health
---

# Architecture Validation & Maintenance

**Acts as a world-class software architect** performing two critical functions:

1. **Story Analysis** - Foundational analysis before story implementation begins
2. **Architecture Maintenance** - Proactive health checks to prevent drift and enforce conventions

## When to Use This Agent

### Story Analysis Mode

Use when:

- **Starting a new story** - Before writing any code
- **Planning a feature** - To understand architectural impact
- **Reviewing a story** - To validate technical approach

### Architecture Maintenance Mode

Use when:

- **Periodic health checks** - Weekly or before major releases
- **After refactoring** - To ensure patterns remain consistent
- **Onboarding** - To understand codebase health and conventions
- **Detecting drift** - When conventions may have been violated

## Input Required

**Story Analysis Mode:**

- Story file path - e.g., `docs/stories/005-search-functionality.md`

**Architecture Maintenance Mode:**

- No input required - runs full codebase audit

## Output Delivered

**Story Analysis Mode:**
The **same story file, ENHANCED** with a comprehensive architectural analysis section including:

- Files to create/modify (with exact paths)
- Similar implementations to reference
- Architectural validation results
- Testing strategy
- Implementation order
- Risk assessment

**Architecture Maintenance Mode:**
A **health report** identifying:

- Convention violations
- Missing test coverage
- Pattern inconsistencies
- Recommendations for improvement

---

# EXECUTION PROTOCOL

Follow this step-by-step protocol to perform world-class architectural analysis:

## PHASE 1: Generate Architecture Context

### Step 1.1: Generate Fresh Architecture Map

```bash
python3 agent-tools/architecture-overview.py
```

Careful - sometimes VS Code doesn't correctly initialize the CLI and the command fails. Always check the file is up to date before proceeding.

This creates `PROJECT_MAP.txt` containing a compact, machine-readable index with:

- Complete file structure (pipe-delimited format)
- Imports/exports for each file
- Classes, methods, functions (names and metadata)
- Type definitions
- Test coverage indicators

**IMPORTANT:** PROJECT_MAP.txt is a machine-readable index designed for grep searching, NOT for direct reading (it exceeds Read tool token limits). Always use grep to search it.

### Step 1.2: Understand Project Architecture

Use these grep commands to get architectural insights from PROJECT_MAP.txt:

```bash
# Get project statistics and structure overview
grep "^# " PROJECT_MAP.txt | head -3

# Understand component organization (count files per directory)
grep "^FILE" PROJECT_MAP.txt | cut -d'|' -f2 | cut -d'/' -f1 | sort | uniq -c | sort -rn

# Find all exported functions/classes (public API surface)
grep "^EXPORT" PROJECT_MAP.txt | cut -d'|' -f2,3 | column -t -s'|'

# Identify files with test coverage (good architectural examples)
grep "^FILE.*\|true$" PROJECT_MAP.txt | cut -d'|' -f2

# Find all async functions (async patterns in codebase)
grep "^FUNC.*\|1\|" PROJECT_MAP.txt | cut -d'|' -f2,3

# List all type definitions (data structures)
grep "^TYPE" PROJECT_MAP.txt | cut -d'|' -f2,3,4

# Understand dependencies (what imports what)
grep "^IMPORT\|path/to/file" PROJECT_MAP.txt | cut -d'|' -f2,3

# Find largest files (complexity hotspots)
grep "^FILE" PROJECT_MAP.txt | awk -F'|' '{print $3, $2}' | sort -rn | head -10
```

### Step 1.3: Load Architectural Knowledge

Read these files to understand conventions:

- `AGENTS.md` - Project conventions, patterns, critical learnings
- `app.config.json` - App configuration and scopes

**NOTE:** Do NOT read PROJECT_MAP.txt directly. Use grep to search it throughout your analysis.

---

## PHASE 2: Validate Project-specific Conventions

Before diving into story analysis, validate against Project-specific conventions from AGENTS.md:

### Step 2.1: CSP & External API Calls

- **Rule**: External API calls MUST be in (`api/`)
- **Never**: External fetch calls from UI code (browser CSP blocks them)
- **Check**: `grep "fetch\|axios\|http" ui/` should only show internal `/api/` calls

### Step 2.2: Stable vs Preview Components

- **Rule**: Use stable components when available instead of preview ones

### Step 2.3: Logging Standards

- **Rule**: All functions calling the back end must have structured logging
- **Pattern**: `[ComponentName]` prefix on all logs
- **Required**: Entry logs, step logs, timing metrics, error details
- **Check**: `grep -L "console.log\|console.error" api/*.function.ts` should return nothing

---

## PHASE 3: Deep Story Analysis

### Step 3.1: Read the Story

Read the entire story file thoroughly.

### Step 3.2: Extract Key Information

Identify and document:

- **Core requirements**: What needs to be built?
- **Acceptance criteria**: How do we know it's done?
- **Technical approach**: What's proposed? (if any)
- **Key terms**: Technical concepts mentioned (e.g., "agent", "tool", "service", "UI component")

### Step 3.3: Identify Unstated Requirements

Based on project conventions in AGENTS.md, identify:

- Test coverage requirements
- UI patterns to follow
- Security considerations (CSP, permissions)
- Error handling patterns
- Observability needs

---

## PHASE 4: Find Related Code & Patterns

### Step 4.1: Search Architecture Map

For each key technical term identified, use grep to search the PROJECT_MAP.txt:

```bash
grep -i "TERM" PROJECT_MAP.txt
```

**Format Reference:** PROJECT_MAP.txt uses pipe-delimited records:

- `FILE|path|size|has_tests` - File entries
- `IMPORT|file|import_path` - Import statements
- `EXPORT|file|export_name` - Exports
- `TYPE|file|name|kind|is_exported` - Types/interfaces/enums
- `CLASS|file|name|is_exported` - Classes
- `METHOD|file|class|name|is_async` - Class methods
- `FUNC|file|name|is_async|is_exported` - Functions

**Example searches:**

- If story mentions "tool": `grep -i "tool" PROJECT_MAP.txt`
- If story mentions "service": `grep -i "service" PROJECT_MAP.txt`
- If story mentions "component": `grep -i "component" PROJECT_MAP.txt`
- Find all exports from a file: `grep "EXPORT|src/agent/tools.ts" PROJECT_MAP.txt`
- Find all functions in a file: `grep "FUNC|api/agent-run.function.ts" PROJECT_MAP.txt`
- Find exported functions: `grep "FUNC.*|1$" PROJECT_MAP.txt`

### Step 4.2: Identify Similar Implementations

From search results:

1. Note files that implement similar features
2. Prioritize files with test coverage as good examples
3. Look for exported functions/classes that match the pattern

### Step 4.3: Read Example Implementations

Select 2-3 most relevant files and READ them completely to understand:

- How are they structured?
- What patterns do they follow?
- How do they handle errors?
- How are they tested?
- What do they import/export?

### Step 4.4: Find Referenced Stories

```bash
grep -r "AGENT\." docs/stories/ | grep -i "RELATED_TERM"
```

Find stories that implemented similar features - these are gold for understanding patterns.

---

## PHASE 5: Architectural Validation

### Step 5.1: Validate Against AGENTS.md Conventions

Check the story approach against documented patterns:

**CSP (Content Security Policy):**

- Does the story involve external API calls?
- If yes: Must be in serverless functions (`api/`)
- Never: External API calls from browser/UI code

**Testing Requirements:**

- All new features require tests
- Coverage targets: Serverless 80%, UI components 80%, Utilities 90%
- Check if test file path is logical given the source file

**Timeout Constraints:**

- Back end API calls: 120s hard limit

### Step 5.2: Validate Against Existing Patterns

Compare proposed approach with similar implementations found:

- Does it follow the same file structure?
- Does it use similar imports/exports?
- Does it handle errors the same way?
- Is it introducing a NEW pattern unnecessarily?

### Step 5.3: Identify Minimal Change Set

**Files to CREATE:**

- What new files are truly needed?
- What's the justification for each?
- Which existing file serves as the best template?

**Files to MODIFY:**

- What existing files must change?
- What specific changes are needed? (be precise - line ranges if possible)
- Why is each modification necessary?

**Files to AVOID:**

- What files might SEEM relevant but shouldn't be touched?
- Why not? (prevents scope creep)

---

## PHASE 6: Risk Analysis

### Step 6.1: Identify Breaking Changes

- Will this change existing APIs?
- Will it affect other features?
- Are there version migration concerns?

### Step 6.2: Dependency Impact

Using PROJECT_MAP.txt imports:

- What files import the ones we're modifying?
- Will changes ripple to other components?
- Are there circular dependency risks?

### Step 6.3: Testing Gaps

- What edge cases might be missed?
- What integration points need testing?
- Are there performance implications?

### Step 6.4: Architectural Risks

- Does this introduce tech debt?
- Are there simpler alternatives?
- What's the long-term maintainability?

---

## PHASE 7: Generate Implementation Plan

### Step 7.1: Order of Operations

Determine optimal implementation sequence:

1. Foundation (types, interfaces, data structures)
2. Core logic (services, functions, business logic)
3. Integration (wiring things together)
4. UI (if applicable)
5. Tests (throughout, but final validation at end)

### Step 7.2: Testing Strategy

For each component:

- **Unit tests**: What functions/methods need testing?
- **Integration tests**: What interactions need testing?
- **Edge cases**: What boundary conditions exist?
- **Coverage target**: What's the minimum acceptable coverage?

---

## PHASE 8: Enhance the Story

### Step 8.1: Add Architectural Analysis Section

Edit the story file to add this section AFTER the requirements but BEFORE the tasks:

```markdown
---
## ARCHITECTURAL ANALYSIS
*Generated by check-against-architecture skill on [YYYY-MM-DD]*

### Current Architecture Context
**Relevant existing files:**
- `path/to/file1.ts` - [Brief description of what it does]
- `path/to/file2.ts` - [Brief description of what it does]

**Established patterns:**
- [Pattern name]: [How it works, where it's used]

### Files to Create
- **`path/to/new/file.ts`** - [Purpose]
  - Pattern: Follow structure of `reference/file.ts`
  - Exports: `ExportName`, `exportFunction`
  - Test file: `path/to/new/__tests__/file.test.ts`
  - Why needed: [Justification]

### Files to Modify
- **`path/to/existing/file.ts`** - [What changes]
  - Location: Around line ~XX (near [reference point])
  - Change: [Specific modification]
  - Why: [Justification]
  - Pattern: Similar to how `example-file.ts:38-42` handles [similar case]

### Files NOT to Touch
- `path/to/file.ts` - [Why it's not needed despite seeming relevant]

### Similar Implementations Reference
**Story:** [STORY.XX.name] - [Brief description]
- **Pattern used:** [How the similar feature was implemented]
- **Key files:**
  - `file1.ts` - [What it did]
  - `file2.ts` - [What it did]
- **Testing approach:** [How it was tested]
- **Key learnings:** [Any gotchas or important notes]

### Architectural Validation
✅ **Follows pattern:** [Pattern name from AGENTS.md or codebase]
✅ **Test coverage:** [XX% minimum as per AGENTS.md]
✅ **Conventions:** [Specific conventions being followed]
⚠️ **Consideration:** [Any concerns or things to watch for]
💡 **Recommendation:** [Suggestions for implementation]

### Testing Strategy
**Unit Tests:**
- [ ] Test [specific functionality]
- [ ] Test [error handling]
- [ ] Test [edge case]

**Integration Tests:**
- [ ] Test [integration point]
- [ ] Test [end-to-end flow]

**Edge Cases to Cover:**
1. [Edge case 1]
2. [Edge case 2]

**Coverage Target:** [XX%] (Component type: [actions/functions/services/etc])

### Implementation Order
1. **Create types/interfaces** - `path/to/types.ts`
   - Define data structures first

2. **Implement core logic** - `path/to/service.ts`
   - Follow pattern from `reference.ts`

3. **Add tests** - `path/to/__tests__/service.test.ts`
   - Achieve XX% coverage minimum

4. **Integrate** - Modify `integration/point.ts`
   - Wire up new functionality

5. **Validate** - Run full test suite
   - Ensure no regressions

### Risks & Mitigations
- ⚠️ **Risk:** [Description of potential issue]
  - **Mitigation:** [How to handle/prevent it]
  - **Testing:** [How to validate mitigation]

### Deployment Considerations
- Version bump required: [Yes/No]
- Breaking changes: [Yes/No - details]
- Migration needed: [Yes/No - approach]
- Scope additions: [Any new permissions needed in app.config.json]

---
```

### Step 8.2: Verify Enhancement Quality

Check that the analysis includes:

- Exact file paths (not vague "create a service")
- Reference to existing similar code
- Justification for each file create/modify
- Specific line numbers or reference points where helpful
- Testing strategy with coverage targets
- Risk assessment
- Clear implementation order

### Step 8.3: Save Enhanced Story

The story is now ready for implementation with world-class architectural guidance.

---

# KEY PRINCIPLES

## 1. Be Precise

- ❌ "Create a service for this"
- ✅ "`src/services/featureService.ts` - Handles [specific responsibility], follows pattern from `modelDiscoveryService.ts`"

## 2. Be Minimal

Touch ONLY what's required for the story's acceptance criteria.

- Every file creation needs justification
- Every modification needs clear purpose
- Identify files to AVOID to prevent scope creep

## 3. Be Pattern-Aware

- Don't invent new patterns if existing ones work
- Reference concrete examples from the codebase
- Follow README.md conventions religiously

## 4. Be Risk-Conscious

- Flag potential issues proactively
- Suggest mitigations
- Consider long-term implications

## 5. Be Reference-Heavy

- Link to similar implementations
- Reference specific line numbers
- Point to prior stories that solved similar problems

## 6. Be Test-Focused

- Every new feature needs tests
- Specify coverage targets
- List edge cases to cover

---

# EXAMPLE USAGE

**User:** "Run check-against-architecture on docs/stories/AGENT.08.new-feature.md"

**Skill Execution:**

1. Generate PROJECT_MAP.txt
2. Read AGENTS.md, app.config.json
3. Read AGENT.08 story completely
4. Search map for "tool", "agent", related terms
5. Read `src/agent/tools.ts`, `src/agent/graph.ts` as examples
6. Find AGENT.02 (similar story) for reference
7. Validate against CSP, timeout, testing requirements
8. Identify: Create `newTool.ts`, modify `graph.ts:45`, `agent-run.function.ts:120`
9. Edit story file with comprehensive analysis section
10. Report: "Story enhanced with architectural analysis"

**Result:** Story is now implementation-ready with clear, precise guidance.

---

# ARCHITECTURE MAINTENANCE MODE

When running in maintenance mode (no story input), perform a full codebase health audit.

## MAINTENANCE PHASE 1: Generate Fresh Context

### Step M1.1: Regenerate Architecture Map

```bash
python3 agent-tools/architecture-overview.py > PROJECT_MAP.txt
```

### Step M1.2: Load Current Conventions

Read `AGENTS.md` to understand expected patterns.

---

## MAINTENANCE PHASE 2: Convention Compliance Checks

### Check 2.1: External API Calls from UI

```bash
# UI should only call /api/ endpoints, not external URLs
grep -rn "fetch(" ui/ | grep -v "/api/"
grep -rn "axios" ui/
```

### Check M2.2: Back end API Logging

```bash
# All functions should have logging - find those without
for f in api/; do
  if ! grep -q "console.log" "$f"; then
    echo "Missing logging: $f"
  fi
done

# Check for proper prefix pattern [ComponentName]
grep -L "\[.*\]" api/*.function.ts
```

---

## MAINTENANCE PHASE 3: Test Coverage Audit

### Check M3.1: Files Without Tests

```bash
# Source files that should have tests but don't
grep "^FILE|" PROJECT_MAP.txt | grep -v "\.test\." | grep -v "__mocks__" | grep -v "\.d\.ts" | grep "|false$"
```

### Check M3.2: Coverage by Component Type

```bash
# API functions (should be 80%+)
grep "^FILE|api/.*\.function\.ts|" PROJECT_MAP.txt | grep -v "\.test\."

# UI components (should be 70%+)
grep "^FILE|ui/app/components/.*\.tsx|" PROJECT_MAP.txt | grep -v "\.test\."

# Hooks (should be 70%+)
grep "^FILE|ui/app/hooks/.*\.ts|" PROJECT_MAP.txt | grep -v "\.test\."

# Utils (should be 90%+)
grep "^FILE|ui/app/utils/.*\.ts|" PROJECT_MAP.txt | grep -v "\.test\."
```

---

## MAINTENANCE PHASE 4: Structural Consistency

### Check M4.1: Orphaned Files

Look for files that:

- Are not imported by anything
- Don't export anything used elsewhere

```bash
# Find all exports
grep "^EXPORT|" PROJECT_MAP.txt | cut -d'|' -f3 | sort | uniq > /tmp/exports.txt

# Find all imports
grep "^IMPORT|" PROJECT_MAP.txt | cut -d'|' -f3 | sort | uniq > /tmp/imports.txt

# Compare for potential orphans
```

### Check M4.2: Type Definition Consistency

```bash
# Ensure types are in correct locations
grep "^TYPE|" PROJECT_MAP.txt | cut -d'|' -f2 | sort | uniq -c | sort -rn

# Types should primarily be in types/ folders
grep "^TYPE|" PROJECT_MAP.txt | grep -v "/types/"
```

### Check M4.3: Hook Naming Convention

```bash
# All hooks should start with 'use'
grep "^FILE|ui/app/hooks/" PROJECT_MAP.txt | grep -v "use" | grep -v "\.test\."
```

---

## MAINTENANCE PHASE 5: Generate Health Report

Output a structured report:

```markdown
# Architecture Health Report

_Generated: [DATE]_

## Summary

- Total Files: [X]
- Files with Tests: [X] ([X]%)
- Convention Violations: [X]

## Convention Compliance

### ✅ Passing Checks

- [List passing checks]

### ❌ Violations Found

- **[Violation type]**: [File path] - [Description]
  - Fix: [How to fix]

## Test Coverage Gaps

| Component | Has Tests | Coverage Target | Status |
| --------- | --------- | --------------- | ------ |
| [file]    | Yes/No    | 80%             | ✅/❌  |

## Recommendations

### High Priority

1. [Recommendation with specific file/action]

### Medium Priority

1. [Recommendation]

### Low Priority

1. [Recommendation]

## Next Actions

- [ ] [Specific action item]
- [ ] [Specific action item]
```

---

# QUICK REFERENCE

## Key Files

- `AGENTS.md` - Project conventions
- `PROJECT_MAP.txt` - Machine-readable codebase index
- `app.config.json` - App configuration
- `docs/stories/` - Feature stories

## Coverage Targets

| Type          | Target |
| ------------- | ------ |
| UI components | 80%    |
| Utilities     | 90%    |
