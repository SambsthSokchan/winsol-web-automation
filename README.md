###WinSol Web Automation


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
```markdown
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
```

---

## 🚀 Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/winsol-web-automation.git
cd winsol-web-automation
```

### 2. Configure Environment Variables
Copy the example file and fill in your credentials:
```bash
cp .env.example .env
```
Open `.env` and set your `WEB_PORTAL_URL` and the passwords for all 6 roles.

### 3. Install Dependencies
```bash
npm install
npx playwright install chromium
```

### 4. Add the Workbook
Place your `WinSol_7_Automation_Sheets.xlsx` file inside the `test-data/` folder. 

---

## 🏃‍♂️ Running the Tests

The framework is configured to allow you to run the entire suite, or test a specific stage/role.

### Run the Complete Pipeline
This runs a quick smoke test, executes all 502 web cases, and verifies the workbook integrity at the end.
```bash
npm test
```

### Run Specific Stages
You can target individual stages using the test IDs (e.g., `TC-BTL` for Stage 1).
```bash
npm run test:btl       # Stage 1 (112 cases)
npm run test:bro       # Stage 2 (85 cases)
npm run test:bm        # Stage 3 (61 cases)
npm run test:analysis  # Stage 4 (117 cases)
npm run test:hq        # Stage 5 (91 cases)
npm run test:bfo       # Stage 6 (36 cases)
```

### Debugging visually
By default, tests run in headless mode (invisible). To watch the browser as it works, set the `HEADED` environment variable:
```bash
# Windows CMD
set HEADED=true && npm run test:btl

# Mac/Linux
HEADED=true npm run test:btl
```

### Pre-flight Checks
Verify that your workbook structure and row mappings are correct before running tests:
```bash
npm run verify:workbook
```

---

## 🤖 Claude Code Integration

This project is designed to be maintained and expanded by Claude Code. 
If you need to add new POM actions or write new tests, use the Playwright MCP server so Claude can explore the live page.

To authorize Claude:
```bash
claude mcp add playwright npx @playwright/mcp@latest --caps=testing
```

Refer to `CLAUDE.md` for the strict locator rules (Semantic locators only, no XPath, no hard waits).

---

## 🛡️ Reliability Features

- **Row-Identity Protection:** The writer verifies that the Excel row it is about to write to actually belongs to the test case ID before writing. If the sheet is modified mid-run, it skips the row rather than corrupting data.
- **Atomic Writes:** Test results are never written one by one. They are staged in RAM and flushed in one single atomic operation to prevent workbook corruption.
- **Fail-Fast Setup:** `global-setup.ts` will halt execution immediately if the Excel workbook is currently open in Microsoft Excel (prevents `EBUSY` file lock errors).
