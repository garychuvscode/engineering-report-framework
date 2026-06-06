# ERF Core API Spec V0.1

## 1. Purpose

This document defines the initial Core API design for Engineering Report Framework (ERF) V0.1.

The Core API is the execution foundation of ERF. It defines the Python modules, manager classes, data objects, function interfaces, result format, responsibility boundaries, and validation behavior required to operate an ERF project.

The Core API should be used by:

- CLI commands
- GUI applications
- AI agents
- Codex app implementation tasks
- Batch automation scripts
- Future g-Pico automation workflows
- Future report assembly workflows

The main purpose of this spec is to prevent the implementation from becoming GUI-first, CLI-specific, or fragmented across different entry points.

ERF V0.1 should follow this architecture:

```text
Python Core API = system capability foundation
CLI             = AI / automation interface
GUI             = human review and report assembly interface
```

---

## 2. Design Principles

ERF Core API V0.1 should follow these principles:

1. Core first.
2. CLI wraps Core.
3. GUI wraps Core.
4. Core logic must not be duplicated in CLI or GUI.
5. CLI and GUI should not directly modify JSON or Excel files.
6. JSON is a machine-readable index, not large raw data storage.
7. Excel is the primary engineering data container in V0.1.
8. I/O operations must include error handling.
9. Main APIs should be testable with pytest.
10. API results should be easy for CLI, GUI, and AI agents to consume.

The implementation should avoid this anti-pattern:

```text
CLI directly edits manifest.json
GUI directly edits workbook index JSON
GUI directly modifies Excel workbook logic
Each entry point implements its own logic
```

The correct pattern is:

```text
CLI / GUI / AI
      ↓
Core API
      ↓
JSON / Excel / File System
```

---

## 3. Package Structure

