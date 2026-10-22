# APIS India Limited — Salary MIS Automation (HO & Sales)

Static web app. Upload the HO and Sales salary sheets (.xlsx), click **Submit & Build MIS**, enter the manual inputs
(Promotor JK, HRA, Hiring fees), fill the **Loan Split** tab, review **Checks**, then **Export MIS to Excel**.
Everything runs inside the user's browser — no salary data is uploaded anywhere.

## Folder contents
| File | Purpose |
|---|---|
| `index.html` | The application |
| `libs/xlsx.full.min.js` | SheetJS 0.18.5 (Excel reader) — bundled, no CDN needed |
| `libs/exceljs.min.js` | ExcelJS 4.4.0 (Excel writer) — bundled, no CDN needed |
| `_headers` | Cloudflare Pages security headers (CSP, noindex, no framing) |

## Deploy on Cloudflare Pages
1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** → **Upload assets** (direct upload).
2. Name the project (e.g. `apis-salary-mis`), then drag in **the contents of this folder** (`index.html`, `_headers`, `libs/`, `README.md`) and click **Deploy**.
3. The site is live at `https://<project>.pages.dev`.
4. **Restrict access (strongly recommended for payroll):** Cloudflare **Zero Trust → Access → Applications → Add → Self-hosted**,
   add the hostname `<project>.pages.dev` and also `*.<project>.pages.dev` (preview URLs), and allow only company email addresses.
5. **Update later:** open the project → **Create deployment** → upload the whole folder again.

Command-line alternative: `npx wrangler pages deploy . --project-name apis-salary-mis` (run inside this folder).

## Approved MIS layout (September 2026 sign-off) — what the app produces
Sheet `<Month>, <yy>` — columns B…S:
B Gross Earning (Basic…Special Allowance) · C Extra/Arrear · D Variable · E Gross Earning · F TDS · G EE PF · H EE ESIC ·
I Adv/Loan/Other Deduction · J Medical · K NPS · L HOLD · M Net Payout · N No. of Manpowers · O Reimbursement Part ·
P Total Gross · Q Total Payout · R ER PF · S ER ESIC

| Row type | Formulas |
|---|---|
| Department rows (HO, Directors, Sales) | `E = SUM(B:D)` · `M = E − SUM(F:L)` · `P = E + O` · `Q = M + O` |
| Total A / Total (A+B) | `SUM` of the block for every column, except `E = SUM(B:D)` and `Q = M + O` of the total row |
| Sales TOTAL (A) | `SUM` of the block for every column (including E and P), except `Q = M + O` |
| Promotor (JK), HRA, Hiring Fees | B (and N for JK) typed in; `E = B + C` · `M = E − SUM(F:L)` |
| Service Charges @3% | `ROUND((JK + HRA) × 3%, 0)` |
| EPF@13% | `JK × 13%` (not rounded) |
| ESIC @3.25% | `ROUND((JK + HRA) × 3.25%, 0)` |
| TOTAL (B) | `SUM` of the six rows (B, C, E…N) |
| Total | `TOTAL (A) + TOTAL (B)` |

HOLD = full Salary Net Pay of employees on hold (Remarks Resigned / Absconding / Hold / ATL / Terminate / Accident, adjustable in **Hold Review**).

Sheet `HOLD&TDS` — four bordered blocks: **Hold** (Net Pay Held, Remarks) · **TDS** · **Deductions** (one row per component:
HO Loan → Salary Advance → Other Deduction, then Sales in the same order; Principal/Interest on loan rows from the **Loan Split** tab;
the "Other Deduction" remark in column H) · **NPS**. Each block total is checked against the MIS column (Checks sheet).

## Notes
- The page is marked `noindex`; `_headers` blocks framing and limits scripts to this site (plus the two CDNs as a fallback only).
- Password-protected salary files: type the password in the page; it is used only in the browser and never stored.
