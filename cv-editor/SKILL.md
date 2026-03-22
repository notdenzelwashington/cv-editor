---
name: cv-editor
description: >
  Use this skill whenever the user wants to edit, improve, reformat, or restructure a CV or resume.
  Triggers include: uploading a .docx CV or resume file, asking to "polish my CV", "clean up my resume",
  "improve my CV layout", "restructure my resume", "make my CV look professional", or any request
  involving editing or redesigning a curriculum vitae or resume document. Always use this skill when
  a .docx CV/resume is present and the user wants any kind of editing, improvement, or reformatting —
  even if they just say "fix this" or "tidy this up". Output is always a polished .docx file.
---

# CV / Resume Editor Skill

## Overview

This skill reads an existing `.docx` CV, extracts all content, edits/restructures it, and outputs a polished `.docx` file. The workflow is always:

1. **Extract** content from the `.docx` (via pandoc)
2. **Analyse** structure and content quality
3. **Improve** language, layout and structure
4. **Generate** a polished `.docx` via python-docx

---

## Step 1 — Extract Content from .docx
```bash
pandoc /mnt/user-data/uploads/<filename>.docx -t plain -o /home/claude/cv_raw.txt
```

Also extract markdown to understand heading structure:
```bash
pandoc /mnt/user-data/uploads/<filename>.docx -t markdown -o /home/claude/cv_struct.md
```

Read both files and form a complete picture of:
- Candidate name and contact info
- CV sections (e.g. Summary, Experience, Education, Skills, etc.)
- Any formatting/structure problems

---

## Step 2 — Analyse and Plan Improvements

Before writing code, think through:

**Content improvements:**
- Tighten bullet points — start each with a strong action verb (Led, Delivered, Designed, etc.)
- Remove filler phrases ("responsible for", "helped with", "assisted in")
- Quantify achievements where possible (suggest placeholders like `[X%]` if numbers are missing)
- Ensure consistent tense: past tense for old roles, present for current role
- Fix grammar, spelling, punctuation

**Structure improvements:**
- Use a standard professional section order: Contact Info → Summary/Objective → Experience → Education → Skills → (Certifications / Projects / Languages if present)
- Ensure each job entry has: Company, Title, Date range, Location, Bullet points
- Remove orphaned content, duplicate headings, or empty sections

---

## Step 3 — Generate .docx with python-docx

Install if needed:
```bash
pip install python-docx --break-system-packages
```

Use python-docx for clean, structured layout. Follow these design rules:

### Layout Rules
- Page: A4 with 18mm left/right margins, 18mm top, 16mm bottom
- Font: Calibri family
- Name: 22pt bold, centred
- Contact line: 9pt, centred, use `|` as separator
- Section headings: 11pt bold, ALL CAPS, with a thin bottom border (accent colour)
- Body text: 10pt
- Bullet points: Use `•` with paragraph indent
- Dates/locations: 9pt italic, right-aligned
- Colour accent for section headings: `#2E5496` (professional blue)

