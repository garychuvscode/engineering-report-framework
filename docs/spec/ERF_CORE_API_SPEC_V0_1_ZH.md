# ERF Core API Spec V0.1 中文版

## 1. Purpose

本文件定義 Engineering Report Framework，簡稱 ERF，在 V0.1 階段的初始 Core API 設計。

Core API 是 ERF 的執行基礎。它定義 Python modules、manager classes、data objects、function interfaces、result format、責任邊界與 validation behavior。

Core API 應被以下入口共用：

- CLI commands
- GUI applications
- AI agents
- Codex app implementation tasks
- Batch automation scripts
- 未來 g-Pico automation workflows
- 未來 report assembly workflows

本 spec 的主要目的，是避免 implementation 變成 GUI-first、CLI-specific，或在不同入口之間產生分裂邏輯。

ERF V0.1 應遵守以下架構：

```text
Python Core API = system capability foundation
CLI             = AI / automation interface
GUI             = human review and report assembly interface
```

---

## 2. Design Principles

ERF Core API V0.1 應遵守以下原則：

1. Core first。
2. CLI wraps Core。
3. GUI wraps Core。
4. Core logic 不可在 CLI 或 GUI 中重複實作。
5. CLI 與 GUI 不應直接修改 JSON 或 Excel files。
6. JSON 是 machine-readable index，不是大量 raw data storage。
7. Excel 是 V0.1 的主要 engineering data container。
8. I/O operations 必須包含 error handling。
9. 主要 APIs 應能用 pytest 測試。
10. API result 應方便 CLI、GUI 與 AI agents 使用。

Implementation 應避免以下反模式：

```text
CLI directly edits manifest.json
GUI directly edits workbook index JSON
GUI directly modifies Excel workbook logic
Each entry point implements its own logic
```

正確模式是：

```text
CLI / GUI / AI
      ↓
Core API
      ↓
JSON / Excel / File System
```

---

## 3. Package Structure

初始 Python package structure 建議如下：

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
| `erf/core/` | Core data models 與 management logic。 |
| `erf/excel/` | Excel workbook、worksheet 與 SUMMARY sheet operations。 |
| `erf/cli/` | CLI entry point。必須呼叫 Core API。 |
| `erf/utils/` | File、JSON、path 與 shared utility functions。 |
| `erf/examples/` | Example workflows 或 helper scripts。 |

第一階段 core implementation 不應先加入 GUI modules。GUI 可以等 Core API 穩定後再加入。

---

## 4. Responsibility Boundary

Core API 應明確分離責任。

| Block | Responsibility |
|---|---|
| `ERFProjectManager` | 協調 project creation、workbook creation、sheet creation 與 validation。 |
| `ManifestManager` | 建立、讀取、儲存與更新 `manifest.json`。 |
| `WorkbookIndexManager` | 建立、讀取、儲存與更新 workbook index JSON files。 |
| `ExcelWorkbookBuilder` | 建立 Excel workbooks 與 worksheets。 |
| `SummarySheetWriter` | 將 workbook index information 寫入 Excel `SUMMARY` sheet。 |
| `ERFValidator` | 驗證 JSON indexes、Excel files 與 project structure 之間的一致性。 |
| CLI | 呼叫 Core API 並格式化 output，不應直接編輯 JSON 或 Excel logic。 |
| GUI | 呼叫 Core API 並顯示 results，不應直接實作 core logic。 |

### 4.1 Boundary Rule

CLI 與 GUI 不可直接修改：

- `manifest.json`
- workbook index JSON files
- Excel workbook structures
- validation logic

所有相關行為都必須透過 Core API。

---

## 5. Core Data Objects

第一版 implementation 可使用 Python `dataclass` objects 或 dictionary-backed classes。

建議 core data objects：

| Data Object | Purpose |
|---|---|
| `ERFProjectConfig` | 建立或載入 project 的 input configuration。 |
| `ManifestData` | `manifest.json` 的 Python representation。 |
| `WorkbookRegistryEntry` | `manifest.json` 中的單一 workbook entry。 |
| `WorkbookIndexData` | workbook index JSON 的 Python representation。 |
| `SheetEntryData` | `sheet_registry` 中的單一 sheet entry。 |
| `ConditionProfile` | 測試操作條件與量測條件。 |
| `ICConfigProfile` | IC 內部設定，例如 trim、register、compensation settings。 |
| `HardwareProfile` | EVB、BOM、board revision 與 external hardware information。 |
| `ExtraInfoProfile` | Project-specific 或 non-standard supplemental information。 |
| `DatasetLocator` | Excel 內部資料位置 reference。 |
| `ValidationResult` | Validation result object。 |
| `ERFResult` | Core API calls 的 unified result object。 |

