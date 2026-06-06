# ERF Design Flow V0.1

## 1. Purpose

This document defines the initial development flow for Engineering Report Framework (ERF) V0.1.

The current ERF specification defines the upper-level data model for IC validation workflows. However, the real data acquisition system, hardware fixtures, instrument control layer, and complete raw data pipeline are not mature yet.

Therefore, the first development goal is not to build a full end-to-end report generator immediately. Instead, ERF should first build a reliable foundation that can create, index, validate, browse, and later assemble engineering report data.

The development direction is:

```text
Python Core first
CLI for AI and automation second
GUI for human review later
```

The core principle is:

```text
Core Python package = execution foundation
CLI                 = AI / automation interface
GUI                 = human review and report assembly interface
```

---

## 2. Design Philosophy

ERF should not lock core logic inside a GUI.

All important behavior should be implemented in reusable Python modules first. The CLI and GUI should call the same core modules.

This allows ERF to support:

- Codex app implementation
- AI agent operation
- Batch automation
- Test automation
- Future g-Pico integration
- Future Tkinter GUI operation
- Future report assembly workflow

The GUI is still valuable in an AI era, but its role should be human-in-the-loop review, filtering, validation, and report assembly instead of being the system core.

---

## 3. Development Phases

ERF V0.1 should be developed in the following phases:

```text
Phase 0 - Spec Baseline
Phase 1 - Python Core Foundation
Phase 2 - Project / Index / Excel Skeleton Generator
Phase 3 - Validator and Dry Run Demo
Phase 4 - CLI for AI and Automation
Phase 5 - Mock Data Workflow
Phase 6 - Minimal GUI for Human Review
Phase 7 - Report Assembly Workflow
Phase 8 - AI Assisted Report Workflow
```

Each phase should have a clear purpose, dry run strategy, and acceptance criteria.

---

## 4. Phase 0 - Spec Baseline

### Goal

Establish the baseline data architecture for ERF V0.1.

Existing spec documents:

```text
docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md
docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_ZH.md
```

### Scope

Phase 0 defines:

- Excel as the engineering data body
- JSON as the machine-readable index
- SUMMARY sheet as the human-readable index
- Workbook as the test group container
- Sheet as the test condition group
- Dataset locator as the Excel data location reference
- V0.1 non-goals and boundaries

### Dry Run

Review the spec manually and confirm:

- Project folder structure is reasonable
- `manifest.json` role is clear
- Workbook index JSON role is clear
- Excel / JSON responsibility split is clear
- V0.1 does not need detailed raw data table definition yet

### Acceptance Criteria

- English spec exists
- Chinese spec exists
- Both specs are semantically aligned
- V0.1 scope and non-goals are clear
- Future implementation tasks can be derived from the spec

---

## 5. Phase 1 - Python Core Foundation

### Goal

Create the initial ERF Python package foundation.

This phase should not implement full report generation, GUI, or hardware data acquisition yet.

### Recommended Package Structure

```text
erf/
├── __init__.py
├── core/
│   ├── project_manager.py
│   ├── manifest_manager.py
│   ├── workbook_index_manager.py
│   ├── sheet_entry.py
│   ├── dataset_locator.py
│   └── validator.py
│
├── excel/
│   ├── workbook_builder.py
│   └── summary_writer.py
│
├── cli/
│   └── erf_cli.py
│
└── examples/
```

### Design Rules

The core layer should:

- Not depend on GUI
- Not depend on real hardware
- Not depend on real raw data
- Be reusable by CLI, GUI, AI agents, and tests
- Provide stable Python APIs

### Dry Run Without Raw Data

Create an empty IC validation project:

```text
ABC123_VALIDATION/
├── manifest.json
├── indexes/
├── workbooks/
└── logs/
```

Verify that Python core can create, save, and load the basic structure.

### Acceptance Criteria

- ERF project folder can be created
- Basic `manifest.json` can be generated
- `manifest.json` can be loaded back
- Workbook registry entry can be added
- Workbook index JSON object can be created
- JSON objects can be serialized and deserialized
- Basic unit tests pass

---

## 6. Phase 2 - Project / Index / Excel Skeleton Generator

### Goal

Generate a complete ERF project skeleton based on the V0.1 spec.

### Main Functions

The implementation should provide functions similar to:

```text
create_project()
add_workbook()
create_workbook_index()
create_excel_workbook()
create_summary_sheet()
add_test_condition_sheet()
write_summary_table()
```

### Dry Run Example

Use a mock IC project:

```text
part_number = ABC123
project_id = ABC123_VALIDATION_2026
test_group = EFF
sheet_id = eff_001
sheet_name = EFF_001
```

Expected output:

```text
ABC123_VALIDATION_2026/
├── manifest.json
├── indexes/
│   └── ABC123_EFF.index.json
├── workbooks/
│   └── ABC123_EFF.xlsx
└── logs/
```

The Excel workbook should contain at least:

