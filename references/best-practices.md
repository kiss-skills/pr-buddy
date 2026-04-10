# Claude Skills Authoring Best Practices

> Source: [Anthropic Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
> Last synced: 2026-03-20

This document serves as the reference standard for skill quality reviews in this repository.

---

## Core principles

### Concise is key

The context window is a public good. Your Skill shares it with everything else Claude needs to know — system prompt, conversation history, other Skills' metadata, and the actual request.

At startup, only the metadata (name and description) from all Skills is pre-loaded. Claude reads SKILL.md only when the Skill becomes relevant, and reads additional files only as needed. However, being concise in SKILL.md still matters: once Claude loads it, every token competes with conversation history and other context.

**Default assumption**: Claude is already very smart. Only add context Claude doesn't already have. Challenge each piece of information:
- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"
- "Does this paragraph justify its token cost?"

**Good example** (~50 tokens):
````markdown
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**Bad example** (~150 tokens):
```markdown
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but
pdfplumber is recommended because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

### Set appropriate degrees of freedom

Match the level of specificity to the task's fragility and variability.

**High freedom** (text-based instructions) — when multiple approaches are valid, decisions depend on context, heuristics guide the approach.

**Medium freedom** (pseudocode or scripts with parameters) — when a preferred pattern exists, some variation is acceptable, configuration affects behavior.

**Low freedom** (specific scripts, few or no parameters) — when operations are fragile and error-prone, consistency is critical, a specific sequence must be followed.

Think of Claude as a robot exploring a path:
- **Narrow bridge**: Only one safe way forward → specific guardrails and exact instructions.
- **Open field**: Many paths lead to success → general direction, trust Claude to find the best route.

### Test with all models you plan to use

Skills act as additions to models, so effectiveness depends on the underlying model.
- **Claude Haiku**: Does the Skill provide enough guidance?
- **Claude Sonnet**: Is the Skill clear and efficient?
- **Claude Opus**: Does the Skill avoid over-explaining?

What works perfectly for Opus might need more detail for Haiku.

---

## Skill structure

### YAML front matter requirements

The SKILL.md front matter requires two fields:

- `name`: Max 64 characters. Only lowercase letters, numbers, and hyphens. No XML tags. No reserved words ("anthropic", "claude").
- `description`: Non-empty. Max 1024 characters. No XML tags. Should describe what the Skill does and when to use it.

### Naming conventions

Use consistent naming patterns. Consider **gerund form** (verb + -ing) for Skill names.

**Good**: `processing-pdfs`, `analyzing-spreadsheets`, `managing-databases`

**Acceptable alternatives**: `pdf-processing`, `process-pdfs`

**Avoid**: Vague (`helper`, `utils`, `tools`), overly generic (`documents`, `data`, `files`), reserved words (`anthropic-helper`, `claude-tools`), inconsistent patterns within your collection.

### Writing effective descriptions

The `description` field enables Skill discovery and should include both **what** the Skill does and **when** to use it.

**Always write in third person.** The description is injected into the system prompt, and inconsistent point-of-view causes discovery problems.
- Good: "Processes Excel files and generates reports"
- Avoid: "I can help you process Excel files"
- Avoid: "You can use this to process Excel files"

**Be specific and include key terms.** Claude uses the description to choose the right Skill from potentially 100+ available Skills.

**Good examples**:
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```
```yaml
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

**Bad examples**:
```yaml
description: Helps with documents
```
```yaml
description: Processes data
```

### Progressive disclosure patterns

SKILL.md serves as an overview that points Claude to detailed materials as needed.

- Keep SKILL.md body under **500 lines** for optimal performance.
- Split content into separate files when approaching this limit.
- All reference files should link **directly from SKILL.md** (one level deep).

**Bad — too deep**:
```
SKILL.md → advanced.md → details.md (actual info)
```

**Good — one level deep**:
```
SKILL.md → advanced.md
SKILL.md → reference.md
SKILL.md → examples.md
```

### Structure longer reference files with table of contents

For reference files longer than 100 lines, include a table of contents at the top so Claude can see the full scope of available information even when previewing with partial reads.

---

## Workflows and feedback loops

### Use workflows for complex tasks

Break complex operations into clear, sequential steps. For particularly complex workflows, provide a checklist that Claude can copy and track progress.

### Implement feedback loops

**Common pattern**: Run validator → fix errors → repeat. This pattern greatly improves output quality.

---

## Content guidelines

### Avoid time-sensitive information

Don't include information that will become outdated. Use an "old patterns" section for historical context.

### Use consistent terminology

Choose one term and use it throughout. Avoid mixing synonyms (e.g., "API endpoint" vs "URL" vs "API route" vs "path").

---

## Common patterns

### Template pattern

Provide templates for output format. Match the level of strictness to your needs.

### Examples pattern

For Skills where output quality depends on seeing examples, provide input/output pairs.

### Conditional workflow pattern

Guide Claude through decision points with clear branching ("Creating new? → follow creation workflow. Editing existing? → follow editing workflow").

---

## Anti-patterns to avoid

- **Windows-style paths**: Always use forward slashes (`scripts/helper.py`), never backslashes.
- **Too many options**: Provide a default recommendation with an escape hatch, not a menu of 5 choices.
- **Deeply nested references**: Keep references one level deep from SKILL.md.
- **Over-explaining basics**: Don't explain concepts Claude already knows.
- **Vague descriptions**: Descriptions must include what the Skill does AND when to use it.

---

## Advanced: Skills with executable code

### Solve, don't punt

Handle error conditions in scripts rather than letting them fail for Claude to figure out.

### Provide utility scripts

Pre-made scripts are more reliable than generated code, save tokens and time, and ensure consistency.

### Create verifiable intermediate outputs

Use the plan-validate-execute pattern: create a structured plan file → validate with a script → execute. This catches errors early.

### Package dependencies

List required packages in your SKILL.md and verify they're available. Don't assume packages are installed.

---

## Checklist for effective Skills

### Core quality
- [ ] Description is specific and includes key terms
- [ ] Description includes both what the Skill does and when to use it
- [ ] SKILL.md body is under 500 lines
- [ ] Additional details are in separate files (if needed)
- [ ] No time-sensitive information (or in "old patterns" section)
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] File references are one level deep
- [ ] Progressive disclosure used appropriately
- [ ] Workflows have clear steps

### Code and scripts
- [ ] Scripts solve problems rather than punt to Claude
- [ ] Error handling is explicit and helpful
- [ ] No "voodoo constants" (all values justified)
- [ ] Required packages listed in instructions and verified as available
- [ ] Scripts have clear documentation
- [ ] No Windows-style paths (all forward slashes)
- [ ] Validation/verification steps for critical operations
- [ ] Feedback loops included for quality-critical tasks

### Testing
- [ ] At least three evaluations created
- [ ] Tested with Haiku, Sonnet, and Opus
- [ ] Tested with real usage scenarios
- [ ] Team feedback incorporated (if applicable)
