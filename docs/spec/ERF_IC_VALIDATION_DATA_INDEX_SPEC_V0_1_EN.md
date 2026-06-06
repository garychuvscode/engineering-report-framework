# ERF IC Validation Data Index Spec V0.1

## 1. Overview

Engineering Report Framework (ERF) V0.1 defines a data indexing architecture for IC validation projects.

The purpose of this spec is to provide a structured way to organize IC validation data using Excel workbooks as the primary data container and JSON files as machine-readable indexes.

In V0.1, ERF focuses on:

- IC validation data organization
- Excel workbook-based data storage
- JSON-based project and workbook indexing
- Human-readable Excel summary sheets
- Dataset location tracking inside Excel
- Support for future report generation, recap, and analysis

This spec does not define the detailed raw data table format yet. The current goal is to define the project structure, index structure, workbook structure, and relationship between Excel and JSON.

---

## 2. Design Goals

ERF V0.1 is designed with the following goals:

1. Use Excel as the main engineering data container.
2. Use JSON as the structural index and metadata description.
3. Avoid frequent Excel file opening just to understand what data exists.
4. Allow engineers to manually open Excel and quickly find data.
5. Keep all main data inside Excel in V0.1.
6. Support IC validation workflows with many test conditions.
7. Support future automation, recap, comparison, and report generation.
8. Keep the architecture simple enough for initial implementation.

---

## 3. Core Design Principle

ERF V0.1 uses this principle:

```text
Excel workbook = engineering data body
JSON index     = machine-readable data map
SUMMARY sheet  = human-readable data map
```

The Excel workbook stores the actual engineering data.

The JSON index describes where the data is located, what condition it belongs to, and how it should be identified.

The SUMMARY sheet allows engineers to open Excel directly and understand the workbook content without reading JSON.

---

## 4. V0.1 Storage Mode

ERF V0.1 uses:

```json
{
  "storage_mode": "excel_contained"
}
```

This means:

- Raw tables are stored inside Excel.
- Processed tables are stored inside Excel.
- Charts are stored inside Excel.
- Waveform tables are stored inside Excel.
- Embedded images are stored inside Excel.
- JSON only stores metadata and locators.
- External `assets/` folder is not required in V0.1.

Future versions may support:

```json
{
  "storage_mode": "hybrid"
}
```

But this is outside the scope of V0.1.

---

## 5. Project Folder Structure

A standard IC validation project should use the following structure:

```text
IC_PROJECT/
├── manifest.json
├── indexes/
│   ├── ABC123_EFF.index.json
│   ├── ABC123_LOAD_REG.index.json
│   └── ABC123_LINE_REG.index.json
├── workbooks/
│   ├── ABC123_EFF.xlsx
│   ├── ABC123_LOAD_REG.xlsx
│   └── ABC123_LINE_REG.xlsx
└── logs/
    └── index_update.log
```

### 5.1 Folder Roles

| Folder / File | Role |
|---|---|
| `manifest.json` | Project-level index. It only stores high-level project and workbook references. |
| `indexes/` | Stores one JSON index file for each workbook. |
| `workbooks/` | Stores Excel workbooks. Each workbook represents one major test group. |
| `logs/` | Stores index generation, update, validation, or error logs. |

---

## 6. Project-Level Architecture

The high-level project structure is:

```text
IC Project
│
├── manifest.json
│     └── project-level index
│
├── Workbook Index JSON files
│     └── one index per workbook
│
└── Excel Workbooks
      ├── SUMMARY sheet
      └── Test condition sheets
```

A project may contain multiple test group workbooks.

Example:

```text
ABC123_VALIDATION_PROJECT
│
├── ABC123_EFF.xlsx
├── ABC123_LOAD_REG.xlsx
├── ABC123_LINE_REG.xlsx
└── ABC123_TRANSIENT.xlsx
```

Each workbook focuses on one major validation category.

---

## 7. Workbook Naming Rule

A workbook should represent one major test group for one IC part number.

Recommended naming format:

```text
{part_number}_{test_group}.xlsx
```

Examples:

```text
ABC123_EFF.xlsx
ABC123_LOAD_REG.xlsx
ABC123_LINE_REG.xlsx
ABC123_TRANSIENT.xlsx
ABC123_ACCURACY.xlsx
```

