# Wiki Schema Configuration

This file governs how the LLM builds and maintains your Wiki. Edit it freely.

## Domain Focus
- Domain: Nutanix IAM (Kronos) — Global IAM fields, federation, and cluster topology (NC vs PC).
- Source notes live in `inbox/` and must not be overwritten by generated wiki pages.
- `wiki/patterns/` stores failure modes and strategies (WikiSkill). Entity/concept pages are facts; pattern pages are how-we-fail / how-we-work. Skills in `~/.cursor/skills/` must not duplicate the whole wiki.
- Tickets (`ENG-*`) are **entities** of subtype `project`.
- Products and platforms (`Nutanix Central`, `Prism Central`, `NCM`, `NKP`, `AHV`) are **entities** of subtype `product`.
- Repositories (`iam-themis`, `ntnx-api-iam`, `iam-utils`, `iam-bootstrap`) are **entities** of subtype `organization` or `other`.
- Fields and mechanisms (`authoringScope`, `products`, global ACP, identity federation, lattice, shard copy, mutation guard) are **concepts** of subtype `term` or `method`.
- People named in reviews are **entities** of subtype `person`.
- Keep NC (Nutanix Central) and PC (Prism Central) as separate entities; never collapse them.

## Wiki Structure
- Entity pages: `entities/` — entity subtype tags are runtime-injected (see Settings → Tag Vocabulary); this file holds no list
- Concept pages: `concepts/` — concept subtype tags are runtime-injected (see Settings → Tag Vocabulary); this file holds no list
- Source pages: `sources/`
- Index: `index.md`
- Log: `log.md`

## Entity Page Template
Pages in `entities/` MUST follow this structure:

**Frontmatter fields:**
- `type: entity` — page category (MUST be exactly "entity")
- `created:` — ISO date of first creation
- `sources:` — array of source file wiki-links
- `tags:` — entity subtype; the valid values are runtime-injected by the **Active Tag Vocabulary** section of every system prompt (driven by Settings). MUST be one of those values.
- `aliases:` (optional) — alternative names (translations, abbreviations)
- `reviewed:` (optional) — if true, page is human-verified and protected

**Sections:**
1. **Description**: 3-6 sentences with concrete facts, bidirectional links
2. **Related Entities**: Links to related entities using [[entities/...]]
3. **Related Concepts**: Links to related concepts using [[concepts/...]]
4. **Mentions in Source**: Verbatim quotes with source attribution — see [Mentions Format](#mentions-format) below

## Concept Page Template
Pages in `concepts/` MUST follow this structure:

**Frontmatter fields:**
- `type: concept` — page category (MUST be exactly "concept")
- `created:` — ISO date of first creation
- `sources:` — array of source file wiki-links
- `tags:` — concept subtype; the valid values are runtime-injected by the **Active Tag Vocabulary** section of every system prompt (driven by Settings). MUST be one of those values.
- `aliases:` (optional) — alternative names (translations, abbreviations)
- `reviewed:` (optional) — if true, page is human-verified and protected

**Sections:**
1. **Definition**: Clear, concise definition
2. **Key Characteristics**: Bullet list of defining traits
3. **Applications**: Real-world usage scenarios
4. **Related Concepts**: Links using [[concepts/...]]
5. **Related Entities**: Links using [[entities/...]]
6. **Mentions in Source**: Verbatim quotes with source attribution — see [Mentions Format](#mentions-format) below

## Naming Conventions
- Filenames: lowercase-with-hyphens (slugified)
- Entity/concept names: Preserve original language from source, NEVER translate
- Wiki-links: Use full paths [[entities/page-name|Display Name]] or [[concepts/page-name|Display Name]]
- Ticket pages: `entities/eng-NNNNNN.md` with alias equal to the full ticket title
- Product short names stay uppercase in display text (`NC`, `PC`) even when the filename is slugified

## Source Page Template
Pages in `sources/` MUST follow this structure:

**Frontmatter fields:**
- `type: source` — page category (MUST be exactly "source")
- `tags:` — INHERITED from the source note's frontmatter (do NOT use LLM-derived concept names). The system programmatically populates this from the source file; the LLM must not overwrite it with extracted concept names. This preserves the user's existing tag vocabulary and prevents pollution from LLM hallucinations.
- `sources:` — array of related wiki page links created from this source
- `created:` / `updated:` — set by the system, see Date Fields below

**Sections:**
1. **Summary**: Brief description of the source content (2-4 sentences)
2. **Key Points**: Bullet list of main insights
3. **Mentioned Pages**: List of [[entities/...]] and [[concepts/...]] pages created from this source

## Date Fields
- `created:` and `updated:` are filled by the system programmatically — NEVER LLM-generated
- The LLM may produce wrong dates during extraction; the system overrides them post-write to ensure correctness
- `created:` is preserved on merge (older value kept); `updated:` is always set to the current date
- `source_note:` (optional) — wiki-link to the original source file

## Mentions Format
"Mentions in Source" entries use academic-footnote style with source attribution. The format is:
- "Verbatim quote in original language (optional translation)" — [[source-name|display-name]]

Rules:
- Quotes must be VERBATIM — never paraphrase, summarize, or translate away the original
- The source wiki-link is required so future page merges can trace each quote to its origin
- Multiple quotes from the same source go in the same block, separated by newlines

## Content Rules
- mentions_in_source MUST be VERBATIM quotes — never paraphrase or translate
- Summaries/descriptions should use the wiki output language
- Entity/concept names must match the source file's original language exactly
- All pages must include bidirectional links where relevant
- Do not invent PR numbers, commit SHAs, CI results, or reviewer positions
- Prefer linking related Kronos tickets to each other (authoringScope → mutation guard → products → global ACP)
- Record open questions on the source page; only promote a fact to an entity/concept if the source states it

## Classification Rules
- **type field:** entity | concept | source — the page category
- **tags field:** stores the subtype (entity_type or concept_type)
- Entity subtypes / Concept subtypes: see the **Active Tag Vocabulary** section injected into every system prompt (driven by Settings). The valid values are NOT listed here — relying on a baked list in this file would drift from your Settings whenever you change the vocabulary.
- Source types: document, conversation, note
- **Rule:** tags MUST only contain values from the Active Tag Vocabulary in the system prompt. A tag not in that list will be removed by the system.
- `ENG-*` tickets → entity / project
- NC, PC, NCM, NKP, Prism, AHV → entity / product
- iam-themis, ntnx-api-iam, iam-utils, iam-bootstrap → entity / other
- API fields, guards, federation, lattice, shard copy → concept / term or method

## Multi-Source Merge Rules
- Sources array: Append new sources, never overwrite
- Aliases: Append alternative names (translations, abbreviations) without overwriting existing ones
- reviewed flag: If true, preserve all existing content, only append genuinely new info
- Contradictions: Preserve both sides with attribution, add to ## Contradictions section
- NO_NEW_CONTENT: Return this signal if source adds nothing new

## Maintenance Policies
- Stale threshold: 90 days without updates
- Contradiction severity: warning, conflict, error
- Orphan page: no inbound links from other wiki pages
- Missing page: referenced by [[link]] but does not exist
