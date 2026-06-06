# ERF Design Flow V0.1 中文版

## 1. 文件目的

本文件定義 Engineering Report Framework，簡稱 ERF，在 V0.1 階段的初始開發流程。

目前 ERF spec 已經定義了 IC validation workflow 的上層資料模型。但是實際的資料採集系統、硬體治具、儀器控制層與完整 raw data pipeline 尚未成熟。

因此，第一階段的開發目標不應該是立刻建立完整 end-to-end report generator，而是先建立一個可靠的基礎，讓 ERF 可以建立、索引、驗證、瀏覽，並在未來整理工程報告資料。

整體開發方向是：

```text
Python Core first
CLI for AI and automation second
GUI for human review later
```

核心原則是：

```text
Core Python package = 執行基礎
CLI                 = AI / automation 入口
GUI                 = human review and report assembly interface
```

---

## 2. 設計理念

ERF 不應該把核心邏輯鎖在 GUI 裡。

所有重要行為應該先實作在可重用的 Python modules 中。CLI 與 GUI 都應該呼叫同一套 core modules。

這樣 ERF 才能支援：

- Codex app 實作
- AI agent 操作
- Batch automation
- Test automation
- 未來 g-Pico integration
- 未來 Tkinter GUI 操作
- 未來 report assembly workflow

在 AI 時代，GUI 仍然有價值，但它的角色應該是 human-in-the-loop review、filtering、validation 與 report assembly，而不是系統核心。

---

## 3. 開發階段

ERF V0.1 建議依照以下階段開發：

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

每個 phase 都應該有明確的目的、dry run 方式與驗收標準。

---

## 4. Phase 0 - Spec Baseline

### 目標

建立 ERF V0.1 的基礎資料架構規格。

目前已存在的 spec 文件：

```text
docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md
docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_ZH.md
```

### 範圍

Phase 0 定義：

- Excel 是工程資料本體
- JSON 是機器可讀索引
- SUMMARY sheet 是人工可讀索引
- Workbook 對應 test group
- Sheet 對應 test condition group
- Dataset locator 定位 Excel 內部資料位置
- V0.1 non-goals 與邊界

### Dry Run

人工 review spec 並確認：

- Project folder structure 合理
- `manifest.json` 角色清楚
- Workbook index JSON 角色清楚
- Excel / JSON 責任分工清楚
- V0.1 目前不需要定義詳細 raw data table 是合理的

### 驗收標準

- English spec 存在
- Chinese spec 存在
- 兩份 spec 語意一致
- V0.1 scope 與 non-goals 清楚
- 後續 implementation tasks 可以由 spec 推導出來

---

## 5. Phase 1 - Python Core Foundation

### 目標

建立 ERF 初始 Python package foundation。

此階段不實作完整 report generation、GUI 或硬體資料採集。

### 建議 Package Structure

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

### 設計規則

Core layer 應該：

- 不依賴 GUI
- 不依賴真實硬體
- 不依賴真實 raw data
- 可被 CLI、GUI、AI agents 與 tests 共用
- 提供穩定 Python APIs

### 沒有 Raw Data 時的 Dry Run

建立一個空的 IC validation project：

```text
ABC123_VALIDATION/
├── manifest.json
├── indexes/
├── workbooks/
└── logs/
```

確認 Python core 可以建立、儲存與讀取這個基本結構。

### 驗收標準

- 可以建立 ERF project folder
- 可以產生基本 `manifest.json`
- 可以重新載入 `manifest.json`
- 可以新增 workbook registry entry
- 可以建立 workbook index JSON object
- JSON objects 可以 serialize / deserialize
- 基本 unit tests 可以通過

---

## 6. Phase 2 - Project / Index / Excel Skeleton Generator

### 目標

根據 V0.1 spec 產生完整 ERF project skeleton。

### 主要功能

Implementation 應提供類似以下功能：

```text
create_project()
add_workbook()
create_workbook_index()
create_excel_workbook()
create_summary_sheet()
add_test_condition_sheet()
write_summary_table()
```

### Dry Run 範例

使用 mock IC project：

