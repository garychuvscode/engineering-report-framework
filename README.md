# Engineering Report Framework (ERF)

Engineering Report Framework (ERF) is an open-source framework for generating engineering validation reports, automation test reports, manufacturing reports, and long-term data analysis reports.

The goal is to provide a reusable reporting pipeline that transforms raw test data into professional reports with traceability, visualization, and automated analysis.

---

## Current Development Status

**Status: Spec and design phase**

ERF is currently in the architecture and planning stage. The repository already includes public specifications for the IC validation data index model, development flow, and Core API design.

The next implementation milestone is **Phase 1 - Core API Foundation**, which will create the Python package skeleton, project/index managers, Excel skeleton generator, SUMMARY sheet writer, and validator.

See the full roadmap here:

- [ROADMAP.md](ROADMAP.md)

---

## Documentation

### Specifications

- [ERF IC Validation Data Index Spec V0.1 - English](docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_EN.md)
- [ERF IC Validation Data Index Spec V0.1 - Chinese](docs/spec/ERF_IC_VALIDATION_DATA_INDEX_SPEC_V0_1_ZH.md)
- [ERF Core API Spec V0.1 - English](docs/spec/ERF_CORE_API_SPEC_V0_1_EN.md)
- [ERF Core API Spec V0.1 - Chinese](docs/spec/ERF_CORE_API_SPEC_V0_1_ZH.md)

### Design Documents

- [ERF Design Flow V0.1 - English](docs/design/DESIGN_FLOW_V0_1_EN.md)
- [ERF Design Flow V0.1 - Chinese](docs/design/DESIGN_FLOW_V0_1_ZH.md)

---

## Scope

Engineering Report Framework is designed to support a wide range of engineering reporting workflows, including:

### Validation Reports

- IC Validation Reports
- Electrical Validation Reports
- Sensor Validation Reports
- System Validation Reports
- Reliability Validation Reports

### Automation Test Reports

- Production Test Reports
- Automated Verification Reports
- Regression Test Reports
- Functional Test Reports
- Manufacturing Test Reports

### Engineering Analysis Reports

- Characterization Reports
- Data Comparison Reports
- Long-Term Trend Analysis
- Golden Sample Comparison
- Failure Investigation Reports

### Future Report Formats

- Excel Reports
- PDF Reports
- PowerPoint Reports
- HTML Reports
- Interactive Dashboards

---

## Vision

Most engineering teams spend significant effort manually organizing test data, generating reports, formatting spreadsheets, and comparing historical results.

Engineering Report Framework aims to provide an open-source reporting engine that allows engineers to focus on engineering rather than report formatting.

The framework is designed to:

- Standardize engineering reports
- Improve report reusability
- Reduce manual effort
- Increase traceability
- Support automated report generation
- Enable long-term data comparison
- Build a common reporting framework for engineering teams

---

## Target Users

- Validation Engineers
- Test Engineers
- Hardware Engineers
- Firmware Engineers
- QA Teams
- Manufacturing Engineers
- Automation Developers
- Lab Engineers

---

## Architecture Concept

```text
Raw Test Data
      │
      ▼
Data Object Layer
      │
      ▼
Analysis Layer
      │
      ▼
Report Generator
      │
      ▼
Excel / PDF / PPT / HTML
```

The framework separates:

- Data Collection
- Data Storage
- Data Analysis
- Report Generation

allowing each layer to evolve independently.

---

## Development Roadmap Summary

ERF will be developed in phases so the project can move forward even before the real hardware data acquisition system is mature.

### Phase 0 - Spec Baseline

- [x] IC validation data index spec
- [x] Design flow document
- [x] Core API spec

### Phase 1 - Core API Foundation

- [ ] Python package skeleton
- [ ] Project manager
- [ ] Manifest manager
- [ ] Workbook index manager
- [ ] Excel workbook builder
- [ ] SUMMARY sheet writer
- [ ] Project validator
- [ ] Basic pytest coverage

### Phase 2 - CLI for AI and Automation

- [ ] `erf create-project`
- [ ] `erf add-workbook`
- [ ] `erf add-sheet`
- [ ] `erf list-workbooks`
- [ ] `erf list-sheets`
- [ ] `erf validate`
- [ ] JSON output mode for AI agents

### Phase 3 - Mock Data and Demo Workflow

- [ ] Mock IC validation project
- [ ] Mock efficiency data
- [ ] Mock waveform data
- [ ] Excel locator update
- [ ] Demo project documentation

### Phase 4 - Human Review GUI

- [ ] Minimal ERF project browser
- [ ] Manifest and workbook index viewer
- [ ] Condition filtering
- [ ] Dataset locator viewer
- [ ] Validation result viewer

### Phase 5 - Report Assembly and AI Assistance

- [ ] Dataset selection workflow
- [ ] Report draft generation
- [ ] AI-assisted summary and review workflow

For details, see [ROADMAP.md](ROADMAP.md).

---

## Integration Targets

- Raspberry Pi
- Raspberry Pi Pico
- Automated Test Fixtures
- Laboratory Equipment
- Manufacturing Test Systems
- SCPI Instruments
- Custom Python Test Frameworks

---

## License

MIT License