The workbook name should stay simple. Detailed test conditions should not be encoded into the workbook filename.

---

## 8. JSON File Naming Rule

Each workbook should have one corresponding index JSON file.

Recommended naming format:

```text
{part_number}_{test_group}.index.json
```

Examples:

```text
ABC123_EFF.index.json
ABC123_LOAD_REG.index.json
ABC123_LINE_REG.index.json
```

The index JSON should describe the internal structure of the corresponding Excel workbook.

---

## 9. Workbook Internal Structure

Each workbook should contain:

```text
ABC123_EFF.xlsx
├── SUMMARY
├── EFF_001
├── EFF_002
├── EFF_003
├── EFF_001_WF_001
└── EFF_002_WF_001
```

### 9.1 Sheet Types

| Sheet Type | Example | Purpose |
|---|---|---|
| Summary sheet | `SUMMARY` | Human-readable workbook index. |
| Main test condition sheet | `EFF_001` | Stores data, chart, image, and notes for one test condition group. |
| Waveform sheet | `EFF_001_WF_001` | Stores large waveform table related to a main test sheet. |
| Other support sheet | Optional | May store supporting data if needed. |

---

## 10. Sheet Naming Rule

Excel worksheet names should stay short and stable.

Recommended naming format:

```text
{test_group}_{serial_number}
```

Examples:

```text
EFF_001
EFF_002
LOAD_REG_001
LINE_REG_001
```

For waveform-related sheets:

```text
{parent_sheet_name}_WF_{serial_number}
```

Examples:

```text
EFF_001_WF_001
EFF_001_WF_002
```

Full test conditions should not be stored in sheet names. Complete conditions should be stored in the workbook index JSON.

---

## 11. Stable ID Rule

ERF should not rely only on filenames or Excel sheet names.

The following IDs should be used:

| ID | Purpose | Example |
|---|---|---|
| `project_id` | Stable project identifier | `ABC123_VALIDATION_2026` |
| `workbook_id` | Stable workbook identifier | `wb_abc123_eff` |
| `sheet_id` | Stable sheet identifier | `eff_001` |
| `dataset_id` | Stable dataset identifier | `ds_eff_001_main` |

This allows filenames or sheet names to change while ERF still maintains stable internal references.

---

## 12. Excel vs JSON Responsibilities

### 12.1 Excel Stores

Excel should store:

- Summary sheet
- Main measurement tables
- Processed tables
- Waveform tables
- Charts
- Embedded images
- Human-readable notes
- Manual review comments

### 12.2 JSON Stores

JSON should store:

- Project metadata
- Workbook registry
- Workbook file path
- Sheet registry
- Sheet ID to sheet name mapping
- Condition profile
- IC configuration profile
- Hardware profile
- Extra information profile
- Dataset locators
- Index status
- Last index update time

### 12.3 JSON Should Not Store

JSON should not store large raw data in V0.1.

Avoid storing:

- Full raw measurement tables
- Full waveform data
- Image binary data
- Large chart data arrays

Instead, JSON should point to their locations inside Excel.

---

## 13. manifest.json

`manifest.json` is the project-level index.

It should only store high-level project information and references to workbook index files.

It should not store detailed sheet information.

### 13.1 Example manifest.json

```json
{
  "schema_version": "0.1.0",
  "index_type": "project_manifest",
  "storage_mode": "excel_contained",
  "project_info": {
    "project_id": "ABC123_VALIDATION_2026",
    "project_type": "ic_validation",
    "project_name": "ABC123 IC Validation Project",
    "created_time": "2026-06-04T10:30:00+08:00",
    "owner": "engineering_team"
  },
  "ic_info": {
    "part_number": "ABC123",
    "vendor": "ExampleVendor",
    "revision": "A1",
    "package": "QFN-32"
  },
  "index_registry": {
    "EFF": {
      "workbook_id": "wb_abc123_eff",
      "test_group": "EFF",
      "workbook_path": "workbooks/ABC123_EFF.xlsx",
      "index_path": "indexes/ABC123_EFF.index.json",
      "description": "Efficiency validation workbook"
    },
    "LOAD_REG": {
      "workbook_id": "wb_abc123_load_reg",
      "test_group": "LOAD_REG",
      "workbook_path": "workbooks/ABC123_LOAD_REG.xlsx",
      "index_path": "indexes/ABC123_LOAD_REG.index.json",
      "description": "Load regulation validation workbook"
    }
  }
}
```

