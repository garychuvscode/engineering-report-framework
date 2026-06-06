# ERF IC 驗證資料索引規格 V0.1

## 1. 文件目的

本文件定義 Engineering Report Framework，簡稱 ERF，在 V0.1 階段針對 **IC 驗證資料管理** 的基礎索引架構。

ERF V0.1 的重點不是先定義每一種 raw data table 的欄位格式，而是先建立一套可擴充的資料組織方式，讓 IC 驗證資料可以被清楚地分類、索引、查找與後續分析。

本階段的核心概念是：

```text
Excel workbook = 工程資料本體
JSON index     = 程式可讀的資料地圖
SUMMARY sheet  = 人工可讀的資料目錄
```

也就是說，工程師實際打開、查看、整理與交付的資料會放在 Excel 裡；而程式需要快速知道資料在哪裡、對應什麼條件、屬於哪個測試項目時，則會透過 JSON index 查找。

---

## 2. V0.1 設計目標

ERF V0.1 的設計目標如下：

1. 以 Excel 作為 IC 驗證資料的主要容器。
2. 以 JSON 作為資料結構、測試條件與位置索引。
3. 避免程式為了查找資料而頻繁開啟 Excel。
4. 讓工程師手動打開 Excel 時，也能透過 `SUMMARY` sheet 快速理解檔案內容。
5. 在 V0.1 階段先把主要資料都放在 Excel 裡，不額外使用 `assets/` 資料夾。
6. 支援 IC 驗證中複雜的測試條件，例如 VIN、VOUT、IOUT、Temp、Mode、Trim code、BOM、EVB revision 等。
7. 為未來的自動化報告產生、資料回顧、歷史比較、AI summary 打好基礎。
8. 先專注在索引架構，不急著定義所有 raw data 的內容格式。

---

## 3. 核心設計原則

ERF V0.1 採用以下原則：

```text
Excel workbook = 工程資料本體
JSON index     = 程式可讀的資料地圖
SUMMARY sheet  = 人工可讀的資料目錄
```

Excel workbook 負責存放實際工程資料。

JSON index 負責描述資料在哪裡、屬於什麼條件，以及如何識別。

SUMMARY sheet 讓工程師直接打開 Excel 時，也可以不用閱讀 JSON 就理解 workbook 內容。

---

## 4. V0.1 儲存模式

ERF V0.1 採用：

```json
{
  "storage_mode": "excel_contained"
}
```

意思是：

- raw data table 放在 Excel。
- processed table 放在 Excel。
- chart 放在 Excel。
- waveform table 放在 Excel。
- embedded image 放在 Excel。
- JSON 只存 metadata 與 locator。
- V0.1 不需要外部 `assets/` 資料夾。

未來版本可支援：

```json
{
  "storage_mode": "hybrid"
}
```

但這不屬於 V0.1 範圍。

---

## 5. 專案資料夾架構

標準 IC validation project 建議使用以下結構：

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

### 5.1 各資料夾角色

| Folder / File | Role |
|---|---|
| `manifest.json` | 專案層級總索引，只存高階 project 與 workbook 對應資訊。 |
| `indexes/` | 每一個 workbook 對應一個 JSON index file。 |
| `workbooks/` | 存放 Excel workbook。每個 workbook 代表一個主要測試大項。 |
| `logs/` | 存放 index 產生、更新、驗證或錯誤紀錄。 |

---

## 6. Project-Level Architecture

高階專案結構如下：

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

一個 project 可以包含多個 test group workbook。

例如：

```text
ABC123_VALIDATION_PROJECT
│
├── ABC123_EFF.xlsx
├── ABC123_LOAD_REG.xlsx
├── ABC123_LINE_REG.xlsx
└── ABC123_TRANSIENT.xlsx
```

每個 workbook 專注在一個主要 validation category。

---

## 7. Workbook 命名規則

一個 workbook 應代表同一顆 IC 的一個主要測試大項。

建議命名格式：

```text
{part_number}_{test_group}.xlsx
```

例如：

```text
ABC123_EFF.xlsx
ABC123_LOAD_REG.xlsx
ABC123_LINE_REG.xlsx
ABC123_TRANSIENT.xlsx
ABC123_ACCURACY.xlsx
```

