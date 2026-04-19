# md_storage_html — Project Context

## What this project is

A desktop GUI application for managing and exporting Markdown files to HTML.
It lets the user browse a folder of `.md` files in a tree, edit them, save them back to disk, and export the whole tree as styled HTML with a navigable `index.html`.

**Author:** arun.ramasamy@gmail.com  
**Version:** 1.2

---

## Technology stack

| Layer       | Technology                                      |
|-------------|-------------------------------------------------|
| Language    | Java (compatible with JDK 8+)                   |
| Runtime     | JRE only needed to run; JDK needed to recompile |
| Tested with | OpenJDK 21.0.10 (Ubuntu 24.04)                  |
| GUI         | Java Swing (`javax.swing`, `java.awt`)           |
| Build tool  | None — compiled directly with `javac`            |
| Config      | `md_storage_html.properties` (auto-created)      |

No Maven, Gradle, or third-party dependencies. Pure Java SE standard library.

---

## Source files

| File                  | Role                                                              |
|-----------------------|-------------------------------------------------------------------|
| `md_storage_html.java`| Entry point — builds the `JFrame`, wires panels, auto-loads state |
| `AppState.java`       | Shared mutable state: folder paths, file contents map, tree model |
| `AppProperties.java`  | Reads/writes `md_storage_html.properties` (MD folder, HTML folder, font size) |
| `FileTreePanel.java`  | Left pane: `JTree` + `JTextArea` editor, tree CRUD, drag-and-drop |
| `ToolbarPanel.java`   | Top toolbar: Save, md folder, html folder, Export, Open In Browser, font size spinner |
| `MarkdownExporter.java`| All file I/O: load from disk, save to disk, export MD→HTML, generate `index.html` |

---

## How to compile and run

```bash
# Compile (JDK required)
javac *.java

# Run (JRE sufficient)
java md_storage_html
```

Both commands must be run from the project directory (`/code/java/proj_03_md_storage`).  
The `.class` files are already present, so you can run without recompiling unless source changes are made.

---

## Configuration file

`md_storage_html.properties` is created automatically next to the `.class` files on first use.

| Property        | Meaning                              | Default |
|-----------------|--------------------------------------|---------|
| `md.folder`     | Source folder of `.md` files         | (none)  |
| `html.folder`   | Destination folder for HTML export   | (none)  |
| `html.font.size`| Font size (px) used in exported HTML | 20      |

---

## Application layout

```
JFrame (800×600)
├── ToolbarPanel  [NORTH]   — buttons + folder info bar
└── JSplitPane    [CENTER]  — divider at 220px
    ├── FileTreePanel [LEFT]  — JTree + CRUD buttons
    └── JScrollPane   [RIGHT] — JTextArea (markdown editor)
```

---

## Key behaviours

- **Auto-load:** On startup, if `md.folder` is saved, the tree is populated from disk.
- **Auto-save to memory:** Switching tree selection flushes the current file's text to `AppState.fileContents` (in-memory map).
- **Dirty indicator:** Title shows `*` suffix when there are unsaved changes.
- **Save (Ctrl+S or button):** Purges `.md` files in the source folder, then writes the full tree back to disk.
- **Export:** Converts every `.md` file to HTML using the built-in Markdown renderer, mirrors the directory structure, and generates `index.html` with a collapsible sidebar.
- **Open In Browser:** Opens `index.html` from the HTML export folder in the default browser via `java.awt.Desktop`.

---

## Keyboard shortcuts

| Shortcut   | Action                        |
|------------|-------------------------------|
| `Ctrl+S`   | Save all files to MD folder   |
| `Ctrl+N`   | New file dialog               |
| `Delete`   | Delete selected tree node     |

---

## Markdown rendering (MarkdownExporter)

Custom hand-written renderer — no third-party library. Supports:
- Headings `#`–`######`
- Bold, italic, bold-italic, strikethrough
- Inline code and fenced code blocks (` ``` `)
- Ordered and unordered lists
- Tables (GitHub-style `|` syntax)
- Blockquotes, horizontal rules
- Links and images
- Responsive CSS with configurable font size

---

## Known constraints / gotchas

- File names are used as keys in `AppState.fileContents`. Renaming a file updates the key correctly, but duplicate file names across folders are not supported (flat key space).
- Drag-and-drop in the tree only moves **file** nodes, not folders.
- The Markdown renderer is line-by-line; complex nested constructs (e.g., lists inside blockquotes) may not render correctly.
- `AppProperties` silently ignores I/O errors — if the properties file is unwritable, settings won't persist.
