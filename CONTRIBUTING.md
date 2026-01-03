# Contributing to Sommelier Spark Documentation

This guide explains how to contribute to Sommelier Spark documentation across all projects.

---

## Where Documentation Lives

| Type of Documentation | Repository | Directory |
|-----------------------|------------|-----------|
| iOS User Guide | sommelier_spark_ios | `docs/user/` |
| iOS Developer Docs | sommelier_spark_ios | `docs/setup/` |
| iOS System Docs | sommelier_spark_ios | `docs/system/` |
| Web User Guide | sommelier_spark | `docs/user/` |
| Administrator Guide | sommelier_spark | `docs/admin/` |
| Web Developer Docs | sommelier_spark | `docs/` |
| Shared Resources | sommelier-spark-docs | `shared/` |
| Documentation Hub | sommelier-spark-docs | Root |

---

## Quick Reference: Where to Add New Docs

| If you're documenting... | Add it to... |
|--------------------------|--------------|
| iOS app feature for users | `sommelier_spark_ios/docs/user/` |
| iOS development process | `sommelier_spark_ios/docs/setup/` |
| iOS system/architecture | `sommelier_spark_ios/docs/system/` |
| Web app feature for users | `sommelier_spark/docs/user/` |
| Admin functionality | `sommelier_spark/docs/admin/` |
| Web development setup | `sommelier_spark/docs/` |
| Cross-project glossary term | `sommelier-spark-docs/shared/GLOSSARY.md` |

---

## Documentation Standards

### Required Metadata

All major documents should include a metadata header:

```markdown
| Document ID | Version | Date | Author |
|-------------|---------|------|--------|
| SS-XXX-NNN | 1.0 | January 2026 | Author Name |
```

**Document ID Format:**
- `SS-USR-NNN` — User documentation
- `SS-ADM-NNN` — Administrator documentation
- `SS-DEV-NNN` — Developer documentation
- `SS-SYS-NNN` — System documentation
- `SS-HUB-NNN` — Hub/meta documentation

### File Naming

| Type | Convention | Example |
|------|------------|---------|
| User Guides | `User_Guide_{Platform}.md` | `User_Guide_iOS.md` |
| Admin Guides | `Administrator_Guide.md` | — |
| Developer Guides | `{Topic}_Guide.md` | `Troubleshooting_Guide.md` |
| System Docs | `{System}_Documentation.md` | `Quiz_Unlock_System_Documentation.md` |
| Indexes | `DOCUMENTATION_INDEX.md` | — |

### Output Formats

| Document Type | Required Formats |
|---------------|------------------|
| User Guides | .md, .docx, .pdf |
| Admin Guides | .md, .docx, .pdf |
| Developer Docs | .md only |
| System Docs | .md only |
| Indexes | .md only |

Generate additional formats using Pandoc:
```bash
# Word document
pandoc Document.md -o Document.docx

# HTML (for PDF printing)
pandoc Document.md -o Document.html --standalone
```

---

## Style Guidelines Summary

See [shared/STYLE_GUIDE.md](shared/STYLE_GUIDE.md) for complete guidelines.

### Tone by Audience

| Audience | Tone | Example |
|----------|------|---------|
| End Users | Warm, encouraging | "You're making great progress!" |
| Administrators | Professional, clear | "Select the user from the list." |
| Developers | Technical, precise | "Run `npm install` to install dependencies." |

### Formatting Conventions

- Use **tables** for structured information
- Use **bullet lists** for steps and options
- Use **code blocks** for commands and code
- Use **bold** for UI elements and key terms
- Use **blockquotes** for tips and notes

### Section Structure

1. Table of Contents (for docs > 3 sections)
2. Overview/Introduction
3. Main content sections
4. FAQ (if applicable)
5. Glossary (for user-facing docs)
6. Version/metadata footer

---

## Review Process

### Before Submitting

1. **Spell check** — Run your content through a spell checker
2. **Link check** — Verify all links work
3. **Format check** — Ensure consistent formatting
4. **Accuracy check** — Test any procedures you document

### Pull Request Guidelines

1. Create a branch: `docs/description-of-change`
2. Make your changes
3. Update relevant indexes if adding new docs
4. Submit PR with description of changes

### What Gets Reviewed

- Technical accuracy
- Adherence to style guide
- Completeness
- Link validity
- Formatting consistency

---

## Updating Existing Docs

### When to Update

| Trigger | Action |
|---------|--------|
| Feature change | Update affected docs |
| Bug fix | Update troubleshooting if relevant |
| Version release | Update version numbers, changelogs |
| User feedback | Clarify confusing sections |

### Version Bumping

- **Major changes** (rewrite, restructure): Increment major version (1.0 → 2.0)
- **Additions** (new sections): Increment minor version (1.0 → 1.1)
- **Fixes** (typos, clarifications): No version change needed

### Updating the Hub

When you add or significantly update documentation:

1. Update `CONSOLIDATED_INDEX.md` in this repo
2. Update `shared/CHANGELOG.md` with the change
3. Update project-specific `DOCUMENTATION_INDEX.md`

---

## Creating New Documents

### Checklist

- [ ] Determine the correct repository and directory
- [ ] Assign a Document ID (if major doc)
- [ ] Include metadata header
- [ ] Follow naming conventions
- [ ] Include table of contents (if > 3 sections)
- [ ] Add to relevant index files
- [ ] Generate required output formats
- [ ] Update hub changelog

### Template

```markdown
# Document Title

| Document ID | Version | Date | Author |
|-------------|---------|------|--------|
| SS-XXX-NNN | 1.0 | Month Year | Your Name |

---

## Table of Contents

1. [Section One](#1-section-one)
2. [Section Two](#2-section-two)

---

## 1. Section One

Content here...

---

## 2. Section Two

Content here...

---

*Document ID — Document Title v1.0*
```

---

## Questions?

- Check the [STYLE_GUIDE.md](shared/STYLE_GUIDE.md) for detailed standards
- Review existing documents for examples
- Open an issue in this repository for clarification

---

*Contributing Guide — Sommelier Spark Documentation*