workbook 檔名應保持簡潔。詳細測試條件不應該編碼到 workbook filename 中。

---

## 8. JSON 檔案命名規則

每一個 workbook 都應有一個對應的 index JSON file。

建議命名格式：

```text
{part_number}_{test_group}.index.json
```

例如：

```text
ABC123_EFF.index.json
ABC123_LOAD_REG.index.json
ABC123_LINE_REG.index.json
```

index JSON 應描述對應 Excel workbook 的內部結構。

---

## 9. Workbook 內部結構

每個 workbook 應包含：

```text
ABC123_EFF.xlsx
├── SUMMARY
├── EFF_001
├── EFF_002
├── EFF_003
├── EFF_001_WF_001
└── EFF_002_WF_001
```

### 9.1 Sheet 類型

| Sheet Type | Example | Purpose |
|---|---|---|
| Summary sheet | `SUMMARY` | 人工可讀 workbook index。 |
| Main test condition sheet | `EFF_001` | 存放某一組測試條件的資料、chart、image 與 notes。 |
| Waveform sheet | `EFF_001_WF_001` | 存放與 main test sheet 相關的大量 waveform table。 |
| Other support sheet | Optional | 視需求存放輔助資料。 |

---

## 10. Sheet 命名規則

Excel worksheet name 應保持簡短且穩定。

建議命名格式：

```text
{test_group}_{serial_number}
```

例如：

```text
EFF_001
EFF_002
LOAD_REG_001
LINE_REG_001
```

waveform 相關 sheet 建議：

```text
{parent_sheet_name}_WF_{serial_number}
```

例如：

```text
EFF_001_WF_001
EFF_001_WF_002
```

完整測試條件不應存放在 sheet name 中。完整條件應存放在 workbook index JSON 中。

---

## 11. Stable ID 規則

ERF 不應只依賴 filename 或 Excel sheet name。

以下 ID 應被使用：

| ID | Purpose | Example |
|---|---|---|
| `project_id` | 穩定 project identifier | `ABC123_VALIDATION_2026` |
| `workbook_id` | 穩定 workbook identifier | `wb_abc123_eff` |
| `sheet_id` | 穩定 sheet identifier | `eff_001` |
| `dataset_id` | 穩定 dataset identifier | `ds_eff_001_main` |

這樣即使 filename 或 sheet name 變更，ERF 仍能維持穩定的內部 reference。

---

## 12. Excel 與 JSON 責任分工

### 12.1 Excel 存放內容

Excel 應存放：

- Summary sheet
- Main measurement tables
- Processed tables
- Waveform tables
- Charts
- Embedded images
- Human-readable notes
- Manual review comments

### 12.2 JSON 存放內容

JSON 應存放：

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

### 12.3 JSON 不應存放內容

V0.1 中 JSON 不應存放大量 raw data。

應避免存放：

- 完整 raw measurement tables
- 完整 waveform data
- Image binary data
- 大量 chart data arrays

JSON 應該指向它們在 Excel 中的位置。

---

## 13. manifest.json

`manifest.json` 是 project-level index。

它只應存放高階 project information 與 workbook index file references。

它不應存放詳細 sheet information。

### 13.1 manifest.json 範例

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

每個 workbook 應有一個對應的 index JSON file。

例如：

```text
workbooks/ABC123_EFF.xlsx
indexes/ABC123_EFF.index.json
```

Workbook index JSON 描述：

- 它對應哪個 workbook
- workbook 內有哪些 sheets
- 每個 sheet 代表什麼
- 每個 sheet 對應什麼測試條件
- 使用了什麼 IC configuration
- 使用了什麼 hardware configuration
- datasets 位於 Excel 的哪裡

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

`workbook_info` 描述目標 Excel workbook。

### 16.1 範例

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

建議值：

| Value | Meaning |
|---|---|
| `valid` | index 與 workbook 一致。 |
| `stale` | workbook 可能在 index 產生後被修改。 |
| `needs_reindex` | index 應被重新產生。 |
| `error` | index validation failed。 |

---

## 17. sheet_registry

`sheet_registry` 是 workbook index JSON 的核心。

