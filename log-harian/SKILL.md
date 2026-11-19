---
name: log-harian
description: Use when the user asks to do their daily activity log (Log Aktivitas Harian / PD-6 / DINTA Bot / /log-harian) for one or more dates: rename work screenshots, draft log entries, and submit them through DINTA Bot in Slack.
---

# Log Aktivitas Harian (DINTA Bot)

Automates the daily work log: screenshots → renamed files → drafted log entries → submitted to DINTA Bot in Slack. DINTA Bot then maps the chosen Task to Inisiatif/EPIC and writes the Google Sheet by itself.

Talk to the user in Indonesian unless they write in another language.

## Konfigurasi (fill in before uploading, or Claude asks on first run)

```
FOLDER_SCREENSHOT : <isi, contoh: C:\Users\NamaKamu\Desktop\Kerja\2026>
LINK_DM_DINTA_BOT : <isi, contoh: https://<workspace>.slack.com/archives/D0XXXXXXXXX>
PLATFORM          : <isi, contoh: Murid>
SQUAD             : <isi, contoh: Pembelajaran Digital>
```

If any value above is still a placeholder (`<isi ...>`), ask the user for it before doing anything else, all missing values in one question. Tell them they can write the values into this section of the skill so they are not asked again.

How to find `LINK_DM_DINTA_BOT`: in Slack, open the DM with DINTA Bot, click the bot's name at the top → copy the link of the conversation, or copy the URL from Slack in the browser.

## Ground rules

- **Never write to the Google Sheet.** DINTA Bot fills it. Reading it is optional.
- **Mode aman (default):** show a draft table and wait for the user's "oke" before submitting anything. Only switch to mode cepat (submit confident entries, ask only on doubtful ones) if the user explicitly says so.
- **Never invent activities.** Every log entry must come from a real screenshot.
- **Never type credentials.** If Slack asks to sign in, the user signs in themselves.
- One activity = one screenshot = one Slack message = one form submission.

## Requirements

- Claude desktop app with this conversation linked to the user's computer (for folder access).
- **Claude in Chrome** extension, signed in to the same Claude account. The built-in browser cannot upload files, so do not use it for submitting.
- Slack signed in inside Chrome (the user may only use the Slack desktop app; if so, ask them to sign in in the Chrome tab Claude opens).

## 1. Read and rename screenshots

- Expected structure: `FOLDER_SCREENSHOT\<Bulan>\<M-D>\`, e.g. `...\November\11-2\`. Date folders use `M-D` without leading zeros. If the user's structure differs, follow theirs and say so.
- If the folder is not connected, call `device_request_folder_access` for the month folder (confirm it exists first with `device_list_dir`). Then list it recursively.
- For each requested date folder, stage the files with `device_stage_files` and view each one with Read.
- Files already named `M-D Melakukan ….png` are done. Unnamed ones (`Screenshot 2026-11-02 135212.png`, `image (2).png`, `Image from iOS.jpg`, …) need a name.
- Name format: `M-D <Deskripsi>.<ext>`, following the user's existing filenames. Descriptions usually start with **"Melakukan …"** (e.g. "Melakukan diskusi …", "Melakukan pembuatan alur dan desain …", "Melakukan eksplorasi desain …"). Base it on what the screenshot actually shows: meeting title, screen being discussed, page or feature.
- Rename on the user's computer with `device_bash`: `cd "$HOME/mnt/<folder>/<M-D>" && mv -n "<old>" "<new>"`. Never delete files.
- **Date check:** compare each file's modified time (from `device_list_dir`, converted to the user's timezone) and any clock visible in the screenshot with the folder's date. If they disagree (e.g. a screenshot from Oct 30 inside folder `11-2`), flag it in the draft and ask.

## 2. Draft table, then wait

Show: # · Deskripsi Log Aktivitas · Task · Yakin? (✅ / ⚠️ with a short reason). Description = filename without the `M-D ` prefix and extension.

## 3. Task mapping

- **The Task list changes every month.** Never rely on last month's names. Read the live dropdown in the DINTA Bot form (step 4) before finalizing the Task for each entry.
- The dropdown has a "Cuti" group, several user-story groups (often engineering or data tasks), and a **"Task Lainnya"** group where many design and coordination tasks live.
- Match each activity on the screenshot content and on the user's past choices. When the user tells you which Task they pick for a kind of activity, remember it for the rest of the session and suggest they add it to the "Pilihan Task" section below.
- If an activity fits two Tasks, or none, mark ⚠️ and ask.

### Pilihan Task (optional, filled by the user)

```
<contoh: diskusi mingguan dengan desainer → <nama Task>>
```

## 4. Submit via DINTA Bot (Claude in Chrome)

Load the Chrome tools in one ToolSearch call (include `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `computer`, `find`, `read_page`, `get_page_text`, `file_upload`, `javascript_tool`, `tabs_close_mcp`). Call `tabs_context_mcp` first and work in a new tab.

- Open `LINK_DM_DINTA_BOT`. Slack may redirect to `app.slack.com/client/...`. If it shows the sign-in page, ask the user to sign in in that tab and wait.

For each approved activity, one at a time:

1. **Upload:** `find` the file input of the **main** message composer ("Message to DINTA Bot"), not the thread reply composer. Call `file_upload` with origin `https://app.slack.com` and the **staged container path** of the file (`/mnt/user-data/uploads/...`). Paths on the user's computer are rejected. Re-stage a file after renaming it so the staged copy has the new name.
2. **Message:** click the "Message to DINTA Bot" textbox, type the description, take a screenshot to check the text and attachment, press Return.
3. **Open the form:** hover the new message → click ⋮ (More actions) → hover **Connect to apps** → click **Jadikan laporan (DINTA Bot)**. The form never appears on its own. If the menu closes without opening the modal, repeat these steps.
4. **Fill the modal:**
   - Date picker defaults to today: click it and pick the correct day in the calendar (use the arrows for another month).
   - Platform and Squad are usually prefilled; check they match `PLATFORM` and `SQUAD`.
   - Task: click the combobox, type a distinctive part of the Task name to filter (the list is virtualized, so options far down may not be found otherwise), then click the option. The modal then shows "Task dipilih:" with the full name.
   - The description is prefilled from the message; check it.
   - Verify with `javascript_tool`: `[...document.querySelectorAll('[role=dialog] input, [role=dialog] textarea')].map(i=>i.value)` must show the date as `MM/DD/YYYY` and the description.
5. **Submit**, then wait about 8–15 seconds. Success = DINTA Bot replies in that message's thread: "Log Aktivitas Harian anda untuk tanggal YYYY-MM-DD sukses terunggah…". A "1 reply" under the message means the bot answered; open the thread to read it when you need to confirm.
6. **Check the composer:** Slack sometimes puts the sent text back into the composer as a draft. If it is not empty, clear it (ctrl+a, Delete) so nothing is sent twice.

Stop and ask the user if a step fails 2–3 times, the extension is not connected, a dialog blocks the page, or Slack behaves unexpectedly. Close the tabs you opened when done.

## 5. Report back

Short table: # · Aktivitas · Task · Status (✅ confirmed by bot / ⚠️ issue). Say which bot replies you actually read and which you only saw as a reply count. Mention anything unusual: date mismatches, a leftover draft you cleared, a changed Task list.
