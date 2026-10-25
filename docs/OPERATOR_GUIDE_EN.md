# Weekly Outstanding Report — Operator Guide (English)

Version 1.1 · 2026-10-25 · Owner: Global Ops (Aiden) · Operator: VNOP (Vicky)

This guide lets an operator produce the weekly outstanding-receivables report on their own PC with Claude Code,
publish the tracker site **from their own GitHub account**, and post the summary to the Teams chat **"Outstanding"**
by copy-and-paste. Posting through Microsoft Graph is not part of this guide; the operator pastes the generated text by hand.

---

## 1. What the weekly run produces

| Output | Where | Use |
|---|---|---|
| Teams message (plain text, Korean) | `automation/out/teams-chat-message.txt` | Copy the whole file and paste it into the "Outstanding" chat |
| Review workbook | `automation/out/Outstanding_Review_<date>.xlsx` | Summary, every open invoice with SOP level, the "ELLIS pending" list, payments |
| Tracker site data | `public/data/bundle.enc.json` (encrypted) | Pushed to your GitHub repository; your site `https://<your-github-username>.github.io/Outstanding-Tracker/` updates in about 2 minutes |
| Key-conclusions memo (Korean) | `handoff/leadership/핵심결론_대표님_재무팀_<date>.md` | Written by Claude during the run; for the CEO / Finance |
| Teams card preview (JSON) | `automation/out/teams-graph-card.json` | Reference only |

The figures follow fixed rules (see section 7). The operator does not edit numbers by hand.

---

## 2. One-time setup

### 2.1 Your own GitHub repository and site (about 15 minutes)
The code and the site live in **your** GitHub account. Nothing from Aiden's account is needed for deployment.

1. Create a GitHub account at https://github.com if you do not have one (any plan; the repository must be **public** for GitHub Pages on the free plan).
2. Fork the source repository: open https://github.com/bstars00-rgb/Outstanding-Tracker, click **Fork** (top right), keep the name `Outstanding-Tracker`, click **Create fork**. Your copy is `https://github.com/<your-github-username>/Outstanding-Tracker`.
3. Enable workflows on the fork: in your repository open the **Actions** tab and click **"I understand my workflows, go ahead and enable them"** (GitHub disables them on forks by default).
4. Enable the site: **Settings → Pages → Build and deployment → Source: "GitHub Actions"**. Save.
5. Choose the site password and register its hash (the password itself is never stored on GitHub):
   - on your PC, after step 2.2 below: `npm run gate:hash -- "<your site password>"`
   - copy the printed `VITE_GATE_HASH=...` value (everything after `=`)
   - **Settings → Secrets and variables → Actions → Variables tab → New repository variable**: name `VITE_GATE_HASH`, value = the copied hash
   Use a long passphrase (12+ characters). The same password goes into `.env` as `DATA_PUBLISH_PASSWORD` (step 2.2) — the two must match, otherwise viewers cannot decrypt the published data.
6. First deployment: **Actions → "Deploy Tracker to GitHub Pages" → Run workflow** (or just push a commit). After about 2 minutes the site is live at `https://<your-github-username>.github.io/Outstanding-Tracker/`. Open it, enter your password, and you should see a "No data published yet" message until your first weekly run.
7. Tell Aiden the site URL. From then on the Teams text links to your site and carries your password, so the chat members use your site.

Keeping your fork up to date with Aiden's fixes (only when Aiden announces a change):
```bash
git remote add upstream https://github.com/bstars00-rgb/Outstanding-Tracker.git   # once
git pull upstream main --no-edit
git push origin main
```
If git reports a conflict in `public/data/bundle.enc.json` (both sides published data), keep yours:
`git checkout --ours public/data/bundle.enc.json`, then `git add public/data/bundle.enc.json` and `git commit --no-edit`, then push.
Or simply ask Claude Code: "update this fork from upstream and resolve the bundle conflict by keeping ours".