```text
part_number = ABC123
project_id = ABC123_VALIDATION_2026
test_group = EFF
sheet_id = eff_001
sheet_name = EFF_001
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

Excel workbook 至少應包含：

```text
SUMMARY
EFF_001
```

### 沒有 Raw Data 時的驗證

Test condition sheet 可以是空 sheet 或 placeholder。

Workbook index JSON 應包含：

- `condition_profile`
- `ic_config_profile`
- `hardware_profile`
- `extra_info_profile`
- `dataset_locators`

此階段 `dataset_locators` 可以先是空 list。

### 驗收標準

- 可以產生完整 ERF project folder
- `manifest.json` 符合 spec
- Workbook index JSON 符合 spec
- Excel workbook 存在
- Excel workbook 有 `SUMMARY` sheet
- Excel workbook 至少有一個 test condition sheet
- SUMMARY sheet 內容能對應 JSON `sheet_registry`
- Dry run 不需要任何真實 raw data

---

## 7. Phase 3 - Validator and Dry Run Demo

### 目標

建立 JSON index files 與 Excel workbooks 之間的一致性驗證能力。

此階段很重要，因為 Excel workbooks 未來可能會被人工修改。

### Validator 應檢查

- `manifest.json` 是否存在
- `manifest.json` required fields 是否存在
- `index_registry` 指向的 index JSON files 是否存在
- `workbook_path` 指向的 Excel workbooks 是否存在
- Workbook index JSON required fields 是否存在
- `SUMMARY` sheet 是否存在
- `sheet_registry` 中的 sheet_name 是否存在於 workbook
- Dataset locator range format 對 V0.1 來說是否合理
- `index_status` 是否為支援值

### Dry Run

使用 Phase 2 產生的 project：

```text
erf validate ABC123_VALIDATION_2026/
```

然後刻意建立錯誤情境：

- 刪除 workbook
- 刪除 `SUMMARY` sheet
- 手動改掉 sheet name
- 移除 required JSON field
- 提供錯誤 range string

### 沒有 Raw Data 時的驗證

Validator 不需要真實 measurement data。

它只檢查 project structure、files、sheets、required fields 與 locator format。

### 驗收標準

- 正常 mock project validation pass
- 缺 workbook 時 validation fail
- 缺 index JSON 時 validation fail
- 缺 `SUMMARY` sheet 時 validation fail
- JSON 中記錄的 sheet 不存在時 validation fail
- Required field 缺少時 validation fail
- Validation result 可以以 dict / JSON / console text 回傳

---

## 8. Phase 4 - CLI for AI and Automation

### 目標

建立 CLI interface，讓 AI agents、Codex app、batch scripts 與 automation systems 可以操作 ERF。

### 初始 CLI Commands

建議 commands：

```bash
erf create-project --part-number ABC123 --project-id ABC123_VALIDATION_2026

erf add-workbook --project ABC123_VALIDATION_2026 --test-group EFF

erf add-sheet --project ABC123_VALIDATION_2026 --test-group EFF --sheet-id eff_001 --sheet-name EFF_001

erf list-workbooks --project ABC123_VALIDATION_2026

erf list-sheets --project ABC123_VALIDATION_2026 --test-group EFF

erf validate --project ABC123_VALIDATION_2026
```

### 設計規則

- CLI 必須呼叫 core modules
- CLI 不應重複實作 core logic
- CLI 應支援 human-readable output
- CLI 應支援給 AI agents 使用的 `--json` output
- CLI 應使用合理 exit codes

### Dry Run

Terminal 或 Codex app 應能執行：

```text
create project
add workbook
add sheet
list workbook
list sheet
validate
```

### 沒有 Raw Data 時的驗證

CLI 先操作 metadata、index JSON 與 Excel skeleton。

不需要真實 raw data。

### 驗收標準

- CLI 可以 create project
- CLI 可以 add workbook
- CLI 可以 add sheet
- CLI 可以 list workbooks
- CLI 可以 list sheets
- CLI 可以 run validation
- CLI success returns exit code 0
- CLI failure returns non-zero exit code
- CLI 支援 AI-friendly JSON output

---

## 9. Phase 5 - Mock Data Workflow

### 目標

在真實資料採集系統就緒前，使用 deterministic mock data 驗證資料寫入、locator 更新、SUMMARY 更新與 demo workflow。

### Mock Data Generators

建議 mock generators：

```text
mock_efficiency_data
mock_waveform_data
mock_scope_image_placeholder
```

初始 mock efficiency table 可包含：

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

產生：

```text
EFF_001 sheet
  ├── condition summary
  ├── mock efficiency table
  ├── simple chart
  └── dataset locator update

EFF_001_WF_001 sheet
  └── mock waveform table
```

### 沒有真實 Raw Data 時的驗證

使用 deterministic mock data，例如固定 seed 或固定數值。

這樣測試結果可以重複。

### 驗收標準

- 可以產生 mock efficiency table
- 可以將 mock data 寫入 Excel
- 可以更新 dataset locator
- SUMMARY sheet 可以顯示 `main_data_range`
- 可以建立 waveform sheet
- mock data 插入後 validator pass
- Demo project 可以作為 repository example

---

## 10. Phase 6 - Minimal GUI for Human Review

### 目標

建立最小可用 GUI 供 human review。

第一版 GUI 應定位為 ERF Project Browser，而不是完整 report generator。

### 初始 GUI 功能

GUI 應能：

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

### 設計規則

- GUI 必須呼叫 core modules
- GUI 不應直接實作 core logic
- GUI 不應成為 CLI / AI workflows 的必要條件
- GUI 應作為 human review interface

### Dry Run

載入 Phase 5 產生的 mock project。

預期顯示 tree：

```text
ABC123_VALIDATION_2026
  └── EFF workbook
        ├── EFF_001
        └── EFF_001_WF_001