The initial Python package structure should be:

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
├── utils/
│   ├── file_utils.py
│   └── json_utils.py
│
└── examples/
```

### 3.1 Module Roles

| Module | Role |
|---|---|
| `erf/core/` | Core data models and management logic. |
| `erf/excel/` | Excel workbook, worksheet, and SUMMARY sheet operations. |
| `erf/cli/` | CLI entry point. It must call Core API. |
| `erf/utils/` | File, JSON, path, and shared utility functions. |
| `erf/examples/` | Example workflows or helper scripts. |

GUI modules should not be added in the first core implementation phase. GUI can be added later after the Core API becomes stable.

---

## 4. Responsibility Boundary

The Core API should clearly separate responsibilities.

| Block | Responsibility |
|---|---|
| `ERFProjectManager` | Coordinates project creation, workbook creation, sheet creation, and validation. |
| `ManifestManager` | Creates, loads, saves, and updates `manifest.json`. |
| `WorkbookIndexManager` | Creates, loads, saves, and updates workbook index JSON files. |
| `ExcelWorkbookBuilder` | Creates Excel workbooks and worksheets. |
| `SummarySheetWriter` | Writes workbook index information into the Excel `SUMMARY` sheet. |
| `ERFValidator` | Validates consistency between JSON indexes, Excel files, and project structure. |
| CLI | Calls Core API and formats output. It must not directly edit JSON or Excel logic. |
| GUI | Calls Core API and displays results. It must not directly implement core logic. |

### 4.1 Boundary Rule

CLI and GUI must not directly modify:

- `manifest.json`
- workbook index JSON files
- Excel workbook structures
- validation logic

All such behavior must go through the Core API.

---

## 5. Core Data Objects

The first implementation may use Python `dataclass` objects or dictionary-backed classes.

Recommended core data objects:

| Data Object | Purpose |
|---|---|
| `ERFProjectConfig` | Input configuration for creating or loading a project. |
| `ManifestData` | Python representation of `manifest.json`. |
| `WorkbookRegistryEntry` | One workbook entry inside `manifest.json`. |
| `WorkbookIndexData` | Python representation of a workbook index JSON. |
| `SheetEntryData` | One sheet entry inside `sheet_registry`. |
| `ConditionProfile` | Test operating and measurement conditions. |
| `ICConfigProfile` | IC internal configuration, such as trim, register, and compensation settings. |
| `HardwareProfile` | EVB, BOM, board revision, and external hardware information. |
| `ExtraInfoProfile` | Project-specific or non-standard supplemental information. |
| `DatasetLocator` | Location reference for data inside Excel. |
| `ValidationResult` | Validation result object. |
| `ERFResult` | Unified result object for Core API calls. |

### 5.1 Data Object Requirements

Each data object should support:

- Conversion to `dict`
- Creation from `dict`
- JSON serialization when needed
- Basic field validation when practical

The first version does not need a heavy schema validation framework. A lightweight dataclass and validation method approach is acceptable.

---

## 6. Core Manager Classes

## 6.1 ERFProjectManager

`ERFProjectManager` is the high-level coordination layer.

It should coordinate:

- Project folder creation
- Manifest creation and update
- Workbook index creation and update
- Excel workbook creation
- SUMMARY sheet update
- Project validation

Recommended methods:

```text
create_project()
load_project()
add_workbook()
add_sheet()
validate_project()
```

`ERFProjectManager` should call lower-level managers instead of implementing all details internally.

---

## 6.2 ManifestManager

`ManifestManager` is responsible only for `manifest.json`.

Recommended methods:

```text
create_manifest()
load_manifest()
save_manifest()
add_workbook_entry()
get_workbook_entry()
list_workbooks()
```

It should not create Excel workbooks directly.

---

## 6.3 WorkbookIndexManager

`WorkbookIndexManager` is responsible only for workbook-level index JSON files.

Recommended methods:

```text
create_workbook_index()
load_workbook_index()
save_workbook_index()
add_sheet_entry()
get_sheet_entry()
list_sheets()
add_dataset_locator()
```

It should not directly modify Excel files.

---

## 6.4 ExcelWorkbookBuilder

`ExcelWorkbookBuilder` is responsible for Excel workbook and sheet creation.

Recommended methods:

```text
create_workbook()
create_summary_sheet()
create_test_condition_sheet()
create_waveform_sheet()
ensure_sheet_exists()
```

It should not decide project-level metadata. It should receive clear instructions from managers.

---

## 6.5 SummarySheetWriter

`SummarySheetWriter` converts workbook index data into a human-readable `SUMMARY` sheet.

Recommended methods:

```text
write_summary()
clear_summary()
build_summary_rows()
```

It should not own the workbook index JSON. It only reads index data and writes the Excel summary table.

---

## 6.6 ERFValidator

`ERFValidator` is responsible for validation.

Recommended methods:

```text
validate_project()
validate_manifest()
validate_workbook_index()
validate_excel_workbook()
validate_sheet_mapping()
validate_dataset_locators()
```

It should return structured validation results rather than only printing messages.

---

## 7. Function Interface

This section defines the first-level Core API functions.

The actual implementation may use class methods, but the behavior and I/O should remain consistent.

---

## 7.1 create_project()

```text
create_project(project_root, project_id, part_number, project_name=None)
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Root folder where the ERF project should be created. |
| `project_id` | `str` | Stable ERF project ID. |
| `part_number` | `str` | IC part number. |
| `project_name` | `str | None` | Optional human-readable project name. |

### Output

Returns `ERFResult`.

### Expected Behavior

Creates:

```text
project_root/
├── manifest.json
├── indexes/
├── workbooks/
└── logs/
```

---

## 7.2 add_workbook()

