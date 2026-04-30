---
name: voice-analyser
description: Build a structured tone-of-voice document from 3 writing samples (1 social post, 1 article/blog, 1 corporate piece like an email). Use when the user wants to capture brand voice, create a voice guide, document tone of voice, extract writing style, build a TOV doc, analyse how a person or brand writes, or set up voice context for downstream content generation. TRIGGERS on "voice analyser", "tone of voice", "TOV doc", "create a voice guide", "extract brand voice", "analyse my writing", "document my voice", "/voice-analyser", or any request to systematically capture how a person or brand sounds in writing. Outputs a 16-section tone-of-voice.md ready to be loaded as context by content-generating agents.
---

# Voice Analyser

Build a structured tone-of-voice document by analysing real writing samples across three formats. The output is a 16-section markdown file that downstream content agents load as context to write in the brand's authentic voice.

## What this produces

A `tone-of-voice.md` file in the working directory (or a path the user specifies) that captures:

- Regional language and cultural style
- Brand personality and tone spectrum across contexts
- Vocabulary preferences (use, avoid, forbidden)
- Grammatical, punctuation, sentence and paragraph patterns
- Channel variations (email vs social vs sales vs support)
- Real quoted examples from the user's own samples
- A scoring rubric for validating future content

A portable Claude Code skill for capturing brand voice from real writing samples. Use this anywhere: clients, internal projects, ad-hoc work.

## Hard rule: three samples in three formats

You MUST collect **three samples across three different formats** before running analysis. This is non-negotiable. Single-format samples produce voice docs with major blind spots:

| Only this | Misses |
|---|---|
| Social only | Long-form structure, depth, argument flow. Voice looks shallower than it is. |
| Blog only | Conversational register, brevity, energy. Voice looks more academic than it is. |
| Corporate only | Personality, humour, casual lexicon. Voice looks stiffer than it is. |

The three required formats:

1. **Social post** (LinkedIn, X, Instagram caption). 100 to 300 words.
2. **Long-form** (blog post, article, newsletter, book excerpt). 500+ words.
3. **Corporate piece** (client email, sales message, formal note). 200 to 500 words.

If the user provides only one or two formats, ask for the missing ones before proceeding. Do not run analysis on incomplete inputs.

## Workflow

Run these steps in order. Do not skip ahead.

### Step 1: Intake

Ask in a single message:

```
Let's build your tone-of-voice doc. Quick intake:

1. Brand or person name?
2. Region/language? (Australian English / UK / US / other)
3. Business category? (optional)
4. Target audience in one sentence?
5. Short personal story or origin moment that captures the voice? (optional, can skip)
```

Wait for the response. Confirm region explicitly. If a `~/.claude/CLAUDE.md` exists with a stated regional preference, default to that. Otherwise ask.

### Step 2: Sample collection

Ask for all three samples in a single message. State the format requirements clearly. If the user provides fewer than three, or two of the same format, push back and ask for the missing format. Confirm receipt of each before moving on.

### Step 3: 11-dimension analysis

Analyse all three samples across the 11 dimensions below. Document each dimension **descriptively**. Never assign scores to dimensions during extraction. Scoring only appears in the final validation rubric (which scores future content against the doc, not the doc itself).

For each dimension, look for:

- What is **constant** across all three samples (= voice, the static fingerprint)
- What **shifts** between formats (= tone, the contextual variation)

Flag any inconsistencies that look like errors rather than intentional shifts.

**ALWAYS use the AskUserQuestion tool for clarification questions.** When you need to resolve an inconsistency, choose between interpretations, or pick a direction, fire `AskUserQuestion` with 2-4 concrete options. Do not rely on free-text questions in chat. Examples that MUST use the tool:

- One sample reads in a different register to the others (legacy voice vs current voice vs intentional shift)
- The user's CLAUDE.md preferences contradict a pattern in the samples (e.g. samples use em dashes but rule says no em dashes)
- Multiple plausible brand archetypes fit the samples
- The user mentions a constraint or audience that wasn't in the intake
- Any time a downstream content agent would need a definitive answer rather than a hedged one

Single-select with a `(Recommended)` first option is the default pattern. Use `multiSelect` only when the choices are genuinely additive.

### Step 4: Preferences via AskUserQuestion

After analysis (and after resolving any clarification questions from Step 3), lock in the five preferences below. Use **one** `AskUserQuestion` call with all five questions batched. The tool accepts 1-4 questions per call, so split into two calls if needed (questions 1-4 in one call, question 5 in another). These capture defaults the samples cannot reveal.

| # | Question | Header | Options |
|---|---|---|---|
| 1 | What target reading level should content default to? | Reading level | "3rd grade (recommended)" / "6th grade" / "8th grade" / "10th+ grade" / "Other (specify)" |
| 2 | What's your em dash policy? | Em dashes | "Never (recommended)" / "Sparingly" / "Freely" |
| 3 | What's your emoji policy? | Emojis | "Never" / "Functional only" / "Freely" / "Channel-dependent" |
| 4 | Which banned-word list should we apply? | Banned words | "Tier 1 list (recommended)" / "Tier 1 + 2" / "Custom list (provide)" / "None" |
| 5 | Any signature phrases, openers, or sign-offs to always include? | Signature phrases | "Yes (provide)" / "No" / "Will add later" |

