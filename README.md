# WinSol Web Automation

An enterprise-grade, URL + Excel automation framework using **Playwright**, **TypeScript**, and **Claude Code (Agent)**. 

This framework reads test cases directly from the master UAT Excel workbook, drives the WinSol Web Portals using semantic UI locators, and writes the execution results (PASS/FAIL) back to the workbook in a single atomic operation.

---

## 🏗️ Architecture & Workflow

1. **Read:** `excelReader.ts` reads Sheets 2–7 and captures the exact Excel row number for every test case.
2. **Execute:** Playwright opens the browser, logs in as the appropriate role, and executes the UI automation.
3. **Stage:** Results are held in memory (zero disk I/O) during the test run to prevent file lock corruption.
4. **Flush:** At the very end of the suite, `excelWriter.ts` performs **one single atomic write** back to the workbook, verifying row identity before writing.

---

## 📁 Directory Structure

```text
winsol-web-automation/
├── .env                        # Real credentials (Gitignored)
├── .env.example                # Template for credentials
├── .gitignore                  # Excludes node_modules, reports, .env
├── CLAUDE.md                   # Agent rules & locator guidelines
├── PROMPT.md                   # The prompt used to generate this framework
├── package.json                # Dependencies and NPM scripts
├── tsconfig.json               # TypeScript configuration
├── playwright.config.ts        # Playwright runner config
├── global-setup.ts             # Creates folders and validates Excel file lock
│
├── test-data/                  # Contains WinSol_7_Automation_Sheets.xlsx
├── utils/
│   ├── constants.ts            # Sheet names, column letters, role mapping
│   ├── excelReader.ts          # Reads test cases & captures exact row numbers
│   └── excelWriter.ts          # In-memory batch writer + atomic flush
├── pages/                      # Page Object Models (POMs)
│   ├── loginPage.ts            # Shared authentication
│   ├── btlReviewPage.ts        # Stage 1: Branch Team Leader
│   ├── broRiskPage.ts          # Stage 2: Branch Risk Officer
│   ├── bmApprovalPage.ts       # Stage 3: Branch Manager
│   ├── analysisPortalPage.ts   # Stage 4: Analysis Team (RUM & BS)
│   ├── hqCommitteePage.ts      # Stage 5: HQ Credit Committee
│   └── bfoAuditPage.ts         # Stage 6: Branch Finance Officer
├── tests/
│   ├── _fixtures.ts            # Custom Playwright fixture for role-based login
│   ├── smoke.spec.ts           # Validates URL reachability
│   └── excel-driven.spec.ts    # Master runner (502 web cases)
├── scripts/
│   └── verify-workbook.ts      # Post-run integrity check
│
├── reports/                    # Auto-generated HTML reports (Gitignored)
└── screenshots/                # Auto-captured failure screenshots (Gitignored)