```

### 沒有真實 Raw Data 時的驗證

GUI 可讀取前面 phases 產生的 JSON 與 Excel skeleton。

不需要真實 measurement data。

### 驗收標準

- GUI 可以 load project folder
- GUI 可以顯示 workbook registry
- GUI 可以顯示 sheet registry
- GUI 可以顯示 condition summary
- GUI 可以顯示 dataset locators
- GUI 可以 run validator
- GUI 使用 core modules，而不是重複實作邏輯

---

## 11. Phase 7 - Report Assembly Workflow

### 目標

讓使用者可以選擇 datasets 並產生 report draft。

此階段將 ERF 從 data index tool 推進到 report assembly tool。

### 主要能力

- Query datasets
- Filter by test group
- Filter by condition profile
- Select dataset locators
- Copy or reference selected Excel ranges / charts / images
- Generate a report workbook

### Dry Run

使用 mock data project 產生：

```text
ABC123_SELECTED_REPORT.xlsx
```

預期內容：

```text
Report_SUMMARY
Selected_EFF_001
Selected_Charts
Selected_Waveforms
```

### 沒有真實 Raw Data 時的驗證

使用 Phase 5 產生的 mock data。

### 驗收標準

- 可以 query datasets
- 可以 select datasets
- 可以 create report workbook
- 可以 copy or reference selected data
- 可以 generate report summary
- report 可被人工打開 review

---

## 12. Phase 8 - AI Assisted Report Workflow

### 目標

讓 AI 透過 CLI / core 查詢 ERF 並提出 report content 建議，由人類透過 GUI 確認最終輸出。

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

### AI 時代的價值

隨著 AI 變得更強，ERF 仍然有價值，因為它提供：

- Reliable engineering data index
- Traceable dataset locations
- Structured condition/config/hardware context
- Repeatable execution interface
- Human confirmation layer

ERF 應成為 AI agents 可以信任的 engineering data map。

### 驗收標準

- CLI output 可以被 AI 解析
- Query results 可以以 JSON 回傳
- AI 可以根據 query result 推薦 datasets
- GUI 可以顯示 selection results
- 人類仍保有最後 review 權限

---

## 13. 建議後續規劃文件

在 heavy implementation 前，建議規劃以下文件。

### 13.1 Core API Spec

```text
docs/spec/ERF_CORE_API_SPEC_V0_1.md
```

用途：

- 定義 core Python classes and functions
- 避免 module naming 隨機發散
- 釐清 managers、builders、validators 與 data models 的邊界

範例 core classes：

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

用途：

- 定義 CLI commands
- 定義 parameters
- 定義 output format
- 定義 exit codes
- 定義給 AI 使用的 JSON output

### 13.3 Test Plan

```text
docs/testing/TEST_PLAN_V0_1.md
```

用途：

- 定義 unit tests
- 定義 integration tests
- 定義 mock project tests
- 定義 Excel validation tests
- 定義 CLI dry run tests

### 13.4 Demo Project Description

```text
examples/ic_validation_demo/README.md
```

用途：

- 說明 demo project 應產生什麼
- 提供 open-source users 可見的入口
- 展示 ERF 在沒有真實硬體時如何運作

### 13.5 AI Coding Rules

```text
AGENTS.md
```

用途：

- 指引 Codex / AI coding agents
- 定義 development rules
- 避免 GUI-first architecture
- 要求 CLI 與 GUI 都呼叫 core modules
- 要求新增功能需有 tests
- 提醒 agents 不要將大量 raw data 存進 JSON

---

## 14. 開發優先順序摘要

建議順序：

```text
1. DESIGN_FLOW_V0_1.md
2. ERF_CORE_API_SPEC_V0_1.md
3. ERF_CLI_SPEC_V0_1.md
4. TEST_PLAN_V0_1.md
5. AGENTS.md
6. Phase 1 implementation by Codex app
```

---

## 15. 總結

ERF V0.1 implementation 不應從 GUI 或完整 report generation 開始。

它應該從可重用 Python core 開始，再透過 CLI 提供給 AI 與 automation 使用，最後才提供 GUI 給人類 review 與 report assembly。

```text
Spec tells what ERF is.
Design flow tells how to build ERF.
Core API spec tells what code interface should exist.
CLI spec tells how AI should operate ERF.
Test plan tells how to know it works.
```

這個流程讓 ERF 即使在真實 raw data acquisition system 尚未成熟前，仍然可以持續推進。