```text
SUMMARY
EFF_001
```

### Verification Without Raw Data

The test condition sheet may be empty or contain placeholders.

The workbook index JSON should contain:

- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `extra_info_profile`
- `dataset_locators`

`dataset_locators` may be an empty list at this phase.

### Acceptance Criteria

- Complete ERF project folder can be generated
- `manifest.json` follows the spec
- Workbook index JSON follows the spec
- Excel workbook exists
- Excel workbook has a `SUMMARY` sheet
- Excel workbook has at least one test condition sheet
- SUMMARY sheet content maps to JSON `sheet_registry`
- The dry run does not require real raw data

---

## 7. Phase 3 - Validator and Dry Run Demo

### Goal

Build consistency validation between JSON index files and Excel workbooks.

This phase is important because Excel workbooks may be manually modified.

### Validator Should Check

- `manifest.json` exists
- Required fields in `manifest.json` exist
- Index JSON files referenced by `index_registry` exist
- Excel workbooks referenced by `workbook_path` exist
- Required fields in workbook index JSON exist
- `SUMMARY` sheet exists
- Sheet names in `sheet_registry` exist in workbook
- Dataset locator range format is valid enough for V0.1
- `index_status` is one of supported values

### Dry Run

Use the project generated in Phase 2:

```text
erf validate ABC123_VALIDATION_2026/
```

Then intentionally create failure cases:

- Delete workbook
- Delete `SUMMARY` sheet
- Rename a sheet manually
- Remove a required JSON field
- Provide an invalid range string

### Verification Without Raw Data

Validator should not require real measurement data.

It should only validate project structure, files, sheets, required fields, and locator format.

### Acceptance Criteria

- Valid mock project passes validation
- Missing workbook fails validation
- Missing index JSON fails validation
- Missing `SUMMARY` sheet fails validation
- Missing sheet recorded in JSON fails validation
- Missing required field fails validation
- Validation result can be returned as dict / JSON / console text

---

## 8. Phase 4 - CLI for AI and Automation

### Goal

Create a CLI interface so AI agents, Codex app, batch scripts, and automation systems can operate ERF.

### Initial CLI Commands

Recommended commands:

```bash
erf create-project --part-number ABC123 --project-id ABC123_VALIDATION_2026

erf add-workbook --project ABC123_VALIDATION_2026 --test-group EFF

erf add-sheet --project ABC123_VALIDATION_2026 --test-group EFF --sheet-id eff_001 --sheet-name EFF_001

erf list-workbooks --project ABC123_VALIDATION_2026

erf list-sheets --project ABC123_VALIDATION_2026 --test-group EFF

erf validate --project ABC123_VALIDATION_2026
```

### Design Rules

- CLI must call core modules
- CLI must not duplicate core logic
- CLI should support human-readable output
- CLI should support `--json` output for AI agents
- CLI should use proper exit codes

### Dry Run

A terminal or Codex app should be able to run:

```text
create project
add workbook
add sheet
list workbook
list sheet
validate
```

### Verification Without Raw Data

CLI should operate on metadata, index JSON, and Excel skeleton only.

No real raw data is required.

### Acceptance Criteria

- CLI can create project
- CLI can add workbook
- CLI can add sheet
- CLI can list workbooks
- CLI can list sheets
- CLI can run validation
- CLI success returns exit code 0
- CLI failure returns non-zero exit code
- CLI supports JSON output for AI-friendly workflows

---

## 9. Phase 5 - Mock Data Workflow

### Goal

Use deterministic mock data to validate data insertion, locator update, SUMMARY update, and demo workflows before real data acquisition is ready.

### Mock Data Generators

Recommended mock generators:

```text
mock_efficiency_data
mock_waveform_data
mock_scope_image_placeholder
```

Initial mock efficiency table may include:

```text
IOUT_A
VIN_V
VOUT_V
IIN_A
Efficiency_%
VLX_avg_V
VLX_ripple_Vpp
ILX_peak_A
```

### Dry Run

Generate:

```text
EFF_001 sheet
  ├── condition summary
  ├── mock efficiency table
  ├── simple chart
  └── dataset locator update

EFF_001_WF_001 sheet
  └── mock waveform table
```

### Verification Without Real Raw Data

Use deterministic mock data with a fixed seed or fixed values.

This allows tests to be repeatable.

### Acceptance Criteria

- Mock efficiency table can be generated
- Mock data can be written into Excel
- Dataset locator can be updated
- SUMMARY sheet can show `main_data_range`
- Waveform sheet can be created
- Validator passes after mock data insertion
- Demo project can be used as a repository example

---

## 10. Phase 6 - Minimal GUI for Human Review

### Goal

Create a minimal GUI for human review.

The first GUI should be an ERF Project Browser, not a full report generator.

### Initial GUI Functions

The GUI should be able to:

- Load project folder
- Read `manifest.json`
- Show workbook list
- Show sheet list
- Show condition summary
- Show `config_id` and `hardware_id`
- Show dataset locators
- Open workbook path
- Run validation
- Display validation results

### Design Rules

- GUI must call core modules
- GUI must not implement core logic directly
- GUI should not be required for CLI / AI workflows
- GUI should be human review interface

### Dry Run

Load the mock project generated in Phase 5.

Expected display tree:

```text
ABC123_VALIDATION_2026
  └── EFF workbook
        ├── EFF_001
        └── EFF_001_WF_001
```

### Verification Without Real Raw Data

GUI can read JSON and Excel skeleton generated by previous phases.

Real measurement data is not required.

### Acceptance Criteria

- GUI can load project folder
- GUI can display workbook registry
- GUI can display sheet registry
- GUI can display condition summary
- GUI can display dataset locators
- GUI can run validator
- GUI uses core modules instead of duplicating logic

---

## 11. Phase 7 - Report Assembly Workflow

### Goal

Allow users to select datasets and generate a report draft.

This phase turns ERF from a data index tool into a report assembly tool.

### Main Capabilities

- Query datasets
- Filter by test group
- Filter by condition profile
- Select dataset locators
- Copy or reference selected Excel ranges / charts / images
- Generate a report workbook

### Dry Run

Use mock data project to generate:

```text
ABC123_SELECTED_REPORT.xlsx
```

Expected content:

```text
Report_SUMMARY
Selected_EFF_001
Selected_Charts
Selected_Waveforms
```

### Verification Without Real Raw Data

Use mock data generated in Phase 5.

### Acceptance Criteria

- Datasets can be queried
- Datasets can be selected
- Report workbook can be created
- Selected data can be copied or referenced
- Report summary can be generated
- Report can be opened and reviewed manually

---

## 12. Phase 8 - AI Assisted Report Workflow

### Goal

Allow AI to query ERF through CLI / core and suggest report content, while humans confirm the final output through GUI.

### Workflow

```text
AI queries ERF through CLI / core
  ↓
AI suggests datasets for a report
  ↓
GUI displays AI suggestions
  ↓
Human confirms or modifies selection
  ↓
ERF generates report draft
```

### Value in an AI Era

As AI becomes more powerful, ERF remains valuable because it provides:

- Reliable engineering data index
- Traceable dataset locations
- Structured condition/config/hardware context
- Repeatable execution interface
- Human confirmation layer

ERF should act as a trusted engineering data map for AI agents.

### Acceptance Criteria

- CLI output can be parsed by AI
- Query results can be returned as JSON
- AI can recommend datasets based on query result
- GUI can display selection results
- Human remains the final reviewer

---

## 13. Recommended Additional Planning Documents

After this design flow, the following documents should be planned before heavy implementation:

### 13.1 Core API Spec

```text
docs/spec/ERF_CORE_API_SPEC_V0_1.md
```

Purpose:

- Define core Python classes and functions
- Prevent random module naming
- Clarify boundaries between managers, builders, validators, and data models

Example core classes:

```text
ERFProjectManager
ManifestManager
WorkbookIndexManager
ExcelWorkbookBuilder
SummarySheetWriter
ERFValidator
```

### 13.2 CLI Spec

```text
docs/spec/ERF_CLI_SPEC_V0_1.md
```

Purpose:

- Define CLI commands
- Define parameters
- Define output format
- Define exit codes
- Define JSON output for AI usage

### 13.3 Test Plan

```text
docs/testing/TEST_PLAN_V0_1.md
```

Purpose:

- Define unit tests
- Define integration tests
- Define mock project tests
- Define Excel validation tests
- Define CLI dry run tests

### 13.4 Demo Project Description

```text
examples/ic_validation_demo/README.md
```

Purpose:

- Explain what the demo project should generate
- Provide a visible entry point for open-source users
- Show how ERF works without real hardware

### 13.5 AI Coding Rules

```text
AGENTS.md
```

Purpose:

- Guide Codex / AI coding agents
- Define development rules
- Prevent GUI-first architecture
- Require CLI and GUI to call core modules
- Require tests for new features
- Remind agents not to store large raw data inside JSON

---

## 14. Development Priority Summary

Recommended order:

```text
1. DESIGN_FLOW_V0_1.md
2. ERF_CORE_API_SPEC_V0_1.md
3. ERF_CLI_SPEC_V0_1.md
4. TEST_PLAN_V0_1.md
5. AGENTS.md
6. Phase 1 implementation by Codex app
```

---

## 15. Final Summary

The ERF V0.1 implementation should not start from GUI or complete report generation.

It should start from a reusable Python core, then expose that core through CLI for AI and automation, and later provide a GUI for human review and report assembly.

```text
Spec tells what ERF is.
Design flow tells how to build ERF.
Core API spec tells what code interface should exist.
CLI spec tells how AI should operate ERF.
Test plan tells how to know it works.
```

This flow allows ERF to move forward even before the real raw data acquisition system is mature.