### 5.1 Data Object Requirements

每個 data object 應支援：

- 轉成 `dict`
- 從 `dict` 建立
- 需要時可 JSON serialization
- 合理的 basic field validation

第一版不需要使用很重的 schema validation framework。使用 lightweight dataclass 與 validation method 即可。

---

## 6. Core Manager Classes

## 6.1 ERFProjectManager

`ERFProjectManager` 是 high-level coordination layer。

它應負責協調：

- Project folder creation
- Manifest creation and update
- Workbook index creation and update
- Excel workbook creation
- SUMMARY sheet update
- Project validation

建議 methods：

```text
create_project()
load_project()
add_workbook()
add_sheet()
validate_project()
```

`ERFProjectManager` 應呼叫 lower-level managers，而不是自己實作所有細節。

---

## 6.2 ManifestManager

`ManifestManager` 只負責 `manifest.json`。

建議 methods：

```text
create_manifest()
load_manifest()
save_manifest()
add_workbook_entry()
get_workbook_entry()
list_workbooks()
```

它不應直接建立 Excel workbooks。

---

## 6.3 WorkbookIndexManager

`WorkbookIndexManager` 只負責 workbook-level index JSON files。

建議 methods：

```text
create_workbook_index()
load_workbook_index()
save_workbook_index()
add_sheet_entry()
get_sheet_entry()
list_sheets()
add_dataset_locator()
```

它不應直接修改 Excel files。

---

## 6.4 ExcelWorkbookBuilder

`ExcelWorkbookBuilder` 負責 Excel workbook 與 sheet creation。

建議 methods：

```text
create_workbook()
create_summary_sheet()
create_test_condition_sheet()
create_waveform_sheet()
ensure_sheet_exists()
```

它不應決定 project-level metadata。它應從 managers 接收明確指令。

---

## 6.5 SummarySheetWriter

`SummarySheetWriter` 將 workbook index data 轉成 human-readable `SUMMARY` sheet。

建議 methods：

```text
write_summary()
clear_summary()
build_summary_rows()
```

它不應擁有 workbook index JSON。它只讀取 index data 並寫入 Excel summary table。

---

## 6.6 ERFValidator

`ERFValidator` 負責 validation。

建議 methods：

```text
validate_project()
validate_manifest()
validate_workbook_index()
validate_excel_workbook()
validate_sheet_mapping()
validate_dataset_locators()
```

它應回傳 structured validation results，而不是只印出 messages。

---

## 7. Function Interface

本章定義第一層 Core API functions。

實際 implementation 可以使用 class methods，但 behavior 與 I/O 應保持一致。

---

## 7.1 create_project()

```text
create_project(project_root, project_id, part_number, project_name=None)
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | ERF project 要建立的位置。 |
| `project_id` | `str` | 穩定 ERF project ID。 |
| `part_number` | `str` | IC part number。 |
| `project_name` | `str | None` | Optional human-readable project name。 |

### Output

回傳 `ERFResult`。

### Expected Behavior

建立：

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
| `project_root` | `str | Path` | Existing ERF project root。 |
| `test_group` | `str` | Test group name，例如 `EFF` 或 `LOAD_REG`。 |
| `workbook_id` | `str | None` | Optional stable workbook ID。若省略，ERF 可自動產生。 |
| `description` | `str | None` | Optional workbook description。 |

### Output

回傳 `ERFResult`。

### Expected Behavior

- 更新 `manifest.json`
- 建立 workbook index JSON
- 建立 Excel workbook
- 建立 `SUMMARY` sheet

---

## 7.3 add_sheet()

```text
add_sheet(project_root, test_group, sheet_id, sheet_name, display_name, sheet_role="main_test_condition")
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Existing ERF project root。 |
| `test_group` | `str` | Target test group。 |
| `sheet_id` | `str` | Stable sheet ID。 |
| `sheet_name` | `str` | Actual Excel worksheet name。 |
| `display_name` | `str` | Human-readable sheet description。 |
| `sheet_role` | `str` | Sheet role。Default 是 `main_test_condition`。 |

### Output

回傳 `ERFResult`。

### Expected Behavior

- 將 sheet entry 加入 workbook index JSON
- 建立對應 Excel worksheet
- 更新 `SUMMARY` sheet

---