---

## 14. Workbook Index JSON

Each workbook should have one corresponding index JSON file.

Example:

```text
workbooks/ABC123_EFF.xlsx
indexes/ABC123_EFF.index.json
```

The workbook index JSON describes:

- Which workbook it maps to
- Which sheets are inside the workbook
- What each sheet represents
- What test conditions are associated with each sheet
- What IC configuration was used
- What hardware configuration was used
- Where datasets are located inside Excel

---

## 15. Workbook Index JSON Structure

### 15.1 Top-Level Structure

```json
{
  "schema_version": "0.1.0",
  "index_type": "workbook_index",
  "storage_mode": "excel_contained",
  "project_ref": {},
  "workbook_info": {},
  "sheet_registry": {}
}
```

---

## 16. workbook_info

`workbook_info` describes the target Excel workbook.

### 16.1 Example

```json
{
  "workbook_info": {
    "workbook_id": "wb_abc123_eff",
    "test_group": "EFF",
    "file_name": "ABC123_EFF.xlsx",
    "file_path": "workbooks/ABC123_EFF.xlsx",
    "description": "Efficiency validation data workbook",
    "summary_sheet_name": "SUMMARY",
    "index_status": "valid",
    "last_index_time": "2026-06-04T10:30:00+08:00",
    "workbook_checksum": "optional_sha256"
  }
}
```

### 16.2 index_status

Recommended values:

| Value | Meaning |
|---|---|
| `valid` | Index matches the workbook. |
| `stale` | Workbook may have changed after index generation. |
| `needs_reindex` | Index should be regenerated. |
| `error` | Index validation failed. |

---

## 17. sheet_registry

`sheet_registry` is the core of the workbook index JSON.

It maps each test condition sheet to its metadata, profiles, and dataset locators.

### 17.1 Example

```json
{
  "sheet_registry": {
    "eff_001": {
      "sheet_id": "eff_001",
      "sheet_name": "EFF_001",
      "display_name": "Efficiency Test Condition 001",
      "sheet_role": "main_test_condition",
      "parent_sheet_id": null,
      "condition_profile": {},
      "ic_config_profile": {},
      "hardware_profile": {},
      "extra_info_profile": {},
      "dataset_locators": [],
      "optional_tags": []
    }
  }
}
```

---

## 18. condition_profile

`condition_profile` stores operating conditions and measurement setup conditions.

It should describe how the DUT was operated during the test.

### 18.1 Should Include

- VIN
- VOUT
- IOUT
- Temperature
- Mode
- Load type
- Switching frequency
- Sweep axis
- Fixed operating conditions
- Measurement metrics

### 18.2 Example

```json
{
  "condition_profile": {
    "condition_id": "cond_eff_001",
    "fixed_conditions": {
      "vin_v": 5.0,
      "vout_v": 3.3,
      "temperature_c": 25,
      "mode": "PWM",
      "load_type": "electronic_load",
      "switching_frequency_khz": 500
    },
    "sweep_axes": {
      "iout_a": {
        "start": 0.0,
        "stop": 2.0,
        "step": 0.1,
        "unit": "A"
      }
    },
    "measured_metrics": [
      "vin_actual_v",
      "iin_actual_a",
      "vout_actual_v",
      "iout_actual_a",
      "efficiency_percent",
      "vlx_avg_v",
      "vlx_ripple_vpp",
      "ilx_peak_a"
    ]
  }
}
```

---

## 19. ic_config_profile

`ic_config_profile` stores IC internal configuration information.

This profile should be used for settings that may affect validation results.

### 19.1 Should Include

- Trim code
- Compensation setting
- Register setting
- Mode configuration
- OTP / NVM setting
- I2C / SPI register snapshot
- Configuration source
- Configuration time

### 19.2 Example

