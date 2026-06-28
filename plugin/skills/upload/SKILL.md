---
name: upload
description: >
  Trigger this skill whenever the user wants to upload, commit, or save attached files directly
  to the Tymraft repo. Trigger on any of the following:

  - "/upload"
  - "upload this", "upload these files", "upload the attached"
  - "commit this file", "commit these files", "save this to the repo"
  - "put this in [folder]", "add this to docs/", "push this file to [location]"
  - Any prompt where one or more files are attached AND the user indicates a destination folder
    (even loosely: "in the briefs folder", "under assets", "in example-folder")
  - Any prompt where the user says "just commit it", "drop it in [folder]", "store this"

  When in doubt and files are attached, trigger. It is better to load this skill unnecessarily
  than to miss it.
---

# Upload Files to Tymraft Repo

This skill commits one or more attached files directly to the `docs/` folder of the
`Tymraft/tymraft.github.io` repository, exactly as provided — no transformation, no wrapping,
no template. The raw file content is committed as-is.

Use plain, friendly language. Avoid jargon.

---

## Step 1 — Check for attached files

Look for files in the current prompt context (uploaded by the user).

- If **no files are attached**, tell the user in plain language:
  > "I don't see any files attached. Please attach the file(s) you'd like to upload and let me
  > know which folder to put them in."
  Then stop — do not proceed.

- If **files are attached**, continue to Step 2.

---

## Step 2 — Determine the destination folder

All uploads go under `docs/`. The user specifies a subfolder (or none for the root of `docs/`).

Parse the destination from the user's prompt:

| User says | Resolved path |
|---|---|
| "in example-folder" | `docs/example-folder/` |
| "under briefs" | `docs/briefs/` |
| "in docs/assets" | `docs/assets/` |
| "just in docs" / no subfolder given | `docs/` |

Rules:
- Always prefix the path with `docs/` — never commit outside `docs/`
- Strip any leading `docs/` the user may have typed to avoid doubling it
- Preserve the original filename exactly (including extension, capitalisation, spaces)
- If the path already has a trailing `/`, don't add another

If no destination is mentioned and it can't be inferred, ask once:
> "Where would you like me to put it? For example: 'in the briefs folder' or 'just in docs'."
Then wait for their answer before proceeding.

---

## Step 3 — Commit the files

For each attached file:

1. Read the raw file content exactly as uploaded
2. Construct the full repo path: `docs/[subfolder/]filename.ext`
3. Check whether a file already exists at that path (use `github:get_file_contents` to retrieve
   the SHA — if a 404 is returned, the file is new)
4. Commit using `github:create_or_update_file`:
   - `owner`: `Tymraft`
   - `repo`: `tymraft.github.io`
   - `branch`: `main`
   - `path`: the resolved path from above
   - `content`: the raw file content (base64-encoded if binary)
   - `message`: `upload: add [filename] to [folder]`
   - `sha`: include only if updating an existing file

If multiple files are being uploaded, commit them one at a time (the GitHub tool doesn't
support mixed binary/text in a single push call when files need individual SHAs checked).
Alternatively, if all files are new (no existing SHAs needed), batch them with `github:push_files`.

---

## Step 4 — Confirm

After committing, confirm in plain language. For a single file:
> "Done! [filename] is now saved in docs/[folder]/."

For multiple files, list them:
> "Done! I've saved the following to docs/[folder]/:"
> - file-one.pdf
> - file-two.docx

If any file failed (e.g. a commit error), report the failure clearly and offer to retry.

---

## Rules and constraints

- **Never modify file content.** Commit exactly what was attached.
- **Never write outside `docs/`.** If the user requests a path outside `docs/`, redirect:
  > "I can only save files under the docs/ folder. Shall I put it in docs/[their folder] instead?"
- **Preserve filenames exactly** — do not slugify, lowercase, or rename.
- **Do not create index files, READMEs, or any other supporting files** unless explicitly asked.
- If the user attaches files but gives no instruction, ask once where they want them saved,
  then proceed immediately after they answer.