它將每個 test condition sheet 對應到 metadata、profiles 與 dataset locators。

### 17.1 範例

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

`condition_profile` 存放操作條件與量測設定條件。

它應描述 DUT 在測試過程中如何被操作。

### 18.1 應包含

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

### 18.2 範例

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

`ic_config_profile` 存放 IC 內部設定資訊。

此 profile 應用於可能影響 validation result 的設定。

### 19.1 應包含

- Trim code
- Compensation setting
- Register setting
- Mode configuration
- OTP / NVM setting
- I2C / SPI register snapshot
- Configuration source
- Configuration time

### 19.2 範例

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

`hardware_profile` 存放外部硬體與 board-level information。

此 profile 應用於可能影響 measurement result 的硬體設定。

### 20.1 應包含

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

### 20.2 範例

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

`extra_info_profile` 存放無法歸類到主要標準 profiles 的補充資訊。

此欄位應謹慎使用。

### 21.1 使用規則

能放進標準 profile 的資訊，應優先放入標準 profile。

只有以下情境才使用 `extra_info_profile`：

- 資訊是 project-specific
- 欄位尚未有標準分類
- 資訊主要作為 notes 或 review 使用
- 資訊不是核心操作條件
- 資訊不是 IC configuration
- 資訊不是 hardware configuration

### 21.2 不應放入的資訊

| Information | Correct Profile |
|---|---|
| VIN / VOUT / IOUT / Temperature / Mode | `condition_profile` |
| Trim code / register setting / compensation | `ic_config_profile` |
| BOM / EVB revision / external components | `hardware_profile` |
| Excel range / chart / image location | `dataset_locators` |

### 21.3 範例

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

`dataset_locators` 描述 datasets 在 Excel workbook 內的位置。

Dataset locator 不是資料本身。

它只回答：

```text
Where is this dataset located inside Excel?
```

Dataset locators 可讓 ERF 定位：

- Main measurement tables
- Processed tables
- Waveform tables
- Charts
- Embedded images
- Notes or supporting tables

---

## 23. Dataset Locator Types

ERF V0.1 應支援以下 locator types：

| Locator Type | Purpose |
|---|---|
| `excel_range` | 定位 worksheet 內的 table 或 range。 |
| `excel_table` | 透過 table name 定位 Excel table。 |
| `excel_chart` | 定位 worksheet 內的 chart。 |
| `excel_image` | 定位 worksheet 內的 embedded image。 |
| `excel_sheet` | 定位整個 worksheet。 |

---

## 24. excel_range Locator

用於定位儲存在 cell range 中的 table-like data。

### 範例

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

用於定位 Excel 內的 chart。

### 範例

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

用於定位 Excel 內的 embedded images。

### 範例

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

V0.1 中 waveform data 應保留在 Excel 內。

若 waveform data 較大，應放在獨立 waveform sheet，但仍存在同一個 workbook 中。

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

`optional_tags` 可用於 human-readable labels。

它不是必要欄位。

它不應取代 structured profiles。

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

不要使用 tags 存放 structured conditions：

```json
{
  "optional_tags": [
    "VIN_5V",
    "TEMP_25C",
    "BOM_A"
  ]
}
```

這些應該存放在：

- `condition_profile`
- `hardware_profile`
- `ic_config_profile`

---

## 29. SUMMARY Sheet

每個 workbook 必須包含一個 `SUMMARY` sheet。

`SUMMARY` sheet 是 workbook index JSON 的 human-readable mirror。

它讓工程師打開 Excel 後可以快速理解：

- 有哪些 sheets
- 每個 sheet 的用途
- 每個 sheet 對應什麼條件
- raw data 位於哪裡
- charts/images/waveforms 位於哪裡
- 是否存在特殊 notes

---

## 30. Recommended SUMMARY Sheet Columns

`SUMMARY` sheet 應包含以下欄位：