```json
{
  "ic_config_profile": {
    "config_id": "cfg_default_trim_001",
    "config_name": "Default Trim Configuration",
    "trim_settings": {
      "vout_trim_code": "0x12",
      "current_limit_trim_code": "0x08",
      "frequency_trim_code": "0x03"
    },
    "compensation_settings": {
      "compensation_mode": "internal",
      "comp_code": "0x05"
    },
    "register_snapshot": {
      "0x01": "0x12",
      "0x02": "0x08",
      "0x03": "0x05"
    },
    "config_source": "i2c_readback",
    "config_time": "2026-06-04T10:30:00+08:00"
  }
}
```

---

## 20. hardware_profile

`hardware_profile` stores external hardware and board-level information.

This profile should be used for hardware settings that may affect measurement results.

### 20.1 Should Include

- EVB name
- Board revision
- BOM revision
- Critical components
- Inductor
- Input capacitor
- Output capacitor
- Sense resistor
- Layout note
- Cable / probe setup if needed

### 20.2 Example

```json
{
  "hardware_profile": {
    "hardware_id": "hw_bom_a_evb_rev1",
    "board_name": "ABC123_EVB",
    "board_revision": "REV_1.0",
    "bom_revision": "BOM_A",
    "critical_components": {
      "inductor": {
        "value": "2.2uH",
        "part_number": "LQHxxxx",
        "vendor": "Murata",
        "dcr_mohm": 35
      },
      "input_cap": {
        "value": "22uF",
        "quantity": 2,
        "part_number": "GRMxxxx"
      },
      "output_cap": {
        "value": "47uF",
        "quantity": 2,
        "part_number": "GRMxxxx"
      }
    },
    "hardware_notes": "Default EVB setup"
  }
}
```

---

## 21. extra_info_profile

`extra_info_profile` stores additional information that does not fit into the main standard profiles.

It should be used carefully.

### 21.1 Rule

Information should be placed in standard profiles first whenever possible.

Use `extra_info_profile` only when:

- The information is project-specific
- The field does not yet have a standard category
- The information is mainly used for notes or review
- The information is not part of the core operating condition
- The information is not part of the IC configuration
- The information is not part of the hardware configuration

### 21.2 Should Not Be Used For

| Information | Correct Profile |
|---|---|
| VIN / VOUT / IOUT / Temperature / Mode | `condition_profile` |
| Trim code / register setting / compensation | `ic_config_profile` |
| BOM / EVB revision / external components | `hardware_profile` |
| Excel range / chart / image location | `dataset_locators` |

### 21.3 Example

```json
{
  "extra_info_profile": {
    "notes": "Special observation during high load test.",
    "custom_fields": {
      "customer_case_id": "CASE-2026-001",
      "operator_comment": "Need to re-check VLX waveform at high load.",
      "lab_temperature_note": "Air conditioner was unstable during this run."
    },
    "review_status": {
      "needs_review": true,
      "review_owner": "validation_team",
      "review_note": "Check waveform quality before final report."
    }
  }
}
```

---

## 22. dataset_locators

`dataset_locators` describe where datasets are located inside the Excel workbook.

A dataset locator is not the data itself.

It only answers:

```text
Where is this dataset located inside Excel?
```

Dataset locators allow ERF to locate:

- Main measurement tables
- Processed tables
- Waveform tables
- Charts
- Embedded images
- Notes or supporting tables

---

## 23. Dataset Locator Types

ERF V0.1 should support the following locator types:

| Locator Type | Purpose |
|---|---|
| `excel_range` | Locate a table or range inside a worksheet. |
| `excel_table` | Locate an Excel table by table name. |
| `excel_chart` | Locate a chart inside a worksheet. |
| `excel_image` | Locate an embedded image inside a worksheet. |
| `excel_sheet` | Locate a full worksheet. |

---

## 24. excel_range Locator

Used for table-like data stored in a cell range.

### Example

```json
{
  "dataset_id": "ds_eff_001_main",
  "dataset_name": "Efficiency Main Measurement Table",
  "dataset_type": "main_measurement_table",
  "data_level": "raw",
  "parent_dataset_id": null,
  "locator": {
    "locator_type": "excel_range",
    "workbook_id": "wb_abc123_eff",
    "sheet_id": "eff_001",
    "sheet_name": "EFF_001",
    "range_a1": "A8:J108",
    "table_name": "tbl_eff_001_main",
    "header_rows": 1,
    "data_start_cell": "A9"
  }
}
```

---