**Always present 3rd grade as `(recommended)` for question 1.** Reasoning to share if asked: it converts better, it matches Hormozi-style direct copy, and it keeps content skim-friendly. Override matrix:

| Audience | Recommended grade |
|---|---|
| Mass consumer / social / sales | 3rd to 5th |
| SMB / general B2B | 6th to 8th |
| Enterprise / consultants | 8th to 10th |
| Technical / engineering | 10th to 12th |
| Legal / medical | 12th+ |

**Always present "Never" as `(recommended)` for em dashes.** This matches a strong personal preference in `~/.claude/CLAUDE.md` ("NO EM DASHES. EVER.") and is one of the strongest AI tells.

### Step 5: Generate the document

Read `references/tone-of-voice-template.md` first. Copy the entire 16-section structure. Populate each section with extracted data and the locked-in preferences.

**Embed real sample quotes** in the `Real Examples` section. Quoted anchor examples are the single highest-value section for downstream AI content generation. Pull at least one strong quote from each of the three samples.

If a section cannot be completed from the available data, mark it `[To be completed - need more samples]` rather than skipping it. All 16 sections must appear.

**Mandatory additions on top of the template** (required for every voice doc this skill generates):

1. **"How to Deploy This Voice" section** at the very top (after metadata, before Quick-Start Anchor). Tells the user to load BOTH this voice doc AND `references/ai-content-generation-guide.md` into any AI tool's context. Includes a table of how to load both into ChatGPT/Claude/Gemini/n8n/etc. The two files are a pair. The voice doc alone is not enough.

2. **"Quick-Start Anchor" section** (after Deploy section, before Purpose). 5-7 bullets of absolute non-negotiables for fast scanning: reading level, spelling, no-go words, signature ("mate who builds, not consultant who talks"), bold-in-emails rule, emoji rule, CTA rule.

