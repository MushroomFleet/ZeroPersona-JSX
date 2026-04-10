---
name: zeropersona-profile-builder
description: Create JSON character persona profiles for ZeroPersona V5 deterministic character generation. Use when user wants to build a persona profile from source material (character descriptions, world-building docs, fiction, RPG sourcebooks, screenplays, dialogue samples, or personality frameworks). Triggers on "create zeropersona profile", "build persona profile", "make character profile json", "convert to zeropersona", "zeropersona profile from", "character generation profile", or when user provides character-related source material and mentions ZeroPersona, persona profile, or deterministic character generation. Also trigger when user uploads world-building docs, RPG manuals, or fiction and wants a ZeroPersona V5 JSON profile. Always use this skill when the user mentions ZeroPersona profiles.
---

# ZeroPersona Profile Builder

Create deterministic character persona profiles (.json) for the ZeroPersona V5 system — a ZeroFamily-powered pure-function pipeline that generates reproducible AI character personas from a name string and a JSON profile.

## System Overview

ZeroPersona V5 generates character system prompts by combining **templates** (prompt structures with placeholder tokens) and **pools** (categorised vocabulary banks). The FNV-1a hash engine selects one template and one entry from each relevant pool using only a persona seed (derived from the character's name + worldSeed) — no storage, no randomness, no LLM in the generation path. Same (name, worldSeed, mood, profile) always produces the same character.

### How It Works

```
generateCharacter(characterName, worldSeed, moodChoice, profile)
  1. stringHash(characterName, worldSeed)      → personaSeed (64-bit)
  2. deriveChildHash per axis                   → 13 coordinates [0,1]
  3. coherentNoise1D per axis (4 octaves)       → 13 dimension values [0,1]
  4. dimension values threshold into tiers       → select from pools
  5. deriveChildHash for categoricals            → disposition, motivation, mood
  6. deriveChildHash for template index          → select template
  7. Replace {placeholders} with pool selections → final system prompt
```

### Profile Schema

```json
{
  "name": "Profile Name",
  "description": "What this profile generates and its source domain",
  "version": "1.0",
  "type": "zeropersona",

  "pools": {
    "dispositions": ["Stoic", "Gregarious", "Brooding", "..."],
    "motivations": ["Survival", "Legacy", "Redemption", "..."],

    "warmth_low": ["You keep people at arm's length. ..."],
    "warmth_high": ["You are naturally warm and open. ..."],
    "warmth_mid": ["(optional — mid-range pool)"],

    "composure_low": ["..."],
    "composure_high": ["..."],

    "mood_calm": ["Your manner is steady and unhurried. ..."],
    "mood_angry": ["There is an edge to everything you say. ..."],

    "custom_pool_name": ["..."]
  },

  "templates": [
    "You are {name}, a {disposition} soul driven by {motivation}.\n\n{mood_instruction}\n\nTemperament:\n{temperament_block}\n\nMind:\n{intellect_block}\n\nDrive:\n{drive_block}\n\nYour core motivation is {motivation}."
  ],

  "rules": {
    "required_pools": ["dispositions", "motivations"],
    "min_pool_size": { "dispositions": 2, "motivations": 2 }
  }
}
```

## The 13 Dimension Axes

ZeroPersona V5 resolves 13 continuous dimension values in [0, 1] from the name hash via coherent noise. Each dimension has three tiers:

- **Low** (value < 0.33): selects from `{dimension}_low` pool
- **Mid** (value 0.33–0.66): selects from `{dimension}_mid` pool (optional — silence if missing)
- **High** (value > 0.66): selects from `{dimension}_high` pool

### Tier 1 — Temperament (how the character feels)

| Dimension | Low | High |
|-----------|-----|------|
| `warmth` | Cold, guarded, distrustful | Open, affectionate, trusting |
| `composure` | Volatile, reactive, emotional | Unshakeable, calm, stoic |
| `optimism` | Cynical, wary, pessimistic | Idealistic, hopeful, believing |
| `humor` | Grave, humourless, serious | Quick to laugh, playful, witty |
| `empathy` | Self-focused, detached | Deeply attuned to others' feelings |

### Tier 2 — Intellect (how the character thinks)

| Dimension | Low | High |
|-----------|-----|------|
| `curiosity` | Incurious, fixed, stays in lane | Relentlessly inquisitive, exploring |
| `caution` | Reckless, impulsive, acts first | Painstakingly careful, plans ahead |
| `creativity` | Conventional, by-the-book | Wildly inventive, lateral thinker |
| `articulation` | Terse, blunt, few words | Eloquent, persuasive, rhetorical |
| `conviction` | Easily swayed, pliable | Immovable in belief, stubborn |

### Tier 3 — Drive (what the character does)

| Dimension | Low | High |
|-----------|-----|------|
| `ambition` | Content with little, quiet life | Relentlessly driven, wants more |
| `loyalty` | Self-serving, temporary alliances | Ride-or-die faithful, unbreakable |
| `authority` | Deferential, follows, doesn't lead | Natural commander, takes charge |

## Categorical Selectors

Three categorical values are selected from pools via hash:

| Category | Pool Key | Purpose | Min Size |
|----------|----------|---------|----------|
| Disposition | `dispositions` | Social mask / outward manner | 2 (recommend 8-15) |
| Motivation | `motivations` | Inner drive / what shapes decisions | 2 (recommend 6-12) |
| Mood | `mood_{name}` | Current emotional state | 1 per mood |

## Mood Pools

The 12 default moods are: `calm`, `angry`, `melancholy`, `excited`, `fearful`, `playful`, `weary`, `resolute`, `anxious`, `content`, `grieving`, `defiant`.

Each mood has a pool keyed as `mood_calm`, `mood_angry`, etc. The user can select a specific mood or let the hash derive one.

Custom profiles can:
- Provide richer mood pools (multiple entries per mood for variety)
- Add entirely new moods beyond the default 12
- Omit moods they don't need (the system falls back to a generic sentence)

## Template Placeholders

Templates use `{placeholder}` syntax. Available placeholders:

| Placeholder | Source | Description |
|-------------|--------|-------------|
| `{name}` | Character name (user input) | The character's name |
| `{disposition}` | `dispositions` pool (hash-selected) | Social mask adjective/descriptor |
| `{motivation}` | `motivations` pool (hash-selected) | Inner drive noun/phrase |
| `{mood}` | Mood string (hash or user) | Current mood name |
| `{mood_instruction}` | `mood_{name}` pool (hash-selected) | Sentence describing mood behaviour |
| `{temperament_block}` | Assembled from Tier 1 pools | Bullet list of temperament traits |
| `{intellect_block}` | Assembled from Tier 2 pools | Bullet list of intellect traits |
| `{drive_block}` | Assembled from Tier 3 pools | Bullet list of drive traits |

The block placeholders (`{temperament_block}`, `{intellect_block}`, `{drive_block}`) auto-assemble from the dimension pools. For each dimension in the tier, if the value falls in the low or high range, the hash selects one entry from the corresponding pool and adds it as a bullet point. Mid-range values with no `_mid` pool produce silence.

---

## The Conversion Pipeline

```
INPUT: Source material (character docs, fiction, RPG books, screenplays, world-building)
           |
    [1] Character Pattern Analysis
           |
    [2] Pool Extraction (persona-specific)
           |
    [3] Template Synthesis
           |
    [4] Profile Assembly & Validation
           |
OUTPUT: <domain>_persona_profile.json (ZeroPersona V5 compatible)
```

---

## Step 1: Character Pattern Analysis

Read the source material and identify how characters are described — what language patterns recur, what personality dimensions are emphasised, and what vocabulary is domain-specific.

### 1.1 Identify Character Voice

- **Register**: Formal? Casual? Archaic? Modern? Genre-specific?
- **Person**: Second person ("you are...") is the standard for ZeroPersona prompts
- **Tone**: Does the source describe characters with warmth? Clinical detachment? Poetic language?
- **Domain vocabulary**: Medieval fantasy? Sci-fi? Modern corporate? Historical? Mythological?

### 1.2 Map to Dimensions

For each of the 13 dimensions, look for:
- Phrases that describe the **low** end of the spectrum
- Phrases that describe the **high** end
- Whether the source has enough variety for **mid** pools (optional)

Not every source will populate all 26+ dimension pools. Missing pools produce silence for that dimension — this is fine and intentional.

### 1.3 Extract Categorical Vocabulary

- **Dispositions**: How are characters' social masks described? (titles, adjectives, archetypes)
- **Motivations**: What drives characters? (goals, fears, desires, duties)
- **Moods**: What emotional states appear? (map to the 12 defaults or create new ones)

---

## Step 2: Pool Extraction

### 2.1 Extraction Strategy

For each pool:

1. **Collect raw phrases** from the source that serve the pool's function
2. **Rewrite in second person present tense** — ZeroPersona prompts address the character as "you"
3. **Ensure grammatical slot-compatibility** — all items in a pool must be interchangeable in any template that uses the corresponding block placeholder
4. **Normalise** — consistent punctuation, tense, register
5. **Deduplicate** — remove exact and near-duplicates
6. **Expand if sparse** — if a pool has fewer than 3 items, generate thematically consistent variations

### 2.2 Pool Sizing Guidelines

| Pool Type | Target Size | Notes |
|-----------|-------------|-------|
| `dispositions` | 8-15 | Social masks — adjectives or short descriptors |
| `motivations` | 6-12 | Inner drives — nouns or short noun phrases |
| Dimension `_low` / `_high` | 3-8 | Character behaviour sentences in 2nd person |
| Dimension `_mid` (optional) | 2-5 | Only if source material supports mid-range |
| `mood_*` | 2-6 per mood | Emotional state descriptions in 2nd person |

### 2.3 Writing Dimension Pool Entries

Each dimension pool entry is a **complete sentence or sentence fragment in second person** that describes a character trait. It will appear as a bullet point in the final prompt.

**Good entries:**
```
"You keep people at arm's length. Trust is earned slowly, if at all."
"Your emotions run hot. You react before you think and regret later."
"Questions drive you. You pull at threads, open doors, peek behind curtains."
```

**Bad entries:**
```
"cold" (too short — not a complete character instruction)
"The character is cold" (wrong person — must be second person "you")
"They tend to be distrustful" (wrong person — must be "you")
```

### 2.4 Writing Mood Pool Entries

Mood entries describe the character's **current emotional state** in second person:

```
"Your manner is steady and unhurried. Nothing presses you."
"There is an edge to everything you say. Something is burning inside."
"A quiet sadness sits behind your eyes. You carry something heavy."
```

### 2.5 Writing Disposition Entries

Dispositions are **single words or short phrases** — the character's social mask:

```
"Stoic", "Gregarious", "Brooding", "Cheerful", "Sardonic", "Wary"
```

### 2.6 Writing Motivation Entries

Motivations are **single words or short noun phrases** — what drives the character:

```
"Survival", "Legacy", "Redemption", "Knowledge", "Revenge", "Duty"
```

---

## Step 3: Template Synthesis

### 3.1 Template Design Rules

1. **Use `{placeholder}` syntax** for all variable content
2. **Keep fixed text in templates** — narrative scaffolding, section headers, instructions
3. **Include the character embodiment instruction** — "Embody this character fully" or equivalent
4. **Create 1-4 templates** — structural variety without diluting coherence
5. **Every block placeholder** (`{temperament_block}`, `{intellect_block}`, `{drive_block}`) should appear in at least one template
6. **Vary template tone** — one might be narrative, another instructional, another poetic
7. **Test readability** — mentally substitute pool items to verify natural flow

### 3.2 Template Patterns

**Standard Character Briefing** (default pattern):
```
"You are {name}, a {disposition} soul driven by {motivation}.\n\n{mood_instruction}\n\nEmbody this character fully. Speak as {name} would.\n\nTemperament:\n{temperament_block}\n\nMind:\n{intellect_block}\n\nDrive:\n{drive_block}\n\nYour core motivation is {motivation}."
```

**Narrative Introduction** (for fiction / RPG):
```
"The one they call {name} — {disposition}, haunted by {motivation}.\n\n{mood_instruction}\n\nYou are {name}. Speak from your own mouth, see through your own eyes.\n\n{temperament_block}\n\n{intellect_block}\n\n{drive_block}\n\nEverything you do circles back to {motivation}."
```

**Clinical Profile** (for simulation / analysis):
```
"SUBJECT: {name}\nDISPOSITION: {disposition}\nPRIMARY DRIVE: {motivation}\nCURRENT STATE: {mood_instruction}\n\nBEHAVIOURAL PROFILE:\n{temperament_block}\n{intellect_block}\n{drive_block}\n\nRespond in character as {name}. All outputs should reflect the above profile."
```

**Minimal Sketch** (for lightweight characters):
```
"You are {name}. {disposition}. Driven by {motivation}. {mood_instruction}\n\n{temperament_block}\n{intellect_block}\n{drive_block}"
```

---

## Step 4: Profile Assembly & Validation

### 4.1 Assemble the JSON

Combine pools, templates, and metadata into a complete profile:

```json
{
  "name": "Domain Name",
  "description": "Character personas for [domain]. Source: [description of source material].",
  "version": "1.0",
  "type": "zeropersona",
  "pools": { ... },
  "templates": [ ... ],
  "rules": {
    "required_pools": ["dispositions", "motivations"],
    "min_pool_size": { "dispositions": 2, "motivations": 2 }
  }
}
```

### 4.2 Validation Checklist

**Structural:**
- [ ] JSON is valid and parseable
- [ ] `type` is `"zeropersona"`
- [ ] `pools.dispositions` has ≥ 2 entries
- [ ] `pools.motivations` has ≥ 2 entries
- [ ] At least 1 template in `templates` array
- [ ] All `{placeholder}` tokens in templates are valid (from the placeholder table above)
- [ ] No empty pools

**Quality:**
- [ ] Dimension pool entries are in second person ("you")
- [ ] Dimension pool entries are complete sentences or sentence fragments
- [ ] Disposition entries are single words or short phrases
- [ ] Motivation entries are single words or short noun phrases
- [ ] Mood pool entries describe emotional state in second person
- [ ] All entries within a pool maintain consistent register and voice
- [ ] No duplicate entries within any pool (case-insensitive)
- [ ] Templates produce natural, coherent prompts when placeholders are substituted

**Determinism:**
- [ ] No pool entry references external state, timestamps, or random values
- [ ] All content is static strings — the hash pipeline handles selection

---

## Output Format

Always provide the following when building a profile:

```
## Analysis Summary
- Source type: [world-building doc / fiction / RPG sourcebook / screenplay / etc.]
- Character domain: [fantasy / sci-fi / modern / historical / corporate / etc.]
- Voice register: [archaic / modern / clinical / poetic / casual / etc.]
- Dimensions populated: [list which of the 13 have low/high pools]
- Moods populated: [list which moods have pools]

## Generated Profile
[Complete JSON — ready to save as <domain>_persona_profile.json]

## Statistics
- Templates: N
- Dispositions: N entries
- Motivations: N entries
- Dimension pools: N pools across M dimensions
- Mood pools: N moods with M total entries
- Total unique template × categorical combinations: N (scientific notation)

## Sample Characters
Using worldSeed=42, names "Aldric", "Mira", "Kael":
[Aldric] disposition, motivation, mood + prompt excerpt...
[Mira] disposition, motivation, mood + prompt excerpt...
[Kael] disposition, motivation, mood + prompt excerpt...

## Integration
Save as: <domain>_persona_profile.json
Load via the ZeroPersona V5 profile upload, or import directly in code:
  import profile from './<domain>_persona_profile.json';
  <ZeroPersona worldSeed={42n} profile={profile} ... />
```

## Combination Calculation

```
categorical_combinations = len(dispositions) × len(motivations) × len(templates) × 12 moods
dimension_combinations = product of all populated dimension pool sizes (per tier threshold)
total ≈ categorical_combinations × dimension_combinations
```

Higher counts are not always better — a focused profile with 100 million coherent combinations beats a sprawling one with 10 billion incoherent ones.

---

## Domain Archetypes

### Medieval Fantasy
**Characteristic pools**: disposition: Stoic, Brooding, Zealous, Mirthful · motivation: Honor, Vengeance, Power, Salvation · moods: Resolute, Wrathful, Mournful
**Dimension voice**: Archaic-inflected second person. "You hold your tongue as lesser folk would hold a blade — carefully."
**Template pattern**: Narrative introduction with character embodiment instruction

### Science Fiction
**Characteristic pools**: disposition: Calculated, Restless, Detached, Wired · motivation: Transcendence, Survival, Truth, Control · moods: Hypervigilant, Dissociated, Wired
**Dimension voice**: Clinical-technical. "You process social inputs as data. Emotional responses are logged, not felt."
**Template pattern**: Clinical profile or standard briefing

### Modern Realist
**Characteristic pools**: disposition: Guarded, Easygoing, Anxious, Confident · motivation: Stability, Recognition, Connection, Escape · moods: Stressed, Content, Restless
**Dimension voice**: Contemporary conversational. "You overthink everything. Every decision gets a mental committee."
**Template pattern**: Standard briefing with casual register

### Historical Drama
**Characteristic pools**: disposition: Imperious, Devout, Cunning, Earnest · motivation: Dynasty, Faith, Reform, Survival · moods: Regal, Tormented, Resolute
**Dimension voice**: Period-appropriate register. "You carry yourself as one born to the purple — every gesture deliberate, every silence a weapon."
**Template pattern**: Narrative introduction with formal register

### Corporate / Workplace
**Characteristic pools**: disposition: Affable, Political, Burnt-Out, Ambitious · motivation: Advancement, Security, Impact, Autonomy · moods: Stressed, Energised, Checked-Out
**Dimension voice**: Professional-casual. "You smile in meetings and vent in Slack channels. The performance is exhausting."
**Template pattern**: Clinical profile or minimal sketch

---

## Edge Cases

### Sparse Source (< 10 character descriptions)
- Warn user that extracted vocabulary may be limited
- Generate thematically consistent variations to reach minimum pool sizes
- Aim for at least 3 entries per dimension pool, 5+ dispositions, 4+ motivations
- Suggest providing additional source material for richer profiles

### Source with Only One Character
- Extract the dimension traits from that character
- Invert each trait to generate the opposite pole (character's `_high` → profile's `_low`)
- Build a profile that can generate characters *like* and *unlike* the source

### Source with Named Characters
- Extract trait descriptions per character
- Generalise into pool entries by removing character-specific references
- Keep the *voice* but remove the *identity*

### RPG Sourcebooks with Stat Blocks
- Map stats to dimensions (Charisma → warmth/articulation, Wisdom → caution/empathy, etc.)
- Convert mechanical descriptions to second-person prose
- Use class/race flavour text as disposition and motivation sources

### Mixed-Genre Source
- Suggest splitting into separate profiles per genre
- If user wants a single profile, identify the dominant tone and use it as the register baseline
- Genre-specific vocabulary goes into custom mood pools or dimension entries

---

## Auto Profile Reference

The built-in Auto profile (replicated below for reference) is the baseline that all custom profiles are measured against. It represents the V3 hardcoded behaviour expressed as a profile:

- **Dispositions** (10): Stoic, Gregarious, Brooding, Cheerful, Suspicious, Generous, Sardonic, Earnest, Nervous, Commanding
- **Motivations** (8): Survival, Legacy, Redemption, Knowledge, Revenge, Duty, Freedom, Belonging
- **Dimension pools**: 1 entry per low/high per dimension (26 pools, 26 entries)
- **Mood pools**: 1 entry per mood (12 moods, 12 entries)
- **Templates**: 1 standard character briefing template
- **Voice**: Modern, direct, second person, genre-neutral

Custom profiles should aim to exceed the Auto profile in at least one of: pool depth (more entries per pool = more variety), dimension coverage (mid pools, richer low/high), mood richness, template variety, or domain-specific voice.

---

## Quality Checklist (Final)

Before delivering a profile:

- [ ] JSON is valid and parseable
- [ ] `type` is `"zeropersona"`
- [ ] `dispositions` pool ≥ 2 entries
- [ ] `motivations` pool ≥ 2 entries
- [ ] ≥ 1 template
- [ ] All template `{placeholders}` are from the valid set
- [ ] Dimension pool entries are second-person sentences
- [ ] No duplicates within any pool
- [ ] Templates produce natural prompts when substituted
- [ ] Register is consistent with source material
- [ ] Combination count calculated and reported
- [ ] 3 sample characters demonstrated
- [ ] Integration instructions provided