## 25. excel_chart Locator

Used for charts stored inside Excel.

### Example

```json
{
  "dataset_id": "ds_eff_001_chart",
  "dataset_name": "Efficiency Curve",
  "dataset_type": "chart",
  "data_level": "processed",
  "parent_dataset_id": "ds_eff_001_main",
  "locator": {
    "locator_type": "excel_chart",
    "workbook_id": "wb_abc123_eff",
    "sheet_id": "eff_001",
    "sheet_name": "EFF_001",
    "chart_name": "chart_eff_001",
    "anchor_cell": "L8",
    "anchor_range": "L8:S25"
  }
}
```

---

## 26. excel_image Locator

Used for embedded images inside Excel.

### Example

```json
{
  "dataset_id": "ds_eff_001_scope_image",
  "dataset_name": "Scope Capture at High Load",
  "dataset_type": "embedded_image",
  "data_level": "raw_evidence",
  "parent_dataset_id": "ds_eff_001_main",
  "locator": {
    "locator_type": "excel_image",
    "workbook_id": "wb_abc123_eff",
    "sheet_id": "eff_001",
    "sheet_name": "EFF_001",
    "image_name": "scope_eff_001_high_load",
    "anchor_cell": "L28",
    "anchor_range": "L28:S45"
  }
}
```

---

## 27. Waveform Dataset Locator

Waveform data should stay inside Excel in V0.1.

If waveform data is large, it should be placed in a separate waveform sheet, but still inside the same workbook.

### Example Workbook Structure

```text
ABC123_EFF.xlsx
├── SUMMARY
├── EFF_001
└── EFF_001_WF_001
```

### Example Locator

```json
{
  "dataset_id": "ds_eff_001_wf_001",
  "dataset_name": "VLX and ILX Waveform at IOUT 1A",
  "dataset_type": "waveform_table",
  "data_level": "raw",
  "parent_dataset_id": "ds_eff_001_main",
  "condition_ref": {
    "iout_a": 1.0
  },
  "locator": {
    "locator_type": "excel_range",
    "workbook_id": "wb_abc123_eff",
    "sheet_id": "eff_001_wf_001",
    "sheet_name": "EFF_001_WF_001",
    "range_a1": "A1:C5000",
    "table_name": "tbl_eff_001_wf_001",
    "header_rows": 1
  }
}
```

---

## 28. optional_tags

`optional_tags` may be used for human-readable labels.

It is not required.

It should not replace structured profiles.

### 28.1 Good Use Cases

```json
{
  "optional_tags": [
    "needs_review",
    "abnormal_ripple",
    "customer_case",
    "golden_sample",
    "mode_transition_observed"
  ]
}
```

### 28.2 Bad Use Cases

Do not use tags to store structured conditions:

```json
{
  "optional_tags": [
    "VIN_5V",
    "TEMP_25C",
    "BOM_A"
  ]
}
```

These should be stored in:

- `condition_profile`
- `hardware_profile`
- `ic_config_profile`

---

## 29. SUMMARY Sheet

Each workbook must include a `SUMMARY` sheet.

The `SUMMARY` sheet is the human-readable mirror of the workbook index JSON.

It allows engineers to open Excel and quickly understand:

- What sheets exist
- What each sheet means
- What condition each sheet represents
- Where raw data is located
- Where charts/images/waveforms are located
- Whether special notes exist

---

## 30. Recommended SUMMARY Sheet Columns

The `SUMMARY` sheet should include the following columns:

| Column | Description |
|---|---|
| `sheet_id` | ERF stable sheet ID. |
| `sheet_name` | Actual Excel worksheet name. |
| `sheet_role` | Main test condition, waveform sheet, support sheet, etc. |
| `display_name` | Human-readable test description. |
| `condition_summary` | Short condition summary. |
| `config_id` | IC configuration ID. |
| `hardware_id` | Hardware profile ID. |
| `dataset_count` | Number of datasets linked to this sheet. |
| `main_data_range` | Main raw data range if available. |
| `chart_range` | Main chart location if available. |
| `image_range` | Main image location if available. |
| `notes` | Human-readable notes. |

---

## 31. SUMMARY and JSON Relationship

The workbook index JSON is the machine-readable source index.

The SUMMARY sheet is the human-readable mirror index.

