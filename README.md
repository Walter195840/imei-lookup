# IMEI Lookup

An Excel task pane add-in. It looks up IMEIs in a table of IMEI ranges (`From IMEI` / `To IMEI` headers),
and has an optional **Ask AI** tab for plain-language questions. Plain HTML, CSS and JavaScript: no build step, no npm.

The add-in only reads the workbook (and can select a row). It never edits cells.

## Files

| File | What it is |
|---|---|
| `manifest.xml` | The add-in manifest that Excel loads (name, icons, ribbon button, permissions). |
| `taskpane.html` | The whole add-in: page, styles and scripts. Hosted on GitHub Pages. |
| `index.html` | Help page. Excel's "Get Support" opens it. |
| `icon-16.png`, `icon-32.png`, `icon-80.png` | Ribbon and store icons. |
| `worker/worker.js` | The team AI proxy (Cloudflare Worker). Holds the Groq and Gemini keys. |
| `worker/README.md` | How to deploy the Worker. |
| `install/` | One-click desktop installer for coworkers (`add-imei-lookup.reg`, `remove-imei-lookup.reg`) and `HOW-TO.txt`. |
| `test.js` | Node tests: `node test.js` (IMEI logic, Ask AI logic, fallback, the Worker). |

Only `manifest.xml`, `taskpane.html`, `index.html` and the three icons need to be on GitHub Pages.

## Set-up checklist

1. **Host it.** Put the files above in a public GitHub repo named `imei-lookup` and turn on Pages (Settings, Pages, branch `main`, folder `/`).
   The manifest uses `https://walter195840.github.io/imei-lookup/`.
2. **Deploy the AI proxy** (optional, for key-free Ask AI). Follow `worker/README.md`, then set `PROXY_URL`
   near the top of the `ai-app` script in `taskpane.html`. While `PROXY_URL` contains `YOUR-`, people paste their own keys instead.
3. **Install it in Excel.** Share `manifest.xml` with people:
   - desktop: add a shared folder as a trusted add-in catalog (File, Options, Trust Center), then Home, Add-ins, Advanced, Shared Folder;
   - or have a Microsoft 365 admin deploy it (Admin center, Integrated apps, Upload custom apps) so it appears for everyone;
   - or Excel on the web: Add-ins, More Add-ins, Upload My Add-in (if your organization allows it).
4. **Updating.** Upload a changed `taskpane.html` to GitHub. In Excel use the pane's arrow menu, Reload; if it still looks old,
   close Office and delete everything in `%LOCALAPPDATA%\Microsoft\Office\16.0\Wef\`.
   A changed `manifest.xml` has to be re-added or re-deployed.

## Features

- **Lookup:** one or many IMEIs (14, 15 or 16 digits), every matching range, Go to row, Copy details, Copy results, Use selection,
  a Recent list (the last few IMEIs, stored only in this browser), and automatic table refresh after you edit the sheet.
  Smart paste finds the IMEIs in a pasted email or messy list. From 6 IMEIs on, the summary table comes first and the cards are folded.
  **Table check** (⋯ menu) lists overlapping ranges, From greater than To, and duplicates, with links to the rows.
- **Ask AI:** answers stream in as they are written; Stop, Regenerate, follow-up suggestions, answer length and rows per lookup in
  the ⋯ menu, AI settings.
- **⋯ menu:** Refresh table, Table settings, Use selection, AI settings, Copy conversation, New chat, Text size.

## Things to know

- **Keys.** No API key is in any file. The Worker holds them as secrets. Do not commit keys.
- **What the AI sees.** Never the sheet. It calls four local tools (`lookup_imei`, `lookup_many`, `find_rows`, `list_columns`);
  only their results (capped at about 6,000 characters) are sent. The pane lists everything that was sent under "What was sent to the AI".
- **Streaming.** Groq streams through the existing Worker. For Gemini to stream, redeploy the current `worker/worker.js`
  (paste it into the Worker's editor and Deploy). Until then Gemini simply answers in one piece.
- **Free limits.** Groq is tried first; if it is rate-limited the question goes to Gemini. Both are free tiers with small per-minute limits.
  Settings, Rows per lookup, can be lowered to stay under them.
- **Loading time.** Hover the status line ("N ranges from ...") to see how long the last table read took. A saved local copy for
  instant opening was left out on purpose: it would keep company data on the PC outside the workbook, and it can show stale data.
  If reads are slow (more than about 10 seconds) it is worth revisiting.
- **Team code.** The Worker accepts an optional `TEAM_CODE` secret. Without it anyone who finds the Worker address can use the free quotas.
- **Publisher name.** The manifest still says `Personal Project`. Change `ProviderName` before sharing widely or publishing to AppSource.

## Changing the name

The name appears in `manifest.xml` (`DisplayName`, and the `Group.Label` / `Button.Label` strings) and in `taskpane.html` (`<title>` and the hidden heading).
