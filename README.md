# voice-analyser

A Claude Code / Claude Cowork skill that builds a structured tone-of-voice document from 3 writing samples (1 social post, 1 article/blog, 1 corporate email). Output is a 16+ section markdown file ready to drop into any AI tool as voice context.

## Why this exists

Most "tone of voice" docs are abstract adjectives ("warm, authentic, bold") that downstream AI ignores. This skill produces something usable: real quotes, do/don't pairs, banned word lists, channel-specific tone matrices, and a scoring rubric. The output is designed to be loaded as context in ChatGPT, Claude, Gemini, n8n, or any AI workflow so the AI writes in the brand's voice without you re-prompting every time.

It refines the standard tone-of-voice agent pattern with three improvements:

1. **Hard rule: 3 formats required** (social, long-form, corporate). Single-format samples produce voice docs with major blind spots.
2. **Uses AskUserQuestion** to lock in 5 preferences after analysis (reading level, em dashes, emojis, banned words, signature phrases).
3. **Defaults reading level to 3rd grade** with explicit override matrix for technical/B2B/legal contexts.

## What you get

A `tone-of-voice.md` file (or whatever you name it) covering 20+ sections:

- 🛑 How to Deploy This Voice (load both files into your AI tool)
- ⚡ Quick-Start Anchor (7 non-negotiables)
- Regional language and cultural style
- Brand personality + tone matrix across 6 contexts
- Vocabulary (use, avoid, forbidden, engagement cues)
- Grammatical patterns + examples
- Punctuation rules (em dashes, emojis, bold-in-emails)
- Sentence structure + 3rd-grade discipline + word swap table
- Rhetorical devices
- Paragraph and channel structures
- 12 channel variations (social, email, blog, SMS, Slack, etc.)
- Real quoted examples from the user's own samples
- Voice scoring rubric (6 criteria, 30 points)
- Application checklist
- Quick reference card

## Installation

### Claude Code (CLI)

Install as a user-level skill:

```bash
git clone https://github.com/ai-orchestrators/voice-analyser.git ~/.claude/skills/voice-analyser
```

Or as a project-level skill:

```bash
git clone https://github.com/ai-orchestrators/voice-analyser.git .claude/skills/voice-analyser
```

The skill will appear in Claude Code's available skills list automatically.

### Claude Cowork (Desktop)

Cowork uses the same skill format as Claude Code. Drop the skill folder at `~/.claude/skills/voice-analyser/` and Cowork will pick it up.

### Manual install (any environment that supports Claude Code skills)

The skill is a standard Claude Code skill folder:

```
voice-analyser/
├── SKILL.md
├── README.md
└── references/
    ├── ai-content-generation-guide.md
    └── tone-of-voice-template.md
```

Drop the whole folder anywhere Claude Code reads skills from.

## Usage

### Trigger phrases

The skill auto-triggers on any of these:

- "voice analyser"
- "tone of voice"
- "TOV doc"
- "create a voice guide"
- "extract brand voice"
- "analyse my writing"
- "document my voice"
- "/voice-analyser"

Or any request that involves capturing how a person or brand writes.

### What happens

1. **Intake.** Skill asks for brand/person name, region, business category, target audience, optional origin story.
2. **Sample collection.** You paste 3 writing samples in 3 different formats. Skill pushes back if you only provide 1 or 2.
3. **Analysis.** Skill runs an 11-dimension analysis across all 3 samples, identifying what's constant (= voice) vs what shifts by format (= tone variations).
4. **Clarification.** If something is ambiguous (e.g. one sample reads in a different register), skill fires `AskUserQuestion` with concrete options.
5. **Preferences.** Skill fires `AskUserQuestion` with 5 batched questions: reading level, em dash policy, emoji policy, banned words, signature phrases.
6. **Generate.** Skill writes the 20+ section voice doc to your current working directory (or path you specify).
7. **Compliance scan.** Skill verifies zero em dashes, zero Tier 1 banned words in prose, regional spelling consistent, real samples quoted in `Real Examples`.

End-to-end takes about 5-10 minutes.

## Deploying the output

The voice doc only works if you load it AND the bundled `references/ai-content-generation-guide.md` into your AI tool's context. Both files together. The voice doc has the rules and patterns; the content generation guide has the full 250+ word banned-words table.

### How to load both into common tools

| AI tool | How to load |
|---|---|
| ChatGPT / GPTs | Paste both into the GPT's "Knowledge" section or system prompt |
| Claude (web/desktop) | Add both as Project files, or paste at the start of the conversation |
| Claude Code | Reference both file paths in the session, or load via a CLAUDE.md / skill |
| Gemini | Paste both into system instructions or project context |
| n8n / Make / Zapier AI nodes | Include both as context in the prompt template |
| Custom API calls | Pass both as system messages or RAG context |

## What's bundled

### `references/ai-content-generation-guide.md`

The bundled AI-content generation guide. ~650 lines covering:

- Tier 1 banned words (the strongest AI tells: delve, tapestry, embark, realm, leverage, navigate, unlock, landscape, foster, robust, synergy, beacon, treasure trove)
- Tier 2 banned words (transition words, intensifiers, adjectives, formulaic phrases)
- Tier 3 contextual bans (combinations like "delving into the intricacies of")
- 250+ word alternatives table
- 2025/2026 AI-detection updates (AI Slop, slopper, glazing, etc.)

### `references/tone-of-voice-template.md`

The 16-section template the skill uses as the structural blueprint for every voice doc it generates.

## Optional: run skill-creator evals

The skill-creator workflow (test cases, eval viewer, benchmark runs) is not bundled. If you want to run quantitative iteration on this skill:

```bash
# In a Claude Code session with the skill-creator plugin installed
claude
> /skill-creator
> Improve the voice-analyser skill
```

## Credits

- Built on the standard `tone-of-voice-agent` and `ai-content-generation-guide` patterns
- Voice analysis dimensions adapted from Nielsen Norman Group's 4-dimension framework
- Banned word research from OpenAI Community, Scribbr, ContentBeta, Macquarie Dictionary 2025

## Licence

MIT. Fork it, modify it, ship it.
