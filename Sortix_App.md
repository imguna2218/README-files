# Sortix — Complete Application Build Guide

No code. Pure architecture. Read this fully before touching a single file.

---

## The Big Picture

Sortix has three phases every time it runs:

**Phase 1 — Scan:** User picks a folder. Rust reads what's inside.

**Phase 2 — Classify:** Each item goes to the model one at a time. Model returns `Category/Subcategory`.

**Phase 3 — Preview & Edit:** User sees the proposed structure, edits it, confirms. Then Rust moves the files.

The user never loses a file. Nothing moves until they explicitly confirm.

---

## The Blocklist — What Rust Must Never Touch

### Absolute system blocklist — hardcode these, no exceptions

These apply whether the user selects Desktop, C drive, or any path:

```
# Windows system folders
Recycle Bin
$Recycle.Bin
$RECYCLE.BIN
System Volume Information
Windows
Program Files
Program Files (x86)
ProgramData
AppData
pagefile.sys
hiberfil.sys
swapfile.sys
desktop.ini
thumbs.db
ntldr
bootmgr
BOOTMGR
boot
Recovery
MsoCache

# Hidden and system files
Any file or folder starting with $ sign
Any file or folder starting with . (dotfiles) — EXCEPT .idea which is a project folder
Any file with the System or Hidden attribute set in Windows metadata

# Common user-critical folders that should never be auto-moved
Documents       ← user may have this as a real folder
Downloads
Desktop
OneDrive
iCloudDrive

# Linux / Mac equivalents if you ever port
proc
sys
dev
etc
usr
bin
sbin
```

### Path-based blocklist — if the selected path IS one of these, refuse to run

```
C:\
C:\Windows
C:\Windows\System32
C:\Program Files
C:\Program Files (x86)
C:\Users\[username]\AppData
/usr
/bin
/etc
/proc
```

If the user tries to point Sortix at `C:\` directly, show a warning: "This looks like a system root directory. Sortix is designed for personal folders like your Desktop or Documents. Please select a more specific folder."

---

## Phase 1 — Folder Selection (The UI Entry Point)

### What the user sees
A clean window opens. One button: **"Choose Folder"**. Clicking it opens the native Windows folder picker — the same one Windows Explorer uses. This is called a "folder dialog" and Rust can trigger it natively.

### What happens after selection
Rust scans exactly one level deep first — it reads all immediate children (files and folders) of the selected path. It does NOT recursively dive into subfolders automatically. Reason: if the user selects their Desktop, you scan the Desktop. You do not scan inside every subfolder on the Desktop. That would be too destructive.

### What gets filtered immediately
Before showing anything to the user, Rust silently removes from the list:
- Everything on the system blocklist above
- Files with Hidden or System attributes
- Files smaller than 1 byte (zero-byte files, shortcuts to system tools)
- The folder the user selected itself

Show the user a count: "Found 82 items to organize. 6 system items ignored."

---

## Phase 2 — Classification (The Model Conversation)

### How Rust talks to Ollama
Rust spawns a subprocess call to `ollama run sortix "filename"` for each item. This is exactly what your Python test script does. One item at a time. Sequential, not parallel — parallel calls to Ollama on a CPU will slow each other down and produce inconsistent results.

### Progress display
Show a progress bar during classification. Something like:

```
Classifying files...  [=================>    ]  67/82
Currently: Cover_letter_capgemini.pdf → Documents/CoverLetters
```

The user should see each classification as it happens in real time. This builds trust — they can see the model is thinking correctly.

### Handling unexpected model output
Sometimes the model may return something that is not in `Category/Subcategory` format. Your Rust code must handle this:
- If output has no `/` → put in `Misc/Uncategorized`
- If output has more than one `/` → take only the first two parts
- If output is empty → put in `Misc/Uncategorized`
- If Ollama times out after 30 seconds → put in `Misc/Uncategorized` and continue

Never crash because of a bad model response. Always fall back to `Misc/Uncategorized` and keep going.

---

## Phase 3 — The Preview and Edit Screen (Most Important Part)

This is where Sortix becomes a real app instead of a script. The user must feel in control before anything moves.

### What the user sees
A two-panel view:

**Left panel — Original location**
A flat list of all 82 items exactly as they are now. Nothing has moved yet.

**Right panel — Proposed structure**
A tree view showing the new folder structure. Expandable folders. Like this:

```
📁 Documents
   📂 Resumes (5 files)
   📂 CoverLetters (6 files)
   📂 Academic (8 files)
📁 Projects
   📂 Source (8 files)
   📂 Apps (8 files)
📁 Shortcuts
   📂 Apps (11 files)