```text
add_workbook(project_root, test_group, workbook_id=None, description=None)
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Existing ERF project root. |
| `test_group` | `str` | Test group name, such as `EFF` or `LOAD_REG`. |
| `workbook_id` | `str | None` | Optional stable workbook ID. If omitted, ERF may generate one. |
| `description` | `str | None` | Optional workbook description. |

### Output

Returns `ERFResult`.

### Expected Behavior

- Updates `manifest.json`
- Creates workbook index JSON
- Creates Excel workbook
- Creates `SUMMARY` sheet

---

## 7.3 add_sheet()

```text
add_sheet(project_root, test_group, sheet_id, sheet_name, display_name, sheet_role="main_test_condition")
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Existing ERF project root. |
| `test_group` | `str` | Target test group. |
| `sheet_id` | `str` | Stable sheet ID. |
| `sheet_name` | `str` | Actual Excel worksheet name. |
| `display_name` | `str` | Human-readable sheet description. |
| `sheet_role` | `str` | Sheet role. Default is `main_test_condition`. |

### Output

Returns `ERFResult`.

### Expected Behavior

- Adds sheet entry to workbook index JSON
- Creates the corresponding Excel worksheet
- Updates `SUMMARY` sheet

---

## 7.4 validate_project()

```text
validate_project(project_root)
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Existing ERF project root. |

### Output

Returns `ValidationResult` or `ERFResult` containing validation details.

### Expected Behavior

Checks project structure, manifest, workbook index files, Excel workbook files, SUMMARY sheet, sheet mapping, and required fields.

---

## 8. Result Object Format

All major Core API calls should return a unified result object.

Recommended format:

```json
{
  "success": true,
  "message": "Project created successfully.",
  "data": {},
  "errors": [],
  "warnings": []
}
```

### 8.1 Field Meaning

| Field | Meaning |
|---|---|
| `success` | Whether the operation succeeded. |
| `message` | Human-readable message. |
| `data` | Operation-specific return data. |
| `errors` | List of structured errors. |
| `warnings` | List of structured warnings. |

### 8.2 Error Format

```json
{
  "error_code": "WORKBOOK_NOT_FOUND",
  "message": "Workbook file does not exist.",
  "context": {
    "path": "workbooks/ABC123_EFF.xlsx"
  }
}
```

This result format is intended to support CLI, GUI, and AI-agent usage.

---

## 9. Error Handling Policy

Core API error handling should follow these rules:

1. Core API should not print large amounts of text directly.
2. Core API should return `ERFResult` or `ValidationResult`.
3. CLI is responsible for printing terminal messages.
4. GUI is responsible for displaying messages in windows or panels.
5. Important file I/O operations must use try-except.
6. Errors should include `error_code`, `message`, and `context`.
7. Unexpected exceptions should be converted into structured errors when practical.

### 9.1 Suggested Error Codes

```text
PROJECT_NOT_FOUND
MANIFEST_NOT_FOUND
INDEX_NOT_FOUND
WORKBOOK_NOT_FOUND
SUMMARY_SHEET_NOT_FOUND
SHEET_NOT_FOUND
INVALID_JSON
INVALID_RANGE
REQUIRED_FIELD_MISSING
EXCEL_WRITE_FAILED
EXCEL_READ_FAILED
JSON_WRITE_FAILED
JSON_READ_FAILED
```

---

## 10. Excel Operation Boundary

In V0.1, Excel is the main engineering data container.

However, the initial Core API should only handle Excel skeleton operations.

### 10.1 V0.1 Excel Operations Should Include

- Create workbook
- Create `SUMMARY` sheet
- Create test condition sheet
- Create waveform sheet
- Write SUMMARY table
- Check whether sheet exists
- Check basic range format

### 10.2 V0.1 Excel Operations Should Not Include

- Complex chart styling
- Full report template formatting
- PDF export
- PowerPoint export
- Large raw data parsing
- Real instrument data acquisition
- Complex Excel dashboard generation

### 10.3 Recommended Library

V0.1 should use `openpyxl` for Excel operations because it is cross-platform and suitable for CLI, CI, Linux, and non-GUI environments.

---

## 11. JSON Operation Boundary

JSON is the machine-readable index layer.

It is not large raw data storage.

### 11.1 JSON May Store

- `project_info`
- `ic_info`
- `index_registry`
- `workbook_info`
- `sheet_registry`
- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `extra_info_profile`
- `dataset_locators`
- `index_status`
- `last_index_time`

### 11.2 JSON Must Not Store in V0.1

- Large raw measurement tables
- Full waveform sample arrays
- Image binary data
- Large chart data arrays

### 11.3 JSON Access Rule

All JSON read/write operations should go through:

- `ManifestManager`
- `WorkbookIndexManager`

CLI and GUI should not directly edit JSON files.

---

## 12. Validation API

The validation API should provide structured consistency checks.

Recommended validation functions:

```text
validate_project()
validate_manifest()
validate_workbook_index()
validate_excel_workbook()
validate_summary_sheet()
validate_sheet_registry()
validate_dataset_locator()
```

### 12.1 Validation Items

Validator should check:

- `manifest.json` exists
- Manifest required fields exist
- Index JSON files exist
- Workbook files exist
- `SUMMARY` sheet exists
- Sheets in `sheet_registry` exist in workbook
- Dataset locator format is reasonable
- `index_status` is valid

### 12.2 Validation Result Example

```json
{
  "success": false,
  "message": "Validation failed.",
  "errors": [
    {
      "error_code": "SUMMARY_SHEET_NOT_FOUND",
      "message": "SUMMARY sheet does not exist.",
      "context": {
        "workbook_path": "workbooks/ABC123_EFF.xlsx"
      }
    }
  ],
  "warnings": []
}
```

---

## 13. CLI / GUI Integration Rule

CLI and GUI must use the Core API.

### 13.1 CLI Must Not

- Directly edit Excel workbooks
- Directly edit `manifest.json`
- Directly edit workbook index JSON
- Reimplement validation logic

### 13.2 GUI Must Not

- Directly modify JSON indexes
- Directly own workbook index logic
- Lock report logic inside Tkinter callbacks
- Duplicate CLI logic

### 13.3 Correct Integration Pattern

```text
CLI command
  → ERFProjectManager / ManifestManager / Validator
  → ERFResult
  → CLI output formatting

