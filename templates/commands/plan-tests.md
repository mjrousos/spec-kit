---
description: Execute the implementation planning workflow for creating functional tests using the test plan template to generate design artifacts.
scripts:
  sh: scripts/bash/setup-test-plan.sh --json
  ps: scripts/powershell/setup-test-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH.

2. **Load context**: Read FEATURE_SPEC and `.specify/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
      - If test frameworks are not specified, default to popular choices for the language/platform
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate source file list and test cases outline (fill Test Cases section)
   - Phase 2: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2. Report branch, IMPL_PLAN path, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

3. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

4. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Test Plan

**Prerequisites:** `research.md` complete

1. **Extract source structure**:
   - Analyze solution to identify source files to be tested
   - Document source files to test in `source-files.md` with brief description of each using format:
       - `path/to/source/file.ext`: [Brief description of the file's purpose and functionality]

2. **Define test cases**:
   - For every source file in `source-files.md`, outline test cases to be implemented under "Test Cases" section of the plan
   - Include purpose of each test case and any special setup/teardown needed
   - Organize test cases by source file or feature as appropriate
   - Ensure coverage of unit, integration, and end-to-end tests as relevant
   - Ensure test cases cover positive and negative scenarios, edge cases, and error handling
   - Identify equivalence classes and boundaries for each method being tested; then include parameterized test sets that include pairwise combinations for key input dimensions. 
   - Document all test cases in IMPL_PLAN under "Test Cases" section with clear descriptions
   - This can be slow, so consider planning test cases by breaking the complete list of source files into parts or even handling one source file at a time then moving to the next (`source-files.md` can be updated to track status)

3. **Test case review**:
   - Review generated test cases for completeness and clarity
   - Ensure that test cases exist for all source files in `source-files.md`
   - Validate that test cases are actionable and specific enough for implementation
   - Make sure the plan includes validating code coverage meets requirements (and that additional tests are added if coverage is insufficient)

### Phase 2: Agent Context Update

**Prerequisites:** `research.md` complete

1. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: agent-specific file

## Key rules

- Use absolute paths
- Planned test cases should include unit tests, integration tests, and end-to-end tests
- ERROR on gate failures or unresolved clarifications
- Be specific and exhaustive for functional behavior. For each test, include the intention, setup/mocks/data, action, and expected result.
- Do not include non-functional checks or tools.
- Idempotency: Re-running this command MUST NOT duplicate artifacts or test cases; see Idempotency section below. If `research.md`, `source-files.md`, or the implementation/test plan already exist, parse them instead of regenerating from scratch.

## Glossary of Testing Terms

- **E2E Tests**: End-to-end tests that validate complete user workflows from start to finish
- **Equivalence Classes**: Groups of input values that should be handled similarly by the system under test
- **Integration Tests**: Tests that verify interactions between multiple components or systems
- **Mock/Test Double**: Fake objects that simulate real dependencies for testing purposes
- **Pairwise Testing**: Combinatorial testing technique that tests all possible pairs of input parameters
- **Parameterized Tests**: Tests that run multiple times with different input values (using [Theory] and [InlineData], for example)
- **Unit Tests**: Fast, isolated tests that verify individual components in isolation
