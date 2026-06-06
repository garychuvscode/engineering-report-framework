# Engineering Report Framework Roadmap

## Current Status

Engineering Report Framework (ERF) is currently in the **spec and design phase**.

The project has established the initial public architecture documents for:

- IC validation data indexing
- Excel-contained workbook storage
- JSON-based project and workbook indexes
- Human-readable Excel SUMMARY sheets
- Python Core API design
- Development flow from Python Core to CLI, GUI, and AI-assisted workflows

This roadmap defines the next development milestones for turning the specifications into a working open-source tool.

---

## Roadmap Philosophy

ERF should be built in this order:

```text
Python Core first
CLI for AI and automation second
GUI for human review later
```

The main reason is maintainability.

Core logic should live in reusable Python modules. CLI and GUI should call the same Core API instead of duplicating logic.

This allows ERF to support:

- AI / Codex-assisted development
- Batch automation
- Future g-Pico integration
- Human review workflows
- Future GUI report assembly
- Future AI-assisted reporting

---

## Phase 0 - Spec Baseline

### Goal

Define the architecture before implementation.

### Status

Completed.

### Deliverables

- [x] IC validation data index specification
- [x] Chinese and English spec documents
- [x] Design flow documents
- [x] Core API specification
- [x] README update with project status and documentation links

### Key Files

- [`docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md`](docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md)
- [`docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_ZH.md`](docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_ZH.md)
- [`docs/design/DESIGN_FLOW_V0_1_EN.md`](docs/design/DESIGN_FLOW_V0_1_EN.md)
- [`docs/design/DESIGN_FLOW_V0_1_ZH.md`](docs/design/DESIGN_FLOW_V0_1_ZH.md)
- [`docs/spec/ERF_CORE_API_SPEC_V0_1_EN.md`](docs/spec/ERF_CORE_API_SPEC_V0_1_EN.md)
- [`docs/spec/ERF_CORE_API_SPEC_V0_1_ZH.md`](docs/spec/ERF_CORE_API_SPEC_V0_1_ZH.md)

---

## Phase 1 - Core API Foundation

### Goal

Build the first executable Python foundation for ERF.

This phase should focus on project creation, JSON index management, Excel skeleton generation, SUMMARY sheet writing, and validation.

It should not focus on GUI, report styling, real hardware data acquisition, or AI summary generation yet.

### Planned Package Structure

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
└── utils/
    ├── file_utils.py
    └── json_utils.py