GUI button
  → ERFProjectManager / Validator
  → ERFResult
  → GUI display
```

---

## 14. Dry Run Workflow

The Core API should support a dry run workflow without real raw data.

Recommended dry run:

```text
1. create_project("ABC123_VALIDATION_2026")
2. add_workbook(test_group="EFF")
3. add_sheet(sheet_id="eff_001", sheet_name="EFF_001")
4. write SUMMARY sheet
5. validate project
6. list workbooks
7. list sheets
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

Expected workbook content:

```text
SUMMARY
EFF_001
```

This dry run does not require real raw data, charts, instruments, or hardware.

---

## 15. Acceptance Criteria

Core API V0.1 is considered complete when it can:

1. Create an ERF project folder.
2. Create `manifest.json`.
3. Add workbook registry entry.
4. Create workbook index JSON.
5. Create Excel workbook.
6. Create `SUMMARY` sheet.
7. Create test condition sheet.
8. Write sheet entry into workbook index JSON.
9. Sync sheet entry into `SUMMARY` sheet.
10. Validate project structure and consistency.
11. Return unified `ERFResult` or `ValidationResult`.
12. Support pytest coverage for create/load/save/validate workflows.
13. Provide APIs that future CLI can call directly.
14. Provide APIs that future GUI can call directly.

---

## 16. Non-Goals

ERF Core API V0.1 does not include:

1. Full raw data table schema.
2. Real hardware data acquisition.
3. Full report template system.
4. GUI implementation.
5. AI summary generation.
6. PDF export.
7. PowerPoint export.
8. HTML dashboard export.
9. Database backend.
10. External assets storage.
11. Fixed compare profile.
12. Complex Excel styling.
13. Advanced chart generation.
14. Production-grade plugin system.

---

## 17. Final Summary

The Core API is the foundation of ERF.

All future interfaces should depend on it:

```text
AI / CLI / GUI / automation
          ↓
      Core API
          ↓
JSON index + Excel workbook + file system
```

This allows ERF to grow safely from project/index management into mock data workflow, report assembly, GUI review, and AI-assisted engineering reporting.
