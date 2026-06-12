# Writing Style Guide

Derived from the [ballerina-platform/ballerina-library](https://github.com/ballerina-platform/ballerina-library/tree/main/docs) documentation corpus.

---

## Document structure

Every document opens with:

1. An H1 title.
2. A metadata block immediately after the title:

```
_Authors_: @handle1 @handle2 \
_Reviewers_: @handle3 \
_Created_: YYYY/MM/DD \
_Updated_: YYYY/MM/DD
```

3. A brief purpose statement (1–3 sentences, no "In this document we will cover…" framing).
4. A table of contents for documents longer than ~4 sections.

Sections use H2. Subsections use H3. Avoid nesting deeper than H3.

---

## Voice and person

- Write in the third person: "The release owner should…", "Repo maintainers must…".
- Use first-person plural ("we") only when expressing shared goals or values, not for instructions.
- No contractions. Use "do not" not "don't", "it is" not "it's".

---

## Modal verbs

Follow RFC 2119 intent and italicise the modal:

| Word | Meaning |
|---|---|
| _must_ | Mandatory — no exceptions. |
| _should_ | Strongly recommended — deviation needs a reason. |
| _recommended_ | Advisory — good practice but context-dependent. |

---

## Lists

- **Numbered lists** for sequential steps where order matters.
- **Bullet lists** for parallel, unordered items.
- Keep nesting shallow — two levels maximum.
- Use a list for three or more parallel items in prose (responsibilities, criteria, steps) — do not chain them with commas in a sentence.
- Short example enumerations (2–3 items, often parenthetical) stay inline.
- Semicolon-separated facts are acceptable only inside table cells; if a cell needs more than ~3, move the content to prose below the table.

---

## Callouts

Use a blockquote with bold label for important asides:

```
> **Note:** Even though this is not a necessity, it is advised to maintain these patch branches.
```

---

## Formatting conventions

- Wrap code, file names, commands, repo names, and version strings in backticks: `gradle.properties`, `package.json`, `1.4.x`.
- Bold a term the first time it is introduced as a key concept.
- Use **Title Case** for H1 and H2 headings.
- Date format: `YYYY/MM/DD`.
- Reference people by their GitHub handle with an `@` prefix: `@NipunaRanasinghe`.

---

## Prose style

- Lead each section with a direct purpose statement: "This section explains…", "The goal is to…".
- Keep sentences short and declarative. Avoid padding adverbs ("basically", "simply", "obviously").
- Prefer plain verbs over idioms and metaphors: "close and create" not "roll", "obtain approval" not "shepherd", "create a branch" not "cut a branch". Established technical terms with no plain equivalent ("cherry-pick", "code freeze") are fine.
- State what something _is_ before explaining how to use or configure it.
- Avoid restating the section heading in the opening sentence.

---

## What to avoid

- "In this document, we will cover…" openers — the title and TOC already do this.
- Passive constructions where an actor exists: prefer "Module owners update the sheet" over "The sheet is updated by module owners".
- Hedging without substance: "It may be possible to consider…" — commit to a recommendation or state the condition directly.
