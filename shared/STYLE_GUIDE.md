# Sommelier Spark — Documentation Style Guide

Standards and conventions for all Sommelier Spark documentation.

---

## Table of Contents

1. [Tone by Audience](#1-tone-by-audience)
2. [Document Structure](#2-document-structure)
3. [Formatting Conventions](#3-formatting-conventions)
4. [Writing Guidelines](#4-writing-guidelines)
5. [Metadata Requirements](#5-metadata-requirements)
6. [Templates](#6-templates)

---

## 1. Tone by Audience

### End Users (Staff)

**Tone:** Warm, encouraging, supportive

**Characteristics:**
- Use "you" and "your" frequently
- Celebrate progress and achievements
- Acknowledge learning challenges
- Provide reassurance and tips

**Example:**
> "You're making great progress! If you're finding a topic challenging, try reviewing the related wines in your Wine Library first."

**Avoid:**
- Technical jargon
- Assuming prior knowledge
- Condescending explanations

---

### Administrators

**Tone:** Professional, clear, action-oriented

**Characteristics:**
- Direct and efficient
- Focus on tasks and outcomes
- Include specific steps
- Reference related features

**Example:**
> "To suspend a user, navigate to Admin → Users, locate the user in the list, and click the Suspend button. The user will immediately lose access but their data will be preserved."

**Avoid:**
- Overly casual language
- Unnecessary filler
- Vague instructions

---

### Developers

**Tone:** Technical, precise, practical

**Characteristics:**
- Use correct technical terminology
- Include code examples
- Reference file paths and commands
- Explain the "why" when helpful

**Example:**
> "Run `npm run build` to create a production build. This generates a static export in the `out/` directory, which Capacitor will bundle into the iOS app."

**Avoid:**
- Oversimplification
- Omitting important details
- Untested commands

---

## 2. Document Structure

### Standard Sections

| Section | Required For | Purpose |
|---------|--------------|---------|
| Metadata Header | Major docs | Version tracking |
| Table of Contents | Docs > 3 sections | Navigation |
| Introduction | All | Context setting |
| Main Content | All | Core information |
| FAQ | User docs | Common questions |
| Glossary | User docs | Term definitions |
| Quick Reference | Developer docs | Command lookup |
| Footer | Major docs | Document ID reference |

### Section Numbering

Use numbered sections for documents with 4+ sections:

```markdown
## 1. Introduction
## 2. Getting Started
## 3. Main Feature
## 4. Advanced Topics
```

Use unnumbered sections for shorter documents:

```markdown
## Overview
## Installation
## Usage
```

---

## 3. Formatting Conventions

### Text Formatting

| Element | Format | Example |
|---------|--------|---------|
| UI elements | **Bold** | Click **Save** |
| Key terms (first use) | **Bold** | A **module** contains lessons |
| File paths | `Code` | Open `docs/README.md` |
| Commands | `Code` | Run `npm install` |
| Variables | `Code` with description | Set `API_KEY` to your key |
| Emphasis | *Italics* | This is *very* important |

### Code Blocks

Use fenced code blocks with language identifiers:

````markdown
```bash
npm run dev
```

```typescript
const config: Config = {
  debug: true
};
```
````

### Tables

Use tables for structured information:

```markdown
| Column A | Column B | Column C |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
```

Align headers and content appropriately:
- Left-align text columns
- Right-align numeric columns (if needed)

### Lists

**Use bullet lists for:**
- Options or features
- Unordered items
- Requirements

**Use numbered lists for:**
- Sequential steps
- Prioritized items
- Procedures

### Tips and Notes

Use blockquotes for tips, notes, and warnings:

```markdown
> **Tip:** This is helpful advice.

> **Note:** This is important information.

> **Warning:** This could cause problems.
```

---

## 4. Writing Guidelines

### Voice

- Use **active voice** over passive voice
- Use **second person** ("you") for instructions
- Use **present tense** for current state

| Avoid | Prefer |
|-------|--------|
| "The button should be clicked" | "Click the button" |
| "It is recommended that users..." | "We recommend you..." |
| "The file will be saved" | "The file saves automatically" |

### Clarity

- One idea per sentence
- Short paragraphs (3-4 sentences max)
- Define terms on first use
- Use consistent terminology

### Instructions

1. Start with the action verb
2. Be specific about locations
3. Describe the expected result

**Example:**
> Click **Settings** in the top-right corner. The Settings panel opens, showing your account options.

### Accessibility

- Don't rely solely on color to convey meaning
- Provide alt text for images
- Use descriptive link text (not "click here")

---

## 5. Metadata Requirements

### Document ID Format

```
SS-{TYPE}-{NUMBER}
```

| Type Code | Category |
|-----------|----------|
| USR | User documentation |
| ADM | Administrator documentation |
| DEV | Developer documentation |
| SYS | System documentation |
| HUB | Hub/meta documentation |
| DOC | Documentation indexes |

### Metadata Header

All major documents require:

```markdown
| Document ID | Version | Date | Author |
|-------------|---------|------|--------|
| SS-USR-001 | 1.0 | January 2026 | Author Name |
```

### Version Numbers

- **1.0** — Initial release
- **1.1** — Minor additions
- **1.2** — More additions
- **2.0** — Major revision/rewrite

### Footer

End major documents with:

```markdown
---

*SS-USR-001 — Document Title v1.0*
```

---

## 6. Templates

### User Guide Section

```markdown
## Section Title

Brief introduction to what this section covers.

### Subsection

Explanation of the feature or concept.

**To do something:**

1. First step
2. Second step
3. Third step

> **Tip:** Helpful advice related to this section.

### Related Topics

- Link to related section
- Link to another related topic
```

### Developer Guide Section

```markdown
## Section Title

What this section covers and why it matters.

### Prerequisites

- Requirement 1
- Requirement 2

### Steps

1. **Step Name**

   ```bash
   command here
   ```

   Explanation of what this does.

2. **Next Step**

   Continue with instructions...

### Verification

How to confirm success:

```bash
verification command
```

Expected output: description

### Troubleshooting

| Problem | Solution |
|---------|----------|
| Error X | Fix Y |
```

### FAQ Section

```markdown
## Frequently Asked Questions

### Question in natural language?

Answer in conversational tone. Keep it concise but complete.

### Another common question?

Another helpful answer. Include links to relevant sections if applicable.
```

---

*Sommelier Spark Documentation Style Guide — January 2026*
