![UI](UI.png)

## Requirements

- Python 3.x (tkinter ships with the standard installer — no extra install needed for the UI itself)
- Dependencies for the AI auto-fill feature, listed in `requirements.txt`:

  ```bash
  pip install -r requirements.txt
  ```

- A **Gemini API key** if you want to use AI-generated activities (see below). Not required
  for manual entry — you can fill everything by hand and skip this entirely.

## 1. Run the UI

```bash
python logbook_ui.py
```

## 2. (Optional) Set up your Gemini API key

The AI auto-fill feature calls Google's Gemini API to generate your Activity/Description text.
You have two ways to provide a key:

- **`.env` file (recommended)** — create a `.env` file next to `logbook_ui.py`:

  ```
  GEMINI_API_KEY=your_api_key_here
  ```

  It's loaded automatically on startup.

- **Manual entry** — if no key is found (or it's invalid), the app will pop up a dialog asking
  you to paste your key the first time you open the AI auto-fill menu. The key is validated
  live against the Gemini API before it's accepted; an invalid key re-prompts you.

If you never use the AI feature, you can ignore this step entirely and just fill the table by hand.

## 3. Build your entries

1. **Days** — number of logbook rows. **Start weekday** — weekday of day 1.
   Click **Rebuild rows** to regenerate the table.
2. Per row, set **Clock In** / **Clock Out** (time text + `am`/`pm` selector),
   **Activity**, and **Description**.
3. Per-row flags:
   - **Off** — mark the day OFF. Fields disable; the script clicks the Off
     button for that row.
   - **Skip** — leave the day completely untouched. The row's button is not
     clicked at all (nothing filled, no Off). Use it for days already filled
     correctly that you don't want changed. Disables the rest of the row.
4. Shortcuts:
   - **Set all time** → fill In/Out + am/pm → **Apply times to all**.
   - **Set all text** → fill Activity/Description → **Apply text to all**.
   - Both skip Off and Skip rows. Tweak individual rows afterward.
   - **Reset All** → confirm popup → restores every row to default times,
     blank text, and clears all Off/Skip flags.
5. *(Optional)* **Save JSON…** to keep your values, **Load JSON…** to reload them later.

> Row order must match the row order on the logbook page (top to bottom).

## ✨ AI Auto-Fill Activities (Gemini)

Instead of typing out every Activity/Description by hand, you can have Gemini generate a
whole month of entries for you.

1. Click **Auto-fill Activities (GenAI)** in the footer.
   - If no valid API key is configured yet, you'll be prompted for one first (see step 2 above).
2. A popup opens with:
   - A **Prompt** text box — describe what you actually worked on during the internship
     (even briefly; Gemini will expand and phrase it professionally).
   - **Import Files** — attach supporting context, such as your task list, weekly reports,
     or any notes/screenshots. Selected files are uploaded to the Gemini File API alongside
     your prompt so the model can use them as additional context.
3. Click **Auto-Fill**. The request runs on a background thread (with a progress bar) so the
   UI stays responsive.
4. Gemini returns a structured JSON list of entries covering roughly 4 weeks (28–31 days),
   which automatically:
   - Populates the **Days** count and rebuilds the table to match.
   - Fills in **Clock In / Clock Out / Activity / Description** for every working day.
   - Marks **Saturdays and Sundays as OFF** by default (unless your prompt says otherwise).
   - Accounts for **Indonesian national holidays**, marking those days OFF as well.
5. Review and tweak the generated rows as needed — everything is still editable afterward,
   and Sundays are re-locked automatically (see below).

If Gemini returns a malformed response, you'll get a clear error dialog instead of a crash —
just adjust your prompt/files and try again.

> The AI only fills in text content; it never touches or submits anything on the actual
> logbook site. You still review the result and run the generated script yourself (step 5).

## 4. Generate the script

Click **Generate script.js + copy**. This:

- writes `script.generated.js` next to the UI, and
- copies the full script to your clipboard.

## 5. Run it on the target site

1. Open the logbook page in your browser, on the month/view that shows the rows.
2. Open DevTools console: `F12` (or `Ctrl+Shift+J`), go to the **Console** tab.
3. If the console asks, type `allow pasting` and press Enter.
4. Paste (`Ctrl+V`) the script and press **Enter**.

The script grabs **every** day button in page order (`.detailsbtn`) — both
empty (blue) and already-filled (orange) — and maps `buttons[i]` to
`entries[i]`. For each row it either fills + submits, marks OFF, or (for **Skip**
rows) does nothing and moves on.

## How button matching works

- One combined array in document order, so index `i` always lines up with day
  `i` regardless of button color. This is why the row order in the UI must match
  the page top-to-bottom.
- Filling a day that's already filled (orange) just **overwrites** it — no extra
  flag needed.
- **Skip** is the only way to leave a day alone; its button is never clicked.
- **Sundays** are auto-OFF on the site and have **no button**. In the UI their
  rows are forced **Off** and locked (flags can't be toggled, fields disabled),
  and they are dropped from the generated `entries[]` so the rest stays aligned.
  Set **Start weekday** correctly so the right rows are detected as Sundays. The
  generate popup reports how many were excluded.

## Build a standalone executable

Bundle the UI into a single executable (no Python needed to run it):

1. Install PyInstaller (once):

   ```bash
   pip install pyinstaller
   ```

2. Build:

   ```bash
   pyinstaller --onefile logbook_ui.py
   ```

The binary lands in `dist/` (e.g. `dist/logbook_ui.exe` on Windows). Build
artifacts (`build/`, `dist/`, `*.spec`) are generated and safe to delete.

> Note: if you use the AI auto-fill feature, make sure your `.env` file (or a manually
> entered API key) is available alongside the built executable, since bundling does not
> embed your API key.

## Notes

- Assumes exactly one `.detailsbtn` per day, in day order. If a filled day
  rendered no button, indexes would shift — Skip the affected rows or use
  matching day count.
- Time values are always well-formed (`09:00 am`) because am/pm is a selector,
  not free text.
- AI-generated text should be reviewed before submitting — treat it as a strong first draft,
  not a guaranteed-accurate account of your work.