### 2.2 Your PC (once, about 20 minutes)
1. Install: **Git** (https://git-scm.com), **Node.js 22 LTS** (https://nodejs.org), **Python 3.11+** (https://python.org, tick "Add to PATH"), **Claude Code** (desktop app).
2. Clone **your** fork and install packages:
   ```bash
   git clone https://github.com/<your-github-username>/Outstanding-Tracker.git
   cd Outstanding-Tracker
   npm ci
   pip install openpyxl
   ```
   The first `git push` asks you to sign in to GitHub once (Git Credential Manager remembers it).
3. Create the local settings file `.env` in the project root (it is git-ignored; never commit it):
   ```
   DATA_SOURCE=file
   REPORT_LANGUAGE=ko
   REPORTING_CURRENCY=JPY
   AI_PROVIDER=mock
   TRACKER_BASE_URL=https://<your-github-username>.github.io/Outstanding-Tracker/
   TEAMS_SENDER=mock
   DRY_RUN=true
   DATA_PUBLISH_PASSWORD=<the same password whose hash you registered as VITE_GATE_HASH>
   TEAMS_CARD_PASSWORD=true
   # WORKBOOK_DIR=C:/Users/<you>/Downloads   (default: your Downloads folder)
   ```
   `TEAMS_CARD_PASSWORD=true` puts the site password into the Teams text so chat members can open the tracker
   (decision of 2026-10-02). Set it to `false` if the password must not appear in the chat.
4. Check: `npm run typecheck` prints nothing, `node --version` shows v22 or newer, and `python -c "import openpyxl"` prints nothing.
5. Open the project folder in Claude Code once so the project skill `weekly-outstanding` is available.

### 2.3 From Aiden (optional)
- A copy of `automation/input/ellis-channels.json` (ELLIS seller directory → managing entity and country). Without it the converter uses the built-in mapping (Ctrip Vietnam → OMH Vietnam; Ctrip Korea, Agoda, Kakao → OMH Korea; others → OMH Singapore).
- The current `automation/config/pic-overrides.json` is already in the repository (account owners confirmed by Aiden).

---

## 3. Every week (Friday, after the OP workbook is final)

1. Save the OP workbook as `Outstanding_Report_<YYYY-MM-DD>.xlsx` in your Downloads folder
   (it must contain the dated tab for this week, e.g. `23-Oct`, plus the `Tier` and `information` sheets).
2. In Claude Code (project folder open) type:
   ```
   weekly outstanding report
   ```
   Claude checks the workbook against last week, runs the pipeline, publishes the site, writes the memo,
   and tells you where the Teams text is. It stops and asks you when something looks wrong (section 6).
   Manual alternative (same result, no memo):
   ```bash
   npm run weekly -- --latest --push
   ```
   If the tab date is not today (for example you run on Monday for Friday's tab), give the date:
   ```bash
   npm run weekly -- --latest 2026-10-23 --push
   ```
3. Open `automation/out/teams-chat-message.txt`, select all, copy, and paste it into the Teams chat **"Outstanding"**.
   Paste as plain text (Ctrl+Shift+V keeps the line breaks in the Teams compose box). Send it once.
4. Send the review workbook and the memo to Aiden (Teams DM or e-mail attachment).
5. Do not run the same report date twice unless a correction is needed; a re-run overwrites the outputs and
   re-publishes the site with a new encryption salt (viewers re-enter the password once).

Time: about 5 minutes when nothing is unusual.

---

## 4. Reading the run output

The command ends with a block like this:
```
── 5/5 done  report 2026-10-23
  Teams Graph: not configured (automation/teams-graph/weekly-post-targets.json missing)   ← expected on your PC
  Teams text : automation/out/teams-chat-message.txt (copy & paste into the Teams chat; includes the site password)
  Review     : automation/out/Outstanding_Review_2026-10-23.xlsx
  Site bundle: public/data/bundle.enc.json (pushed)
```
Before that you should see `RESULT dry-run report_date=<date>` ("dry-run" refers to the old webhook sender; it is normal)
and `verification ok=true`. "previous snapshot: <last week's date>" confirms the week-over-week comparison baseline.

Quick sanity checks before pasting:
- The title date equals the workbook tab date.
- "ELLIS 원장 표시" (ledger) = "실질 고객 미수" (real receivables) + "입금확인·ELLIS 미반영" (received, not in ELLIS). The review workbook's Summary sheet shows the same reconciliation.
- No seller appears with an empty owner except brand-new channels (tell Aiden).

---

## 5. What the converter does with the workbook (so you can predict the numbers)

| Workbook item | Tracker treatment |
|---|---|
| Dated tab matching the report date (`23-Oct`) | Current invoice list. The previous dated tab is used to date payments. |
| `Noted` = "Payment received …", "AGT paid …", "<seller> paid …" | Money is at the bank but ELLIS is not updated → counted as **collected** in the tracker and listed under **입금확인·ELLIS 미반영** (owner: Lina (Josh) / Accounting). The ELLIS balance is kept in the notes. |
| `Noted` naming a **different** seller (copy-paste mistake) | Not counted as received; listed in the data notes for VNOP to confirm. |
| `Noted` = "Due to holiday …", "Chasing", "Post-pone …" | Kept as a note only; the invoice stays open. |
| "Business Trip in …", "Unsold room (JP)" | Internal accounts: excluded from receivables, amounts kept in the notes. |
| `Tier` sheet | Tier per channel. PIC comes from `automation/config/pic-overrides.json` first (confirmed by Aiden), then the sheet. |
| `information` sheet | FX rates to JPY (approximate), SOP thresholds. |
| New seller not in the Tier sheet | Appears as "Untiered" with no owner → report it to Aiden. |

SOP levels (from the OMH SOP): L1 = Tier 1 and ≥ ¥1M overdue (CEO within 24h) · L2 · L3 · L4 < ¥100K (monthly).
Approval route: > ¥500K CEO, otherwise Local Director. Tier deadline = due date + 6/4/3 months.

---

## 6. When to stop and ask Aiden instead of pasting

- The run prints `FAILED`, `validation failed`, or `RESULT failed`.
- The workbook layout changed (new/renamed columns, no dated tab, a new kind of note you have not seen).
- Numbers look implausible: real receivables jump more than 10×, or the ledger does not reconcile in the Summary sheet.
- A new seller has no owner/tier, or an internal-looking account appears (출장 / 하드블럭).
- The newest workbook is older than 6 days (the run refuses it: "looks like last week's file").
- `git push` is rejected (conflict with upstream or sign-in problem): the Teams text is still valid; fix the push (section 2.1 / 8) or ask Aiden.

Never paste a message you have not read. Never share the Teams text outside the "Outstanding" chat (it contains the password).

---

## 7. Fixed conventions (do not change without Aiden)

- Reporting currency JPY; Korean wording in the Teams text and memo (`REPORT_LANGUAGE=en` exists but is not used for leadership).
- Managing entity from the ELLIS seller directory when available, else the built-in mapping.
- One Teams post per report date; corrections are posted only when Aiden asks.
- "대표님 결정 필요" (CEO decision) appears only for real CEO decisions (e.g. SOP L1). The ELLIS reflection backlog is an Accounting task and goes to "이번 주 조치".
- Internal accounts excluded; PIC overrides win over the workbook Tier sheet.
- The published site data is encrypted with the site password; plaintext JSON never leaves the PC.
- The site is deployed from the operator's own GitHub repository (a fork); Aiden's repository stays the source of code updates.

---

## 8. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `python` is not recognized | Reinstall Python with "Add to PATH", or set `PYTHON=C:/path/to/python.exe` in `.env`. |
| `ModuleNotFoundError: openpyxl` | `pip install openpyxl` |
| `Workbook not found` | Check the file name pattern `Outstanding_Report*.xlsx` and the folder (`WORKBOOK_DIR`). |
| `DATA_FILE must be a ReceivablesDataset …` | The converter produced an invalid dataset — usually a new workbook quirk. Ask Aiden; include the console output. |
| `Site bundle: not written` | `DATA_PUBLISH_PASSWORD` is empty in `.env`. |
| `git push` fails | Sign-in problem (first push opens a GitHub login) or your fork is behind: `git pull origin main --no-edit`, then re-run with `--push`. |
| Site shows last week after 5 minutes | Open the **Actions** tab of your repository: the "Deploy Tracker to GitHub Pages" run must be green. A red run: open it and read the failed step; typecheck/test failures mean the code changed — tell Aiden. |
| Actions tab says workflows are disabled | Click "I understand my workflows, go ahead and enable them" (fork default). |
| Site is 404 | Settings → Pages → Source must be "GitHub Actions"; wait for the first green deploy. |
| Site accepts the password but says the data "could not be decrypted" | `DATA_PUBLISH_PASSWORD` in `.env` differs from the password behind `VITE_GATE_HASH`. Make them identical and run the week again. |
| Site asks for the password after every publish | Normal: each publish uses a new encryption salt; enter the password once. |
| Teams paste loses line breaks | Paste as plain text (Ctrl+Shift+V) or paste into Notepad first and copy again. |

---

## 9. Glossary

- **ELLIS**: the settlement system. The "ledger" number is what ELLIS shows as outstanding.
- **Received, not in ELLIS (입금확인·ELLIS 미반영)**: VNOP confirmed the payment arrived; Accounting has not recorded it in ELLIS yet.
- **Real receivables (실질 고객 미수)**: ledger minus the received-not-in-ELLIS amounts — what customers still owe.
- **SOP L1–L4**: urgency levels from the OMH Outstanding Handling SOP.
- **Tracker**: the password-protected site with KPIs, aging, customer risk, invoices and the action board.

Questions: Aiden (Global Ops). Source repository: https://github.com/bstars00-rgb/Outstanding-Tracker (your fork is the one you push to) — technical details in `docs/RUNBOOK_MANUAL_RUN.md` (Korean) and `.claude/skills/weekly-outstanding/SKILL.md`.
