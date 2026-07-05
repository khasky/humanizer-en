# Humanizer EN

A skill for AI agents. Removes the traces of machine generation from English text. The English counterpart of [humanizer-ru](https://github.com/Vladimir-Human/humanizer-ru).

## Install and integration

The humanizer-en skill installs into Claude.ai and into the local Claude Code CLI. For teams there is a separate org-level install path.

### 1. Claude.ai (web)

1. Download the repository as a ZIP archive:
   `https://github.com/khasky/humanizer-en/archive/refs/heads/main.zip`
2. Sign in to Claude.ai and go to **Settings** > **Skills**.
3. Click **Upload skill** and choose the downloaded ZIP.

> **Note.** If Claude.ai rejects an archive downloaded straight from GitHub because of the nested `humanizer-en-main` folder, clone the repo and zip the folder by hand:
> ```bash
> git clone https://github.com/khasky/humanizer-en.git
> zip -r humanizer-en.zip humanizer-en/
> ```

### 2. Organizations (Enterprise & Team)

An org admin uploads the skill to the shared library — it becomes available to the whole team.

### 3. API and local agents (Claude Code)

When using the API (the `/v1/messages` endpoint or equivalents), pass the skill via the container.skills parameter — see the docs of your client.

For local use via [skills.sh](https://skills.sh):

```sh
npx skills add khasky/humanizer-en
```

Or by hand:

```sh
mkdir -p ~/.claude/skills
git clone https://github.com/khasky/humanizer-en.git ~/.claude/skills/humanizer-en
```

Or just the skill file:

```sh
mkdir -p ~/.claude/skills/humanizer-en
cp SKILL.md ~/.claude/skills/humanizer-en/
```

## Usage

In Claude Code or another agent:

```text
/humanizer-en [paste text]
```

Or directly:

```text
Humanize this text: [your text]
```

## What it does

Detects and fixes 36 patterns of machine-written English (25 base + 11 extensions). Built on [Wikipedia:Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) and [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup).

Since v2.3, SKILL.md is a map with a decision tree. The full description of the patterns and checks lives in the loadable `references/` files.

### Architecture

```
humanizer-en/
├── SKILL.md                              # Map, decision tree, checklist
├── README.md                             # This file
├── scripts/
│   └── check_markers.py                  # Auto-run of every regex over samples
├── .github/workflows/
│   ├── no-ai-cliches.yml                 # The skill's own text is checked for clichés
│   ├── regex-check.yml                   # Marker run in CI
│   └── self-scan.yml                     # The skill scans itself for markers
└── references/
    ├── content-patterns.md               # Content patterns #1–9 + #6a
    ├── language-patterns.md              # Language patterns #10–15 + #15a–15f
    ├── structural-style-patterns.md      # Structural and style #16–21 + #21a
    ├── communication-patterns.md         # Communicative #22–25 + extensions
    ├── chatbot-artifacts.md              # Unambiguous markers with regexes
    ├── source-fabrication.md             # Source-citation checks
    ├── false-positives.md                # What is NOT an AI tell
    ├── llm-fingerprints.md               # Model fingerprints (July 2026)
    └── test-fixtures.md                  # Test samples for the regexes
```

### Content patterns

| # | Pattern | Severity |
|---|---------|----------|
| 1 | Regression to the mean — concrete facts replaced by empty praise ("eminent", "titan") | 🔴 |
| 2 | Inflated significance — "a pivotal moment in the industry's history" | 🟡 |
| 3 | Media-presence emphasis — "cited by NYT, BBC, Forbes" with no context | 🟡 |
| 4 | Participle tails — "highlighting… reflecting… underscoring…" | 🟡 |
| 5 | Promotional language — "hidden gem", "nestled in the heart of" | 🟡 |
| 6 | Vague attributions — "experts believe" with no source | 🔴 |
| 7 | Challenges and prospects — "despite the challenges, it continues to thrive" | 🟢 |
| 8 | Officialese and corporate jargon — "utilize", "in order to", "leverage" | 🟡 |
| 9 | Text about the text — describing the article instead of the subject | 🟡 |
| 6a | Named pseudo-attribution of the RAG era — "the critic underscored its enduring influence" with no real quote | 🟡 |

### Language patterns

| # | Pattern | Severity |
|---|---------|----------|
| 10 | AI vocabulary — "delve, tapestry, seamless, robust, testament, boasts" | 🔴 |
| 11 | Avoiding "to be" — "serves as" instead of "is" | 🟡 |
| 12 | "Not only… but also" — negative parallelism | 🟡 |
| 13 | Rule of three — forced triples | 🔴 |
| 14 | Synonym chasing — "the hero… the protagonist… the central figure" | 🟢 |
| 15 | False ranges — "from the Big Bang to dark matter" | 🟡 |
| 15a | Dangling / misplaced modifiers — "Using this method, results improve" | 🟡 |
| 15b | Hedging cascade — "perhaps, in some cases, depending on…" | 🟡 |
| 15c | Transition crutches and conclusion filler — "However, it's worth noting…", "In conclusion…" | 🟡 |
| 15d | Abrupt style shift within one text | 🟢 |
| 15e | Formulaic collocations — "a testament to", "navigate the complexities", "at the heart of" | 🟡 |
| 15f | Lack of idiom — a long text with no living turn of phrase | 🟢 |

### Structural and style patterns

| # | Pattern | Severity |
|---|---------|----------|
| 16 | Excess em-dashes and bold | 🔴 |
| 17 | Emoji lists — 🚀 **Speed:** … | 🔴 |
| 18 | Quotation marks — curly quotes as a weak tell (heavy autocorrect caveat) | 🟡 |
| 19 | Excessive tables — a 2–3 row table where prose is clearer | 🔴 |
| 20 | Markdown residue — `**bold**`, `#headings` in plain text | 🔴 |
| 21 | Heading-hierarchy violation — a jump from H1 to H3 | 🔴 |
| 21a | Boilerplate section headings — "Introduction", "Conclusion", "Key Takeaways" | 🔴 |

### Communicative patterns

| # | Pattern | Severity |
|---|---------|----------|
| 22 | Leftover chat turns and templates — "Hope this helps!", `[insert name]` | 🔴 |
| 23 | Knowledge-limit disclaimers — "while specific details are limited…" | 🟡 |
| 23a | Statement of unavailability with speculation — "the data is not published, however it is likely…" | 🟡 |
| 24 | Sycophantic tone — "Great question!" | 🟡 |
| 24a | Pseudo-therapeutic register and fake liveliness — "You're not wrong to feel that way", "Short. Punchy. Deliberate." | 🟡 |
| 25 | Generic positive conclusions — "the future looks bright" | 🟡 |
| 25a | Mid-sentence cutoff — the text ends in the middle of a sentence | 🟡 |

### Unambiguous markers (new in v2.3, extended in v2.5–v2.9)

Regexes for chatbot copy-paste traces. One such marker in ordinary text is almost certainly AI. Every regex is run automatically: `python3 scripts/check_markers.py` — three sample levels each, mandatory in CI (23 of 23 pass). The same script scans arbitrary text: `python3 scripts/check_markers.py --scan file.md`. A third workflow (`self-scan.yml`) runs the same regexes over the project's own text on every change.

| Marker | Source | Regular expression |
|---|---|---|
| `:contentReference[oaicite:N]{index=N}` | OpenAI ChatGPT | `:contentReference\[oaicite:\d+\]\{index=\d+\}` |
| `oai_citation:N‡` | OpenAI ChatGPT | `oai_citation:\d+‡` |
| `turn0search0`, `turn0fetch0` | OpenAI web search | `turn\d+(search|fetch)\d+` |
| `?utm_source=chatgpt.com` | OpenAI ChatGPT | `[?&]utm_source=chatgpt\.com` |
| `?utm_source=openai` | OpenAI API | `[?&]utm_source=openai` |
| `attached_file://` | OpenAI ChatGPT | `attached_file:\/\/` |
| `grok_card://` | xAI Grok | `grok_card:\/\/` |
| `vertexaisearch.cloud.google.com/grounding-api-redirect` | Google Gemini | `vertexaisearch\.cloud\.google\.com/grounding-api-redirect` |
| `[^N^]` | Microsoft Copilot | `\[\^\d+\^\]` |
| `【N†source】` | OpenAI Assistants | `【\d+(?::\d+)?†source】` |
| `citeturn0file0` | OpenAI ChatGPT (stream) | `citeturn\d+[a-z]+\d+` |
| `turn0file2`, `fileciteturn0file2turn0file6` | OpenAI file_search | `turn\d+file\d+` |
| `](sandbox:/mnt/data/…)` | OpenAI ChatGPT (data analysis) | `\]\(sandbox:/mnt/data/` |
| Invisible chars `U+E200–E204` | OpenAI ChatGPT (citation control separators) | `[\ue200-\ue204]` |
| `<think>…</think>` | DeepSeek and other reasoning models | `</?think>` |
| Run-on `ISO+3ISO+3` | OpenAI ChatGPT (footnote render error) | `[A-Za-z)]\+\d+[A-Z]` |
| `[cite_start]` | Google Gemini (PDF analysis) | `\[cite_start\]` |
| `[cite: 8]`, `[Cite: 12]` | Google Gemini (source-fragment reference) | `\[[Cc]ite:\s?\d+\]` |
| Zero-width `U+200B`–`U+200D`, `U+2060`, `U+FEFF` | OpenAI o3/o4-mini and successors; EU AI Act Article 50 marking | `[\u200b-\u200d\u2060\ufeff]` |

The full list with reference samples is in `references/test-fixtures.md`.

### Source fabrication (new in v2.3)

A separate class of checks for text with citations: 404, a DOI that resolves to a different article, a non-existent ISBN, an author who died before the publication date, a book citation with no page numbers. See `references/source-fabrication.md`.

### False-positive boundaries (new in v2.3)

Em-dashes in Emily Dickinson, curly quotes from macOS autocorrect, the rule of three in rhetorical prose, officialese in a legal document, Title Case in headings — these are **not** AI tells. The skill deliberately does not "fix" them. See `references/false-positives.md`.

### Model fingerprints (new in v2.3)

Stylistic tells by vendor, current as of July 2, 2026: OpenAI GPT-5.5 (flagship since April 23, 2026), Anthropic Claude Fable 5 (global since July 1, 2026) / Sonnet 5 (June 30, 2026) / Opus 4.8, Google Gemini 3.5 Flash (standard after Google I/O 2026) and Deep Research mode, xAI Grok 4.3, DeepSeek V4, Qwen 3.7, Meta Muse Spark, Mistral Large 3 / Magistral, Perplexity Sonar, Amazon Nova 2, Cohere Command A+. Freshness: through September 30, 2026; unscheduled review August 2, 2026 (Article 50 of the EU AI Act takes effect). See `references/llm-fingerprints.md`.

**Severity scale:** 🔴 instantly gives away AI · 🟡 strong signal · 🟢 weak signal

## Example

**Before:**

> 🚀 **Innovation:** This software is undoubtedly a testament to our commitment to quality. Moreover, it delivers a seamless, intuitive, and powerful user experience — ensuring efficiency. Experts believe this is a revolution.

**After:**

> We added batch processing, keyboard shortcuts, and offline mode. Testers say tasks finish faster.

## Differences from the Russian edition (humanizer-ru)

- Pattern #8 is reframed as English officialese and corporate jargon ("utilize", "in order to", "leverage", nominalizations) rather than Russian bureaucratese.
- Pattern #15a is the English dangling / misplaced modifier ("Using this method, results improve"), the native-English form of the Russian gerund error.
- Pattern #15e is repurposed to formulaic English collocations ("a testament to", "navigate the complexities"); the Russian slot was about calques from the English semantic field, which does not apply here.
- Pattern #18 (quotes) treats curly vs straight quotes as a weak tell with a heavy autocorrect caveat — there is no guillemet rule.
- Pattern #21a is repurposed to boilerplate section headings ("Introduction", "Conclusion"); Title Case, which the Russian edition fought as a calque, is instead listed among the ineffective indicators (`false-positives.md`).
- Sources cite English Wikipedia only; the run-on regex (`[A-Za-z)]\+\d+[A-Z]`) is Latin-only.

## Sources

- [Wikipedia:Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing)
- [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup)

## Versions

- **2.9.0** (July 4, 2026) — Gemini artifacts, Unicode watermarks, and the regulatory layer. Two new unambiguous-marker sections: Gemini citation tags `[cite_start]` / `[cite: 8]` leaking during PDF analysis (section A.9), and zero-width characters `U+200B`–`U+200D`, `U+2060`, `U+FEFF` (section A.10; homoglyph spaces like `U+202F` are left to manual review — French typography and layout cause false positives). Two pattern extensions from English "Signs of AI writing": #6a "Named pseudo-attribution of the RAG era" and #23a "Statement of unavailability with speculation". In the false-positive guard — §14 "Non-standard spaces". In the fingerprints — Article 50 of the EU AI Act (from August 2, 2026), the code of practice of June 10, 2026, and the Anthropic Claude Fable 5 / Sonnet 5 flagship line (June–July 2026). 36 patterns, 23 regexes (23 of 23 pass). Existing rules untouched.
- **2.8.0** (June 11, 2026) — brainstorm over fresh catalogs and 2026 research. New unambiguous marker: file_search markers `turn0file2` / `fileciteturn0file2turn0file6` (OpenAI developer forum, strongest on GPT-5.2). Two pattern extensions: #15e "Formulaic collocations and AI-favourite phrases", #15f "Lack of idiom" (arXiv:2405.09279), #24a "Pseudo-therapeutic register and fake liveliness" (legacy of the withdrawn GPT-4o mode). In the false-positive guard — §13 "Different error types in humans and models". In the fingerprints — syntactic stylometry (93–98% accuracy) and density uniformity (DivEye). The checker gained a `--scan` mode for arbitrary text, and a third workflow `self-scan.yml` was added. 35 patterns, 20 regexes (20 of 20 pass). Existing rules untouched.
- **2.7.0** (June 11, 2026) — reconciliation with the current "Signs of AI writing" and the AI Cleanup project. New unambiguous marker: "Source+digit" run-ons (`ISO+3ISO+3`) — a ChatGPT footnote render error (section A.8). Two pattern extensions: #21a "Boilerplate section headings" and #25a "Mid-sentence cutoff". In the false-positive guard — "Ineffective indicators" and "Human syntax" (per the corpus study, PNAS 2025). In source checking — the stale access date. 31 patterns, 19 regexes (19 of 19 pass). Existing rules untouched.
- **2.6.0** (June 11, 2026) — automated marker checking and two new unambiguous markers. Added `scripts/check_markers.py`: a run of every regex across three sample levels on the Python standard library, wired into CI (`regex-check.yml`). In `chatbot-artifacts.md` — section A.7: invisible ChatGPT citation control chars `U+E200–U+E204` and leftover reasoning tags `<think>` from DeepSeek and similar models. 18 regexes (18 of 18 pass). Existing rules untouched.
- **2.5.0** (May 31, 2026) — substantive update. Four new unambiguous markers in `chatbot-artifacts.md`: Microsoft Copilot footnote `[^N^]`, OpenAI Assistants marker `【N†source】`, ChatGPT streaming marker `citeturn0file0`, broken data-analysis link `](sandbox:/mnt/data/`. 16 regexes (16 of 16 pass). In `llm-fingerprints.md`, added Perplexity, Amazon Nova, Cohere Command A+, Gemini Deep Research mode. In `false-positives.md`, added the section on academic and scientific register; the rule of three gained a boundary for opinion writing.
- **2.4.0** (May 23, 2026) — rewrote the skill description. Activation words expanded from four to thirty-plus: explicit synonyms ("humanize", "make it sound human", "AI detector", "check for AI", "de-slop", "reads like a chatbot"), seven models by name. Added a "do not use" section with explicit boundaries: non-English text, code and scripts, legal documents, literary prose, poetry, and résumés. The skill body is unchanged — the edit is in the header only.
- **2.3.0** (May 20, 2026) — SKILL.md turned into a map with links to the detail in `references/`. Added unambiguous markers with regexes (`chatbot-artifacts.md`), source-fabrication checks (`source-fabrication.md`), false-positive boundaries (`false-positives.md`), model fingerprints (`llm-fingerprints.md`), and test fixtures (`test-fixtures.md`). Four extensions (#15a–15d): dangling modifiers, hedging cascade, transition crutches and conclusion filler, abrupt style shift. Regexes run: 12 of 12 pass. Nothing old was broken.
- **2.2.0** — Updated compatibility (added Claude.ai), added metadata tags, edge-case handling (non-English text, code), fixed the install instructions. Strengthened patterns #1 and #13 with sub-variants (horoscope statements, symmetric sections).
- **2.1.0** — Added an explicit activation trigger to the description per the agentskills.io spec. Added license, compatibility, and metadata fields at the top of the file.
- **2.0.0** — Added patterns #1 (Regression to the mean), #9 (Text about the text), #19 (Tables), #20 (Markdown), #21 (Hierarchy). The 🔴/🟡/🟢 severity scale. Expanded the marker dictionaries. 25 patterns.
- **1.0.0** — First release. 22 patterns.

## Credits

English adaptation of [humanizer-ru](https://github.com/Vladimir-Human/humanizer-ru) by Vladimir-Human, used under the MIT License. The pattern catalog and file architecture originate in that project; this repository adapts and rewrites them for English text.

## License

MIT — see [LICENSE](LICENSE).
