# 0002. Content formats: AsciiDoc for course topics, JSON for the encyclopedia

- **Status:** Accepted
- **Date:** 2026-11-17

## Context

The knowledge base has two parts (see [concept](../../01-product/concept.md)):

- **Course topics** teach the fundamentals. They are free-form: any number of paragraphs and headings, step-by-step sequences with images, images with captions, general tables, highlighted notes and warnings, links and demonstration videos.
- **Encyclopedia entries** describe species, varieties, rootstocks, pests and diseases. Entries of the same kind have the same fields, including longer text fields (for a pest, for example symptoms, causes, prevention and treatment).

Both parts are stored in MongoDB ([ADR-0001](0001-knowledge-base-mongodb.md)), and references between documents are `{id, name}` pairs.

## Decision

1. **Course topics** are written in **AsciiDoc**.
   - AsciiDoc provides the needed building blocks out of the box: admonitions (NOTE, TIP, WARNING), image captions, tables and video embedding.
   - References to encyclopedia entries and other topics use a project-specific link syntax carrying the `{id, name}` pair. The website renders them as links, or as plain names if the target does not exist.
   - Tables in course topics hold general information only; species-specific data belongs in the encyclopedia, so the two cannot contradict each other.
2. **Encyclopedia entries** are **JSON** documents.
   - Each kind of entry has a fixed set of fields, so the data can be filtered and used by later features (planner, recommender).
   - Longer text fields are **plain text**, without markup.
   - Relationships are stored in **reference fields** — for example affected species, resistant varieties, related course topics — as lists of `{id, name}` pairs, and shown next to the text.
3. **Demonstration videos** are external videos, referenced from course topics, preferably from a partner garden center.
4. The development documentation (this repository) stays in **Markdown**; AsciiDoc is used only for course content.

## Reasons

- Course topics are prose with rich structure; a markup language fits them better than structured JSON blocks, and AsciiDoc needs fewer custom extensions than Markdown for admonitions, captions and videos.
- Encyclopedia entries are data first: fixed fields keep them filterable and machine-usable.
- Keeping links in reference fields instead of inline text keeps encyclopedia text simple and the relationships queryable.

## Consequences

- Positive: each part uses the format that fits its content.
- Negative: the website needs an AsciiDoc renderer (for example Asciidoctor.js) with an extension for the project link syntax.
- Negative: encyclopedia text cannot contain inline links, highlights or lists; anything that needs them belongs in a course topic or a reference field.
- Negative: external videos can disappear or change; their content is not under our control.
- Negative: two markup languages are in use (AsciiDoc for course content, Markdown for documentation).

## Open points

- The exact AsciiDoc link syntax for `{id, name}` references.
- The field list for each kind of encyclopedia entry.
