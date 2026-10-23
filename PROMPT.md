# Claude Code Prompt — WinSol URL + Excel Automation

Project: D:\winsol-web-automation
Fill the 20 empty files with working Playwright + Excel automation code.

FILE LIST TO FILL:
tsconfig.json, playwright.config.ts, global-setup.ts,
utils/constants.ts, utils/excelReader.ts, utils/excelWriter.ts,
pages/loginPage.ts, pages/btlReviewPage.ts, pages/broRiskPage.ts,
pages/bmApprovalPage.ts, pages/analysisPortalPage.ts,
pages/hqCommitteePage.ts, pages/bfoAuditPage.ts,
tests/_fixtures.ts, tests/smoke.spec.ts, tests/excel-driven.spec.ts,
scripts/verify-workbook.ts, CLAUDE.md, README.md

CRITICAL RULES (do not skip):

1. CommonJS — no "type": "module" in package.json (already correct)
2. excelReader.ts:
   - sheet_to_json(sheet, { range: 3, defval: '' })
   - .map((row, idx) => ({ row, excelRow: idx + 4 }))  ← BEFORE filter
   - .filter TC- prefix
   - return { sheetName, testCaseId, module, testTitle, testSteps, testData,
              expectedResult, excelRow }
   - excelRow computed ONLY here, never in writer
3. excelWriter.ts:
   - in-memory Map<string, ExecutionResult>
   - stageResult(), resetPending(), pendingCount()
   - flushResults():
     - read xlsx once
     - for each pending: assert A{excelRow} === testCaseId (else skip + collect failed[])
     - write J/N/O/P/Q
     - atomic write: tmp file + fs.renameSync, 3x retry on EBUSY
     - return { written, failed }
   - ExecutionResult MUST include excelRow
4. Date: new Date().toISOString().slice(0, 10) — NEVER split('T')
5. POMs: goto(route) + assertRendered() with real expect() (heading visible,
   table visible, table row count > 0)
6. Test names: `[${tc.testCaseId}] ${tc.testTitle}` for --grep
7. excel-driven.spec.ts:
   - module-level const allCases = readAllSheets()
   - beforeAll: resetPending()
   - afterAll: flushResults() + log written/failed
   - HANDLERS map: sheetKey → POM.goto() + POM.assertRendered()
   - try/catch/finally: stageResult in finally (in-memory only)
   - screenshot: set remarks only AFTER screenshot resolves
8. playwright.config.ts:
   - dotenv.config(); throw if !WEB_PORTAL_URL
   - globalSetup ./global-setup
   - workers: 1, fullyParallel: false
   - headless: process.env.HEADED !== 'true'
9. tsconfig.json: ES2022, commonjs, moduleResolution node, strict, types ["node"]
10. Sheet names (exact):
    "2. BTL & DBM (Review)"
    "3. BRO (Advisory Risk Review)"
    "4. BM (Decision & Escalation)"
    "5. Analysis Team (RUM & BS)"
    "6. HQ Committee (Decision)"
    "7. BFO (Read-Only Audit)"

AFTER FILLING:
Run: npx tsc --noEmit
Report any errors.

## Prompt to paste into Claude Code