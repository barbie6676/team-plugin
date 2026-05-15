# team-plugin

Shared Claude Code skills for the team.

## Install

In Claude Code, run:

```
/plugin marketplace add github@barbie6676/team-plugin
/plugin install team-plugin@barbie6676/team-plugin
```

## Skills

### screenshot-cleaner

Cleans up macOS screenshots from a folder. Reviews each screenshot one-by-one with an image description, asks whether to delete or keep it, offers to rename kept files, and organizes them into monthly subfolders.

**Usage:**
- `clean my screenshots` — scans `~/Desktop` by default
- `clean screenshots in ~/Downloads` — scans a custom folder

**What it does:**
1. Finds all files matching the macOS screenshot naming convention (`Screenshot YYYY-MM-DD at ...png`)
2. Shows each one with a description and asks to delete or keep
3. If kept, suggests a short descriptive filename and asks to rename
4. Moves kept files into `Screenshots/YYYY-MM/` inside the target folder