## 7.4 validate_project()

```text
validate_project(project_root)
```

### Input

| Parameter | Type | Description |
|---|---|---|
| `project_root` | `str | Path` | Existing ERF project root。 |

### Output

回傳包含 validation details 的 `ValidationResult` 或 `ERFResult`。

### Expected Behavior

檢查 project structure、manifest、workbook index files、Excel workbook files、SUMMARY sheet、sheet mapping 與 required fields。

---

## 8. Result Object Format

所有主要 Core API calls 應回傳 unified result object。

建議格式：

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
| `success` | Operation 是否成功。 |
| `message` | Human-readable message。 |
| `data` | Operation-specific return data。 |
| `errors` | Structured errors list。 |
| `warnings` | Structured warnings list。 |

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

此 result format 目的是同時支援 CLI、GUI 與 AI-agent usage。

---

## 9. Error Handling Policy

Core API error handling 應遵守以下規則：

1. Core API 不應直接 print 大量文字。
2. Core API 應回傳 `ERFResult` 或 `ValidationResult`。
3. CLI 負責印出 terminal messages。
4. GUI 負責在視窗或 panel 顯示 messages。
5. 重要 file I/O operations 必須使用 try-except。
6. Errors 應包含 `error_code`、`message` 與 `context`。
7. 可行時，unexpected exceptions 應轉成 structured errors。

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

在 V0.1 中，Excel 是主要 engineering data container。

但是初始 Core API 只應處理 Excel skeleton operations。

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

V0.1 應使用 `openpyxl` 處理 Excel operations，因為它跨平台，適合 CLI、CI、Linux 與 non-GUI environments。

---

## 11. JSON Operation Boundary

JSON 是 machine-readable index layer。

它不是大量 raw data storage。

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

所有 JSON read/write operations 應透過：

- `ManifestManager`
- `WorkbookIndexManager`

CLI 與 GUI 不應直接編輯 JSON files。

---

## 12. Validation API

Validation API 應提供 structured consistency checks。

建議 validation functions：

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

Validator 應檢查：

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

CLI 與 GUI 必須使用 Core API。

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

Core API 應支援沒有真實 raw data 的 dry run workflow。

建議 dry run：

```text
1. create_project("ABC123_VALIDATION_2026")
2. add_workbook(test_group="EFF")
3. add_sheet(sheet_id="eff_001", sheet_name="EFF_001")
4. write SUMMARY sheet
5. validate project
6. list workbooks
7. list sheets
```

預期輸出：

```text
ABC123_VALIDATION_2026/
├── manifest.json
├── indexes/
│   └── ABC123_EFF.index.json
├── workbooks/
│   └── ABC123_EFF.xlsx
└── logs/
```

預期 workbook content：

```text
SUMMARY
EFF_001
```

這個 dry run 不需要真實 raw data、charts、instruments 或 hardware。

---

## 15. Acceptance Criteria

當 Core API V0.1 能做到以下項目，可視為完成：

1. 建立 ERF project folder。
2. 建立 `manifest.json`。
3. 新增 workbook registry entry。
4. 建立 workbook index JSON。
5. 建立 Excel workbook。
6. 建立 `SUMMARY` sheet。
7. 建立 test condition sheet。
8. 將 sheet entry 寫入 workbook index JSON。
9. 將 sheet entry 同步到 `SUMMARY` sheet。
10. 驗證 project structure 與 consistency。
11. 回傳 unified `ERFResult` 或 `ValidationResult`。
12. 以 pytest 覆蓋 create/load/save/validate workflows。
13. 提供未來 CLI 可直接呼叫的 APIs。
14. 提供未來 GUI 可直接呼叫的 APIs。

---

## 16. Non-Goals

ERF Core API V0.1 不包含：

1. Full raw data table schema。
2. Real hardware data acquisition。
3. Full report template system。
4. GUI implementation。
5. AI summary generation。
6. PDF export。
7. PowerPoint export。
8. HTML dashboard export。
9. Database backend。
10. External assets storage。
11. Fixed compare profile。
12. Complex Excel styling。
13. Advanced chart generation。
14. Production-grade plugin system。

---

## 17. Final Summary

Core API 是 ERF 的基礎。

所有未來 interfaces 都應依賴它：

```text
AI / CLI / GUI / automation
          ↓
      Core API
          ↓
JSON index + Excel workbook + file system
```

這讓 ERF 可以安全地從 project/index management 成長到 mock data workflow、report assembly、GUI review 與 AI-assisted engineering reporting。