Recommended rule:

```text
Workbook index JSON = machine-readable index
SUMMARY sheet       = human-readable mirror
```

When the workbook index JSON is updated, the SUMMARY sheet should also be updated.

If Excel is manually edited, ERF should provide a future re-index function to rebuild or validate the JSON index.

---

## 32. Index Synchronization

Because Excel files may be edited manually, ERF should track index status.

Each workbook index JSON should include:

```json
{
  "index_status": "valid",
  "last_index_time": "2026-06-04T10:30:00+08:00",
  "workbook_checksum": "optional_sha256"
}
```

Future implementation may check:

- Whether workbook exists
- Whether sheet names still exist
- Whether ranges still exist
- Whether required sheets exist
- Whether SUMMARY matches JSON
- Whether workbook checksum changed

---

## 33. Required Fields

### 33.1 Required in manifest.json

- `schema_version`
- `index_type`
- `storage_mode`
- `project_info.project_id`
- `project_info.project_type`
- `ic_info.part_number`
- `index_registry`

### 33.2 Required in Workbook Index JSON

- `schema_version`
- `index_type`
- `storage_mode`
- `project_ref`
- `workbook_info.workbook_id`
- `workbook_info.file_path`
- `workbook_info.test_group`
- `workbook_info.summary_sheet_name`
- `sheet_registry`

### 33.3 Required in Each Sheet Entry

- `sheet_id`
- `sheet_name`
- `display_name`
- `sheet_role`
- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `dataset_locators`

---

## 34. V0.1 Non-Goals

ERF V0.1 does not define:

- Detailed raw data table column format
- CSV storage format
- External asset folder format
- Database storage
- Automatic comparison rules
- Fixed compare profile
- AI summary generation
- PDF export
- PowerPoint export
- HTML dashboard generation

These may be added in future versions.

---

## 35. Comparison Policy

ERF V0.1 does not define a fixed `compare_profile`.

Reason:

IC validation comparison rules may depend on user needs, project requirements, tolerance assumptions, and engineering judgment.

Instead, ERF V0.1 focuses on storing enough structured information so users or future tools can decide whether datasets are comparable.

The following profiles provide the required context for future comparison:

- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `extra_info_profile`
- `dataset_locators`

---

## 36. Implementation Direction for Codex App

The first implementation should focus on the index foundation, not report styling.

Recommended initial tasks:

1. Create ERF project folder structure.
2. Generate `manifest.json`.
3. Generate workbook-level index JSON.
4. Create Excel workbook.
5. Create `SUMMARY` sheet.
6. Create test condition sheets using stable `sheet_id`.
7. Write sheet mapping into workbook index JSON.
8. Write human-readable sheet mapping into `SUMMARY`.
9. Add basic validation to check:
   - workbook file exists
   - index JSON exists
   - required fields exist
   - SUMMARY sheet exists
   - sheet names in JSON exist in workbook

---

## 37. Acceptance Criteria for V0.1

V0.1 is considered complete when ERF can:

1. Create a new IC validation project.
2. Create a project-level `manifest.json`.
3. Create at least one workbook index JSON.
4. Create at least one Excel workbook.
5. Create a `SUMMARY` sheet in the workbook.
6. Create at least one test condition sheet.
7. Store condition/config/hardware/extra profiles in JSON.
8. Store dataset locators in JSON.
9. Write a readable summary table into Excel.
10. Validate basic consistency between JSON and Excel.

---

## 38. Final V0.1 Architecture Summary

```text
IC_PROJECT/
├── manifest.json
│     └── Project-level index
│
├── indexes/
│   └── {part_number}_{test_group}.index.json
│         └── Workbook-level machine-readable index
│
├── workbooks/
│   └── {part_number}_{test_group}.xlsx
│         ├── SUMMARY
│         │     └── Human-readable workbook index
│         │
│         ├── Test condition sheets
│         │     └── Main data, charts, images, notes
│         │
│         └── Waveform sheets
│               └── Large waveform tables inside Excel
│
└── logs/
      └── Index update and validation logs
```

ERF V0.1 keeps engineering data inside Excel and uses JSON to describe structure, meaning, and location.

This provides a clean foundation for future automation test report generation, IC validation recap, and engineering report workflows.