| Column | Description |
|---|---|
| `sheet_id` | ERF stable sheet ID。 |
| `sheet_name` | 實際 Excel worksheet name。 |
| `sheet_role` | Main test condition、waveform sheet、support sheet 等。 |
| `display_name` | 人類可讀的測試描述。 |
| `condition_summary` | 簡短條件摘要。 |
| `config_id` | IC configuration ID。 |
| `hardware_id` | Hardware profile ID。 |
| `dataset_count` | 與此 sheet 連結的 dataset 數量。 |
| `main_data_range` | 主要 raw data range。 |
| `chart_range` | 主要 chart location。 |
| `image_range` | 主要 image location。 |
| `notes` | 人工可讀 notes。 |

---

## 31. SUMMARY and JSON Relationship

Workbook index JSON 是 machine-readable source index。

SUMMARY sheet 是 human-readable mirror index。

建議規則：

```text
Workbook index JSON = machine-readable index
SUMMARY sheet       = human-readable mirror
```

當 workbook index JSON 更新時，SUMMARY sheet 也應同步更新。

如果 Excel 被人工修改，ERF 未來應提供 re-index function 以重建或驗證 JSON index。

---

## 32. Index Synchronization

因為 Excel files 可能被人工修改，ERF 應追蹤 index status。

每個 workbook index JSON 應包含：

```json
{
  "index_status": "valid",
  "last_index_time": "2026-06-04T10:30:00+08:00",
  "workbook_checksum": "optional_sha256"
}
```

未來 implementation 可檢查：

- workbook 是否存在
- sheet names 是否仍存在
- ranges 是否仍存在
- required sheets 是否存在
- SUMMARY 是否與 JSON 一致
- workbook checksum 是否改變

---

## 33. Required Fields

### 33.1 manifest.json 必要欄位

- `schema_version`
- `index_type`
- `storage_mode`
- `project_info.project_id`
- `project_info.project_type`
- `ic_info.part_number`
- `index_registry`

### 33.2 Workbook Index JSON 必要欄位

- `schema_version`
- `index_type`
- `storage_mode`
- `project_ref`
- `workbook_info.workbook_id`
- `workbook_info.file_path`
- `workbook_info.test_group`
- `workbook_info.summary_sheet_name`
- `sheet_registry`

### 33.3 每個 Sheet Entry 必要欄位

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

ERF V0.1 不定義：

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

這些可於未來版本加入。

---

## 35. Comparison Policy

ERF V0.1 不定義固定的 `compare_profile`。

原因：

IC validation 的比較規則可能取決於使用者需求、專案要求、tolerance assumptions 與工程判斷。

因此，ERF V0.1 的重點是保存足夠完整的 structured information，讓使用者或未來工具自行判斷 datasets 是否可比較。

以下 profiles 提供未來比較所需的上下文：

- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `extra_info_profile`
- `dataset_locators`

---

## 36. Implementation Direction for Codex App

第一階段 implementation 應專注於 index foundation，而不是 report styling。

建議初始任務：

1. 建立 ERF project folder structure。
2. 產生 `manifest.json`。
3. 產生 workbook-level index JSON。
4. 建立 Excel workbook。
5. 建立 `SUMMARY` sheet。
6. 使用穩定 `sheet_id` 建立 test condition sheets。
7. 將 sheet mapping 寫入 workbook index JSON。
8. 將 human-readable sheet mapping 寫入 `SUMMARY`。
9. 加入 basic validation 檢查：
   - workbook file exists
   - index JSON exists
   - required fields exist
   - SUMMARY sheet exists
   - sheet names in JSON exist in workbook

---

## 37. Acceptance Criteria for V0.1

當 ERF 能做到以下項目時，可視為 V0.1 完成：

1. 建立新的 IC validation project。
2. 建立 project-level `manifest.json`。
3. 建立至少一個 workbook index JSON。
4. 建立至少一個 Excel workbook。
5. 在 workbook 中建立 `SUMMARY` sheet。
6. 建立至少一個 test condition sheet。
7. 在 JSON 中保存 condition/config/hardware/extra profiles。
8. 在 JSON 中保存 dataset locators。
9. 將 readable summary table 寫入 Excel。
10. 驗證 JSON 與 Excel 的 basic consistency。

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

ERF V0.1 將工程資料保留在 Excel 中，並使用 JSON 描述結構、語意與位置。

這為未來的 automation test report generation、IC validation recap 與 engineering report workflows 提供乾淨的基礎。