3. **"3rd-Grade Discipline" subsection** in Sentence Structure. Concrete tactics, not vague targets. Includes:
   - Prefer monosyllabic words ("get" not "acquire", "build" not "construct", "fix" not "rectify")
   - The Breath Test (if you can't say a sentence in one breath, split it)
   - One idea per sentence
   - Cut every word that isn't load-bearing
   - Hemingway / Readable.com check before publishing
   - A "long word → short word" swap table (15-20 rows)

4. **"Engagement Cues" table** in Vocabulary section. List triggers like "Tip:", "Bonus:", "Here's how:", "Step-by-step:", "Quick win:", "Steal this:", "The truth:". These drive scroll, opens, and replies in social and blog content.

5. **"Relatability" subsection** in Voice Characteristics (only if it fits the brand). Look for human-side hooks the user has revealed in the intake or samples (e.g. parent-builder, recovering corporate, immigrant founder, ex-athlete, late bloomer). Document the hook as a strategic empathy tool, with when-to-lean-in and when-to-skip guidance. Skip this subsection entirely if no clear hook surfaces.

6. **"Supportive Brutal Truths" subsection** in Voice Characteristics. The pattern of: name the hard truth → show you're on their side → give them the move. Direct because you want the reader to win.

7. **Symbol-led punchlines** in Punctuation section. Encourage `=` (for results: "No trust = No sales"), `↓`, `→`, `✓`, `✗`. Symbols are scannable and don't read as AI.

8. **Bold and Formatting in Emails (CRITICAL) subsection** in Punctuation. Hard rule: never bold paragraph text or random words inside sentences (telltale AI sign). Bold IS allowed on standalone heading lines or section labels in emails. Includes a right/wrong example.

9. **Refined Emoji rules** in Punctuation. Hard rules: NEVER in client communications. Anywhere else: max 1 per output. Only when it earns its place. Prefer symbols over emojis. Use 🛑 only for genuine warnings, 🤯 only for genuine surprise. Auto-fail emoji list (decorative sparkles, corporate-cute prayer hands, LinkedIn-guru lightbulb).

When generating, deliberately avoid these AI-tells in the doc body itself: em dashes, bold mid-paragraph, "delve", "embark", "leverage", "unlock", "landscape", "ground-breaking", "ever-evolving".

### Step 6: Compliance scan

Before declaring done, verify the output against `references/ai-content-generation-guide.md`:

- [ ] All 16 template sections present
- [ ] Zero Tier 1 banned words used as **prose content** in the doc (delve, tapestry, embark, realm, leverage, navigate, unlock, landscape, foster, robust, synergy, beacon, treasure trove). They MAY appear in the "words to avoid" tables — that's their purpose. Run a context-aware grep, not a blanket grep.
- [ ] **Zero em dashes (— or –) anywhere in the doc body**, including tables, lists, callouts. Use full stops, commas, colons, parentheses, or split into two sentences. This is a hard rule when the user has set em-dash policy to "Never". Run `grep -nE '—|–' <doc>` and the result must be empty. If you accidentally write em dashes during generation, fix them all before reporting done.
- [ ] Regional language consistent throughout (no AU/UK/US mixing)
- [ ] At least one quote from each of the three samples in `Real Examples`
- [ ] Reading level, em dash, emoji, banned-words, and signature-phrase preferences captured in the relevant sections
- [ ] Voice scoring rubric included (6 criteria, 30 points, 24 minimum)

**Generation tip:** While writing the doc, deliberately avoid the em dash key. It's the single most common slip. Use ". " or ": " or split sentences. If you find yourself reaching for "X — Y", write it as "X. Y" or "X: Y" or "X, Y" instead.

Fix anything that fails. Report the location of the saved file and a one-line summary of the voice captured.

## The 11 voice dimensions

Document each descriptively. Look across all three samples for what's constant vs what shifts.

| # | Dimension | What to capture |
|---|---|---|
| 1 | **Vocabulary & regional language** | Regional variant (AU/UK/US), word choices, jargon level, complexity, distinctive lexicon, reading age |
| 2 | **Grammatical patterns** | Active vs passive, tense usage, pronouns (I/we/you), contraction rate |
| 3 | **Punctuation** | Em dash, ellipsis, exclamation, semicolon, ALL CAPS, parentheses. Frequency and rhythm. |
| 4 | **Sentence structure & length** | Average words per sentence, % short (<10) vs long (>30), variance, fragment usage |
| 5 | **Rhetorical devices** | Metaphors, similes, rhetorical questions, anecdotes, analogies, repetition |
| 6 | **Paragraph structure** | Length, topic sentences, transitions, white space, deductive vs inductive |
| 7 | **Tone & mood** | Formal/casual, serious/funny, respectful/irreverent, matter-of-fact/enthusiastic (NN/g 4-dim, 1-5 each, per context) |
| 8 | **Coherence & cohesion** | How ideas flow, transitional phrases, parallel structure, focus |
| 9 | **Brand personality** | 3-5 adjectives, archetype, who they'd be at a dinner party, what they'd never say |
| 10 | **Channel variations** | What shifts between social vs long-form vs corporate (= the tone matrix) |
| 11 | **Forbidden elements** | Words, tone, topics, competitor mentions, industry cliches to avoid |

**Critical:** dimensions are observations, not grades. Do not write "Score: 9/10" inside any dimension section.

## Reading level guidance

Default to 3rd grade reading level (Flesch-Kincaid 80+). This is more aggressive than typical recommendations but converts better and matches the no-fluff, direct copy style most modern brands aim for. The override matrix above tells you when to suggest higher.

Always state the chosen reading level in the `Quick Reference Card` and `Sentence Structure` sections of the output.

## Banned words

Reference: `references/ai-content-generation-guide.md`. The full Tier 1, 2, and 3 lists plus alternatives table live there.

When generating the voice doc, you must:

1. Scan the doc itself for Tier 1 banned words. Zero tolerance.
2. Embed the chosen banned-words list (Tier 1, Tier 1+2, custom, or none) into the `Vocabulary > Forbidden Phrases` section of the output.
3. Reference the guide path in the doc so downstream agents know where to find the full list.

## Output template

Reference: `references/tone-of-voice-template.md`. The 16 mandatory sections are:

1. Purpose
2. Regional Language & Culture
3. Voice Overview (personality, archetype, tone spectrum)
4. Voice Characteristics (sounds like / never sounds like)
5. Vocabulary (use / avoid / forbidden)
6. Grammatical Patterns
7. Punctuation
8. Sentence Structure (with metrics)
9. Rhetorical Devices
10. Paragraph Structure
11. Channel Variations
12. Real Examples (quoted from user's samples)
13. Voice Scoring Rubric (6 criteria, 30 points, 24 minimum)
14. Application Checklist
15. Quick Reference Card
16. Updates and Revisions

Every section must appear. Mark gaps as `[To be completed - need more samples]`.

## Output location

Default: `tone-of-voice.md` in the current working directory. If the user specifies a path, use that. Confirm the destination before writing.

## Anti-patterns

Don't:

- Run analysis on fewer than three formats. Push back and ask for the missing format.
- Score individual dimensions during extraction. Scoring only appears in the validation rubric.
- Skip the AskUserQuestion step. The five preferences are load-bearing.
- **Ask clarification questions as free text in chat.** Always use the `AskUserQuestion` tool with 2-4 concrete options when you need a decision from the user. Free-text prompts produce ambiguous answers and slow the workflow.
- Generate the doc from memory. Always read `references/tone-of-voice-template.md` first and follow its structure.
- Use abstract voice adjectives without concrete examples ("warm, authentic, bold" alone is meaningless to downstream AI). Always pair adjectives with quoted samples and do/don't pairs.
- Mix regional language variants in the output. AU spelling with US vocabulary fails validation.
- Use Tier 1 banned words in the voice doc itself. Scan before declaring done.
