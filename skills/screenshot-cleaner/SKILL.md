---
name: screenshot-cleaner
description: Clean up screenshots from a folder. Defaults to ~/Desktop if no path is given. Use this skill whenever the user mentions cleaning, organizing, managing, or clearing out screenshots — even if they just say "my desktop is cluttered" or "can you go through my screenshots". Also triggers when a path is provided, e.g. "clean screenshots in ~/Downloads" or "organize screenshots at /some/path". Triggers on phrases like "clean my screenshots", "clear desktop screenshots", "organize screenshots", "go through my screenshots", "delete old screenshots", "my desktop is full of screenshots".
---

# Screenshot Cleaner

Finds all screenshots in a target folder, reviews them one-by-one: describe what the image shows, ask whether to delete it, and if not, suggest a short descriptive name and offer to rename it. Kept files are moved into monthly subfolders.

## Step 1: Determine target folder

Check if the user provided a path argument (e.g. "clean my screenshots in ~/Downloads" or "clean screenshots at /some/path").

- **If a path was provided** → use that as the target folder
- **If no path was provided** → default to `~/Desktop`

Confirm the target to the user: `"Scanning ~/Desktop for screenshots..."` (or whichever path applies).

## Step 2: Find screenshots

Scan the target folder (one level deep) for files matching the macOS screenshot naming convention:

```
Screenshot YYYY-MM-DD at H.MM.SS AM/PM.png
```

Use this bash command (replace `TARGET` with the resolved path):
```bash
find TARGET -maxdepth 1 -name "Screenshot ????-??-?? at *.png" | sort
```

**Naming convention examples (all valid):**
- `Screenshot 2026-05-14 at 3.17.44 PM.png`
- `Screenshot 2025-12-01 at 10.05.09 AM.png`
- `Screenshot 2026-01-08 at 9.43.22 PM.png`

If no screenshots are found, tell the user and stop.

## Step 3: Review one-by-one (in target folder, before moving anything)

Tell the user how many were found and that you'll go through them together:
> "Found 9 screenshots. Let's go through them one by one — oldest first."

Work through the screenshots in chronological order (oldest first). For each one:

### 2a. Describe the image

Read the image file using your vision capability and describe it in 1–2 sentences. Be specific — name the app, mention the content, note anything distinctive. A good description helps the user decide quickly.

Good: "A Slack conversation in #eng-infra about a failed deployment on March 14."
Too vague: "A screenshot of a chat app."

### 2b. Ask to delete

Present like this (adjust the count):

```
─────────────────────────────────
Screenshot 3 of 9 · Screenshot 2026-04-30 at 9.43.22 PM.png
─────────────────────────────────
A Slack conversation in #eng-infra discussing a failed deployment on March 14.

Delete this? [y/n]
```

- **y** → delete the file immediately using AppleScript (moves to Trash), print `Deleted. ✓`, move to the next:
  ```bash
  osascript -e 'tell application "Finder" to delete POSIX file "FULL/PATH/TO/FILE.png"'
  ```
- **n** → proceed to 2c

### 2c. Offer to rename, then move to monthly folder (only if user chose to keep)

Suggest a short, descriptive filename based on what you saw in the image:
- Lowercase, words separated by hyphens, no spaces
- Keep the `.png` extension
- Aim for 2–5 words, enough to be identifiable later

```
Rename to: slack-deployment-discussion.png? [y/n]
```

- **y** → use the suggested name
- **n** → keep the original filename

Then move the file (with its final name) into the appropriate monthly folder under `TARGET/Screenshots/` (where TARGET is the folder being cleaned):

```
TARGET/Screenshots/
├── 2026-05/
│   └── slack-deployment-discussion.png
└── 2026-04/
    └── Screenshot 2026-04-30 at 9.43.22 PM.png
```

Extract the year and month from the original filename (e.g., `Screenshot 2026-05-14 at ...` → `2026-05`) and create the folder if needed:

```bash
mkdir -p TARGET/Screenshots/2026-05
osascript -e 'tell application "Finder" to move POSIX file "TARGET/Screenshot 2026-05-14 at 3.17.44 PM.png" to POSIX file "TARGET/Screenshots/2026-05/"'
```

Print `Moved to Screenshots/2026-05/. ✓`

## Step 3: Summary

After all screenshots have been reviewed, print a brief summary:

```
─────────────────────────────────
Done! Here's what happened:
  Deleted:  5
  Kept:     4  →  TARGET/Screenshots/2026-05/ (3) and TARGET/Screenshots/2026-04/ (1)
─────────────────────────────────
```

If everything was deleted, say so and note that no folders were created.

## Notes

- Never delete a file without explicit "y" from the user.
- Use `osascript` (AppleScript via Finder) for all file deletions and moves — `rm` and `mv` fail silently on macOS Desktop due to sandboxing. Deletion moves files to Trash (recoverable). `mkdir -p` still works for creating the destination folders.
- If the user types something other than y/n, ask again — don't guess.
- If a suggested rename already exists in the destination folder, append `-2`, `-3`, etc.
- If the user wants to stop mid-way (e.g., "stop", "quit", "that's enough"), print a partial summary and stop gracefully.
- Only create `~/Desktop/Screenshots/` subfolders when a file is actually being kept — never pre-create them.
