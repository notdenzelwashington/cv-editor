---
name: cv-editor
description: >
  Use this skill whenever the user wants to edit, improve, reformat, or restructure a CV or resume.
  Triggers include: uploading a .docx CV or resume file, asking to "polish my CV", "clean up my resume",
  "improve my CV layout", "restructure my resume", "make my CV look professional", or any request
  involving editing or redesigning a curriculum vitae or resume document. Always use this skill when
  a .docx CV/resume is present and the user wants any kind of editing, improvement, or reformatting —
  even if they just say "fix this" or "tidy this up". Output is always a polished PDF.
---

# CV / Resume Editor Skill

## Overview

This skill reads an existing `.docx` CV, extracts all content, edits/restructures it, and outputs a polished PDF. The workflow is always:

1. **Extract** content from the `.docx` (via pandoc)
2. **Analyse** structure and content quality
3. **Improve** language, layout and structure
4. **Generate** a polished PDF via ReportLab

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

## Step 3 — Generate PDF with ReportLab

Use ReportLab Platypus for clean, structured layout. Follow these design rules:

### Layout Rules
- Page: A4 (`from reportlab.lib.pagesizes import A4`)
- Margins: 18mm left/right, 18mm top, 16mm bottom
- Font: Helvetica family (universally available, no embedding needed)
- Name: 22pt bold, centred
- Contact line: 9pt, centred, use `|` as separator
- Section headings: 11pt bold, ALL CAPS, with a thin horizontal rule beneath
- Body text: 10pt, 12pt leading
- Bullet points: Use `•` with `leftIndent=14`, `firstLineIndent=-10`
- Dates/locations: right-aligned using `ParagraphStyle` with `alignment=TA_RIGHT`
- Colour accent for section rules: `#2E5496` (professional blue)

### Code Template

```python
from reportlab.lib.pagesizes import A4
from reportlab.lib.units import mm
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib.enums import TA_LEFT, TA_CENTER, TA_RIGHT
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, HRFlowable
from reportlab.lib.colors import HexColor

ACCENT = HexColor("#2E5496")
PAGE_W, PAGE_H = A4
MARGIN = 18 * mm

doc = SimpleDocTemplate(
    "/mnt/user-data/outputs/cv_improved.pdf",
    pagesize=A4,
    leftMargin=MARGIN, rightMargin=MARGIN,
    topMargin=18*mm, bottomMargin=16*mm
)

# --- Styles ---
name_style = ParagraphStyle("Name", fontName="Helvetica-Bold", fontSize=22,
                             leading=26, alignment=TA_CENTER, spaceAfter=2)
contact_style = ParagraphStyle("Contact", fontName="Helvetica", fontSize=9,
                                leading=12, alignment=TA_CENTER, spaceAfter=10)
section_style = ParagraphStyle("Section", fontName="Helvetica-Bold", fontSize=11,
                                leading=14, spaceBefore=10, spaceAfter=2,
                                textTransform="uppercase")
job_title_style = ParagraphStyle("JobTitle", fontName="Helvetica-Bold", fontSize=10,
                                  leading=13, spaceBefore=6)
company_style = ParagraphStyle("Company", fontName="Helvetica", fontSize=10,
                                leading=13, textColor=HexColor("#555555"))
date_style = ParagraphStyle("Date", fontName="Helvetica-Oblique", fontSize=9,
                             leading=12, alignment=TA_RIGHT, textColor=HexColor("#555555"))
body_style = ParagraphStyle("Body", fontName="Helvetica", fontSize=10, leading=13)
bullet_style = ParagraphStyle("Bullet", fontName="Helvetica", fontSize=10,
                               leading=13, leftIndent=14, firstLineIndent=-10,
                               spaceAfter=1)

def section_heading(title):
    return [
        Paragraph(title, section_style),
        HRFlowable(width="100%", thickness=1, color=ACCENT, spaceAfter=4),
    ]

def bullet(text):
    return Paragraph(f"• {text}", bullet_style)

# --- Build story ---
story = []

# Name + contact
story.append(Paragraph("CANDIDATE NAME", name_style))
story.append(Paragraph("email@example.com | +60 12-345 6789 | LinkedIn | City, Country", contact_style))

# Summary
story += section_heading("Professional Summary")
story.append(Paragraph("Summary text here.", body_style))
story.append(Spacer(1, 4))

# Experience
story += section_heading("Experience")
# For each role:
story.append(Paragraph("Job Title", job_title_style))
story.append(Paragraph("Company Name — City", company_style))
story.append(Paragraph("Jan 2022 – Present", date_style))
story.append(bullet("Achieved X by doing Y, resulting in Z."))
story.append(Spacer(1, 4))

# Education
story += section_heading("Education")
story.append(Paragraph("Degree Name", job_title_style))
story.append(Paragraph("University Name — City", company_style))
story.append(Paragraph("2018 – 2022", date_style))
story.append(Spacer(1, 4))

# Skills
story += section_heading("Skills")
story.append(Paragraph("Skill A • Skill B • Skill C • Skill D", body_style))

doc.build(story)
```

---

## Step 4 — Quality Checks

After generating the PDF, verify:
- [ ] File is not empty and opens correctly
- [ ] Name is prominent and readable
- [ ] All original sections and roles are present (nothing dropped)
- [ ] Dates are consistently formatted (e.g. `Jan 2022 – Mar 2024`)
- [ ] No orphaned headings at bottom of page
- [ ] Bullet points start with action verbs

If there are problems, fix the Python and regenerate.

---

## Step 5 — Present and Summarise

Use `present_files` to deliver the PDF. Then give the user a brief summary of the improvements made:
- What content was tightened/improved
- What structural changes were made
- Any placeholders added (e.g. `[quantify]`) where numbers were missing

---

## Key Rules

- **Never drop content** — every job, qualification, and skill in the original must appear in the output
- **Never invent facts** — if you strengthen a bullet, keep it factually grounded; use `[X%]` placeholders for missing numbers rather than guessing
- **Keep it to 1–2 pages** — flag to the user if the content is too long and suggest what to trim
- **Respect the user's voice** — improve clarity, don't replace their personality with corporate jargon
- **Use `--break-system-packages`** when installing Python packages: `pip install reportlab --break-system-packages`