```

Clicking a folder expands it to show the individual files inside.

### What the user can edit in the right panel

**Rename a folder** — double click a folder name, type a new name. If they rename `Documents/Resumes` to `Documents/CV`, Rust updates all files assigned to that category.

**Delete a folder** — right click, delete. All files in that folder go back to `Misc/Uncategorized`. They will stay in place (not moved).

**Move a file** — drag a file from one subfolder to another. The user may disagree with where the model put something.

**Delete a file from the plan** — right click a file, choose "Leave in place". That file will not be moved. It stays exactly where it is.

**Create a new folder** — right click the tree, "New folder". User can create custom categories the model didn't think of.

### The confirmation button
At the bottom: **"Organize Now"** button. It is disabled until the user has seen the preview for at least 3 seconds (prevents accidental clicks).

Next to it: **"Cancel"** — closes everything, nothing is touched.

---

## Phase 4 — The Move Operation

### How files actually move
Rust uses `std::fs::rename` to move files. This is an atomic operation on the same drive — it just updates the directory entry, it does not copy and delete. It is fast and safe.

### Folder creation order
Create all destination folders first, then move files. Never try to move a file into a folder that doesn't exist yet.

### What happens if a filename already exists at the destination
Example: user has `resume.pdf` on Desktop and also `resume.pdf` already in `Documents/Resumes`. Rust must not silently overwrite. Options:
- Rename the incoming file: `resume (2).pdf`
- Ask the user which one to keep
Recommendation: auto-rename with `(2)`, `(3)` suffix. Show a summary at the end of how many files were renamed.

### The undo file
Before moving anything, Rust writes a JSON file called `sortix_undo.json` in the selected folder. This file records every single move operation: original path → new path. Format:

```json
[
  {
    "from": "C:/Users/DELL/Desktop/invoice_2024.pdf",
    "to": "C:/Users/DELL/Desktop/Finance/Invoices/invoice_2024.pdf"
  }
]
```

This is your safety net. If anything goes wrong, an "Undo Last Operation" button reads this file and moves everything back.

### Progress during move
Show a second progress bar: "Moving files... 45/82". This should be fast — under 5 seconds for 82 files.

---

## Phase 5 — Completion Screen

Show a summary:
```
✓ 82 items organized
✓ 12 folders created
✓ 3 files renamed to avoid conflicts
✓ 6 system items left untouched

Undo this operation? [Undo] [Done]
```

Keep the Undo button visible for the entire session. Once the user closes the app, the `sortix_undo.json` file stays on disk so they can run Sortix again and undo from there.

---

## The Technology Stack for Rust

### GUI library — use Tauri
Tauri lets you build the UI in HTML/CSS/JavaScript while the backend logic (file scanning, Ollama calls, file moving) stays in Rust. This is the right choice because:
- The preview/edit tree view is complex UI — much easier in HTML than in a native Rust GUI
- Tauri produces a small executable (~10MB overhead)
- The user gets a real window, not a terminal

### Alternative if you want pure Rust UI — use egui
egui is an immediate-mode GUI library in pure Rust. Simpler than Tauri but the UI will look more basic. Good choice if you want to avoid JavaScript entirely.

### File system — std::fs
Rust's standard library handles all file operations. No external crates needed for the actual moving and scanning.

### Ollama communication — two options
Option A: Subprocess call — `std::process::Command::new("ollama").arg("run").arg("sortix").arg(filename)`. Simple, works immediately.
Option B: HTTP call to Ollama's local API at `http://localhost:11434/api/generate`. More professional, faster, no subprocess overhead.

Recommendation: Start with subprocess (Option A) to get it working. Switch to HTTP (Option B) once the app is stable.

---

## The Folder Structure of the Rust Project

```
Sortix/
├── src/
│   ├── main.rs          ← app entry point
│   ├── scanner.rs       ← reads directory, applies blocklist
│   ├── classifier.rs    ← talks to Ollama, one file at a time
│   ├── blocklist.rs     ← all hardcoded system paths and names
│   ├── mover.rs         ← creates folders, moves files, writes undo file
│   └── undo.rs          ← reads undo file, reverses moves
├── ui/                  ← if using Tauri: HTML/CSS/JS for the interface
├── models/              ← your GGUF file lives here
├── Modelfile_v3
├── Cargo.toml
└── README.md
```

---

## The Exact User Journey — Step by Step

```
1. User double-clicks Sortix.exe
2. Window opens — "Choose a folder to organize"
3. User clicks "Choose Folder" — Windows folder picker opens
4. User selects their Desktop (or any folder)
5. Sortix scans Desktop — finds 82 items, filters out 6 system items
6. Progress bar appears — "Classifying 82 items..."
7. Each file appears one by one with its assigned category
8. Classification finishes — preview screen appears
9. User sees left panel (original) and right panel (proposed structure)
10. User drags "Chat With PDF" from Misc/Uncategorized to Projects/Apps
11. User right-clicks "Recycle Bin" → "Leave in place"
12. User clicks "Organize Now"
13. Rust creates folders, moves files, writes sortix_undo.json
14. Completion screen: "82 items organized. Undo?"
15. User clicks Done — app closes
```

---

## What to Build First (The Order)

1. **Blocklist module** — write this first, test it, never touch it again
2. **Scanner module** — reads a directory, returns a filtered list
3. **Classifier module** — takes one filename, calls Ollama, returns a path
4. **Mover module** — takes the final plan, creates folders, moves files, writes undo file
5. **Basic CLI version** — wire all four modules together with no UI, just terminal output. This is your working prototype.
6. **Add the GUI** — once the CLI version works perfectly, wrap it in Tauri or egui

Do not touch the GUI until the CLI version works end to end. The model is done. The logic modules are straightforward. The GUI is just a skin on top of logic that already works.

That is the complete guide. When you are ready to start any specific module, come back and we write it together.