### Code Template
```python
from docx import Document
from docx.shared import Pt, Mm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

doc = Document()

# --- Page margins ---
for section in doc.sections:
    section.page_width  = int(210 * 36000)
    section.page_height = int(297 * 36000)
    section.left_margin   = Mm(18)
    section.right_margin  = Mm(18)
    section.top_margin    = Mm(18)
    section.bottom_margin = Mm(16)

ACCENT = RGBColor(0x2E, 0x54, 0x96)

def fmt(run, bold=False, italic=False, size=10, colour=None):
    run.bold = bold
    run.italic = italic
    run.font.size = Pt(size)
    run.font.name = "Calibri"
    if colour:
        run.font.color.rgb = colour

def add_bottom_border(para, colour_hex="2E5496"):
    pPr = para._p.get_or_add_pPr()
    pBdr = OxmlElement("w:pBdr")
    bottom = OxmlElement("w:bottom")
    bottom.set(qn("w:val"),   "single")
    bottom.set(qn("w:sz"),    "4")
    bottom.set(qn("w:space"), "2")
    bottom.set(qn("w:color"), colour_hex)
    pBdr.append(bottom)
    pPr.append(pBdr)

def tight(para, before=0, after=0):
    para.paragraph_format.space_before = Pt(before)
    para.paragraph_format.space_after  = Pt(after)

def section_heading(title):
    p = doc.add_paragraph()
    tight(p, before=10, after=2)
    run = p.add_run(title.upper())
    fmt(run, bold=True, size=11, colour=ACCENT)
    add_bottom_border(p)

def bullet(text):
    p = doc.add_paragraph()
    tight(p, before=0, after=1)
    p.paragraph_format.left_indent       = Mm(5)
    p.paragraph_format.first_line_indent = Mm(-5)
    run = p.add_run(f"\u2022  {text}")
    fmt(run, size=10)

def body(text):
    p = doc.add_paragraph()
    tight(p, before=0, after=4)
    run = p.add_run(text)
    fmt(run, size=10)

def job_title(title):
    p = doc.add_paragraph()
    tight(p, before=6, after=0)
    run = p.add_run(title)
    fmt(run, bold=True, size=10)

def company_line(name_loc):
    p = doc.add_paragraph()
    tight(p, before=0, after=0)
    run = p.add_run(name_loc)
    fmt(run, size=10, colour=RGBColor(0x55, 0x55, 0x55))

def date_line(dates):
    p = doc.add_paragraph()
    p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    tight(p, before=0, after=2)
    run = p.add_run(dates)
    fmt(run, italic=True, size=9, colour=RGBColor(0x55, 0x55, 0x55))

# Name
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
tight(p, before=0, after=2)
run = p.add_run("CANDIDATE NAME")
fmt(run, bold=True, size=22)

# Contact
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
tight(p, before=0, after=8)
run = p.add_run("email@example.com | +60 12-345 6789 | LinkedIn | City, Country")
fmt(run, size=9, colour=RGBColor(0x55, 0x55, 0x55))

# Summary
section_heading("Professional Summary")
body("Summary text here.")

# Experience
section_heading("Experience")
job_title("Job Title")
company_line("Company Name — City")
date_line("Jan 2022 – Present")
bullet("Achieved X by doing Y, resulting in Z.")

# Education
section_heading("Education")
job_title("Degree Name")
company_line("University Name — City")
date_line("2018 – 2022")

# Skills
section_heading("Skills")
body("Skill A  •  Skill B  •  Skill C  •  Skill D")

doc.save("/mnt/user-data/outputs/cv_improved.docx")
print("Saved cv_improved.docx")
```

---

## Step 4 — Quality Checks

After generating the file, verify:
- [ ] File opens correctly in Word / LibreOffice
- [ ] Name is prominent and readable
- [ ] All original sections and roles are present (nothing dropped)
- [ ] Dates are consistently formatted (e.g. `Jan 2022 – Mar 2024`)
- [ ] No orphaned headings at bottom of page
- [ ] Bullet points start with action verbs

If there are problems, fix the Python and regenerate.

---

## Step 5 — Present and Summarise

Use `present_files` to deliver the `.docx`. Then give the user a brief summary of the improvements made:
- What content was tightened/improved
- What structural changes were made
- Any placeholders added (e.g. `[quantify]`) where numbers were missing

---

## Key Rules

- **Never drop content** — every job, qualification, and skill in the original must appear in the output
- **Never invent facts** — if you strengthen a bullet, keep it factually grounded; use `[X%]` placeholders for missing numbers rather than guessing
- **Keep it to 1–2 pages** — flag to the user if the content is too long and suggest what to trim
- **Respect the user's voice** — improve clarity, don't replace their personality with corporate jargon
- **Use `--break-system-packages`** when installing Python packages: `pip install python-docx --break-system-packages`
