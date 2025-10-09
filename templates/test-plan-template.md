# Test Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan-tests` command. See `.specify/templates/commands/plan-tests.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**End-to-End Testing Frameworks**: [e.g., Cypress, Selenium, Playwright or N/A]
**Mocking Frameworks**: [e.g., moq, unittest.mock, MockK, sinon or N/A]
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [single/web/mobile - determines source structure]  

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this feature)

```
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan-tests command output)
├── research.md          # Phase 0 output (/speckit.plan-tests command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan-tests)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for these tests. Delete unused options and expand the chosen structure with
  real paths. The delivered plan must not include Option labels.
-->

Note that in all cases tests should be grouped under a tests/ directory or similar.

```
# [REMOVE IF UNUSED] Option 1: Functional tests divided by type (DEFAULT)
tests/
├── contract/
├── integration/
├── unit/
└── end-to-end/

# [REMOVE IF UNUSED] Option 2: Tests organized by feature
tests/
├── feature1/
├── feature2/
└── feature3/

# [REMOVE IF UNUSED] Option 3: Single test suite (small/simple projects)
tests/
└── all_tests/
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

### Test Cases
[Outline the test cases to be implemented, organized by source file or feature as
appropriate. Include brief descriptions of the purpose of each test case.]

### Do Not Change Existing Product Code

This plan is for creating new test files only. Do not modify existing product code. Only add new test files.

If a test fails because of a bug in the existing code, document the failure and allow the test to fail. Do not change existing code to make tests pass.

If a minor change is absolutely necessary to enable testing (e.g., making a method public), ask for approval first.

## Complexity Tracking

*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
