# cv-editor — Claude Skill

A Claude skill that takes an existing `.docx` CV or resume and outputs a polished, professionally formatted PDF.

## What it does

- Extracts content from a `.docx` CV using `pandoc`
- Improves language: tightens bullet points, removes filler phrases, fixes grammar
- Restructures sections into a standard professional order
- Generates a clean PDF using Python's ReportLab library
- Flags missing quantifiers with `[X%]` placeholders rather than inventing facts

## How to use

1. Copy the `cv-editor/SKILL.md` file into your Claude skills directory, e.g.:
   ```
   /mnt/skills/user/cv-editor/SKILL.md
   ```
2. Upload a `.docx` CV to Claude and ask it to improve or reformat it — the skill will trigger automatically.

## Trigger phrases

- "Polish my CV"
- "Clean up my resume"
- "Make my CV look professional"
- "Reformat my resume"
- Upload a `.docx` CV and say "fix this" or "tidy this up"

## Output

Always a polished `.pdf` file, delivered via Claude's file presenter.

## Requirements

Claude's environment needs:
- `pandoc` (for `.docx` extraction)
- `reportlab` Python package (installed automatically via `pip`)

## Folder structure

```
cv-editor/
  SKILL.md        ← the skill definition and instructions
README.md
LICENSE
.gitignore
```