```

### Planned Tasks

- [ ] Create initial Python package skeleton
- [ ] Create `ERFResult` and `ValidationResult` result objects
- [ ] Create `ERFProjectManager`
- [ ] Create `ManifestManager`
- [ ] Create `WorkbookIndexManager`
- [ ] Create `ExcelWorkbookBuilder`
- [ ] Create `SummarySheetWriter`
- [ ] Create `ERFValidator`
- [ ] Implement `create_project()`
- [ ] Implement `add_workbook()`
- [ ] Implement `add_sheet()`
- [ ] Implement `validate_project()`
- [ ] Add basic pytest coverage

### Dry Run Target

ERF should be able to create this project without real raw data:

```text
ABC123_VALIDATION_2026/
├── manifest.json
├── indexes/
│   └── ABC123_EFF.index.json
├── workbooks/
│   └── ABC123_EFF.xlsx
└── logs/
```

The workbook should contain:

```text
SUMMARY
EFF_001
```

### Acceptance Criteria

- [ ] ERF can create a project folder
- [ ] ERF can generate `manifest.json`
- [ ] ERF can generate workbook index JSON
- [ ] ERF can create an Excel workbook
- [ ] ERF can create a `SUMMARY` sheet
- [ ] ERF can create a test condition sheet
- [ ] ERF can sync sheet entries into SUMMARY
- [ ] ERF can validate the generated project
- [ ] Unit tests pass

---

## Phase 2 - CLI for AI and Automation

### Goal

Expose the Core API through a command-line interface so AI agents, Codex app, batch scripts, and automation workflows can operate ERF.

### Planned Commands

- [ ] `erf create-project`
- [ ] `erf add-workbook`
- [ ] `erf add-sheet`
- [ ] `erf list-workbooks`
- [ ] `erf list-sheets`
- [ ] `erf validate`

### AI-Friendly Behavior

- [ ] Support `--json` output
- [ ] Use stable exit codes
- [ ] Return structured errors
- [ ] Avoid requiring GUI or manual interaction

### Acceptance Criteria

- [ ] CLI can create a project
- [ ] CLI can add a workbook
- [ ] CLI can add a sheet
- [ ] CLI can list workbooks
- [ ] CLI can list sheets
- [ ] CLI can validate a project
- [ ] CLI output can be parsed by AI agents

---

## Phase 3 - Mock Data and Demo Workflow

### Goal

Create a working demo without real hardware or mature data acquisition systems.

This phase allows ERF to show practical value before real measurement integration is ready.

### Planned Tasks

- [ ] Create deterministic mock efficiency data
- [ ] Create deterministic mock waveform data
- [ ] Write mock data into Excel workbook
- [ ] Update dataset locators
- [ ] Update SUMMARY sheet with data ranges
- [ ] Create demo project under `examples/ic_validation_demo/`
- [ ] Add demo README

### Mock Data Example

Initial mock efficiency data may include:

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

### Acceptance Criteria

- [ ] Mock data can be generated repeatedly
- [ ] Mock data can be written to Excel
- [ ] Dataset locators are updated correctly
- [ ] Validator passes after mock data insertion
- [ ] Demo project can be used by new contributors

---

## Phase 4 - Minimal GUI for Human Review

### Goal

Create a small GUI for human review, not a full report generator.

The GUI should be an ERF project browser that helps users inspect project indexes, workbook entries, sheet entries, conditions, and validation results.

### Planned Functions

- [ ] Load ERF project folder
- [ ] Show workbook registry
- [ ] Show sheet registry
- [ ] Show condition summary
- [ ] Show config and hardware IDs
- [ ] Show dataset locators
- [ ] Run validator
- [ ] Display validation results
- [ ] Open workbook path

### Design Rule

The GUI must call the Core API. It must not reimplement JSON parsing, Excel logic, or validation logic directly.

### Acceptance Criteria

- [ ] GUI can load a demo project
- [ ] GUI can show workbook and sheet structure
- [ ] GUI can show dataset locators
- [ ] GUI can run validation
- [ ] GUI remains independent from core logic

---

## Phase 5 - Report Assembly Workflow

### Goal

Allow users to select datasets and generate a report draft.

This phase turns ERF from a data index tool into a report assembly tool.

### Planned Functions

- [ ] Query datasets
- [ ] Filter by test group
- [ ] Filter by condition profile
- [ ] Select dataset locators
- [ ] Copy or reference selected Excel ranges / charts / images
- [ ] Generate report workbook

### Acceptance Criteria

- [ ] User can select datasets
- [ ] ERF can generate a report workbook
- [ ] Selected data can be copied or referenced
- [ ] Report can be opened and manually reviewed

---

## Phase 6 - AI-Assisted Report Workflow

### Goal

Allow AI agents to query ERF and suggest report content while humans keep final review authority.

### Planned Workflow

```text
AI queries ERF through CLI / Core
  ↓
AI suggests datasets for a report
  ↓
GUI displays AI suggestions
  ↓
Human confirms or modifies selection
  ↓
ERF generates report draft
```

### Acceptance Criteria

- [ ] CLI query result can be parsed by AI
- [ ] AI can recommend datasets based on structured index data
- [ ] GUI can display selected datasets
- [ ] Human remains the final reviewer

---

## Near-Term Implementation Priority

The next development milestone is:

```text
Phase 1 - Core API Foundation
```

Recommended first Codex task:

```text
Implement the Phase 1 Core API Foundation based on:
- docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md
- docs/design/DESIGN_FLOW_V0_1_EN.md
- docs/spec/ERF_CORE_API_SPEC_V0_1_EN.md

Focus on package skeleton, project creation, manifest generation, workbook index generation, Excel skeleton creation, SUMMARY sheet writing, and basic validation.
Do not implement GUI, AI summary, PDF export, real hardware data acquisition, or full report styling yet.
```

---

## Long-Term Vision

ERF aims to become a reusable open-source engineering reporting foundation for:

- IC validation
- Automated test reporting
- Hardware engineering workflows
- Manufacturing data review
- Traceable engineering data management
- AI-assisted report generation

The long-term goal is to reduce repetitive engineering reporting work while keeping data traceable, reviewable, and reusable.
