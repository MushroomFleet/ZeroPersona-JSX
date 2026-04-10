# ZeroPersona-JSX

Deterministic, multi-dimensional AI persona generation from a single string. Part of the [ZeroFamily](https://github.com/MushroomFleet) methodology.

ZeroPersona is a pure-function pipeline that converts a user-entered string into a rich, coherent AI persona — same string, same persona, any machine, any session, forever. No randomness. No storage. No API calls. Just a name in, a system prompt out.

The core pipeline: **String → FNV-1a 64-bit hash → 13-axis coherent noise field → natural-language system prompt**. The hash produces coordinates in a continuous persona space. 4-octave coherent noise ensures nearby strings produce related personas. Dimension values are thresholded into instruction fragments and assembled via template substitution.

## How It Works

```
"Aldric"  →  normalize("aldric")
          →  fnv64(bytes + worldSeed)     →  0xa7f3e2b1c9d04e8f
          →  deriveChildHash per axis      →  13 coordinates [0,1]
          →  coherentNoise1D per axis      →  13 dimension values [0,1]
          →  threshold + pool selection    →  instruction fragments
          →  template assembly             →  complete system prompt
```

The same `(name, worldSeed, profile)` tuple always produces the identical system prompt. The `worldSeed` parameter allows different installations to maintain independent persona spaces from the same input strings.

## Standalone JSX Component

ZeroPersona is designed as a self-contained React component droppable into any project:

```jsx
import { ZeroPersona } from './zero-persona';

function MyApp() {
  const [persona, setPersona] = useState(null);

  return (
    <ZeroPersona
      worldSeed={42n}
      onPersonaGenerated={(p) => {
        setPersona(p);
        // Use p.systemPrompt with any AI API
      }}
      onBookmarkSave={(bookmark) => {
        // Persist to your storage
      }}
    />
  );
}
```

The component accepts `worldSeed`, an optional `profile` (JSON vocabulary override), and `bookmarks` as props. It emits a `Persona` object containing the generated system prompt, all dimension values, categorical selectors, and a coordinate vector. The host application decides what to do with the output — ZeroPersona produces a string, not an API call.

All hash and noise functions are internal pure functions. Zero external dependencies beyond React. No API calls. No storage. No side effects.

## Demo Versions

The repository includes interactive HTML demos that showcase the system's evolution from prototype to profile-driven character generator. Each demo is a single self-contained HTML file with the full pipeline running live in the browser.

### V1 — Prototype

The original proof of concept. Type any string, get a 13-dimensional AI assistant persona with radar chart visualization, dimension bars, pipeline trace, and a generated system prompt. Demonstrates the core insight: string → hash → coherent noise → persona.

Dimensions are organized as Communication (verbosity, formality, directness, warmth, humor), Cognitive (abstraction, certainty, creativity, depth, pedagogy), and Identity (domain affinity, temporal frame, agency). Categorical selectors: domain archetype, voice register, reasoning style.

### V2 — Domain Selection

**Live demo: [scuffedepoch.com/zero-persona/](https://scuffedepoch.com/zero-persona/)**

Refines the input model: the string is now a **persona name** — an anchor point in persona space, not a description. Removes example pills. Adds a **domain dropdown** allowing the user to pin expertise (Engineering, Humanities, Sciences, etc.) while the hash drives everything else. Domain defaults to hash-derived ("Auto") preserving V1 behaviour.

Bookmarks store the domain override, enabling domain-specialised personas from a single name seed.

### V3 — NPC Character Generation

Reframes the entire system for **world-building**. Domains are replaced by **20 Occupations** (Blacksmith, Healer, Scholar, Merchant, Soldier, Farmer, Hunter, Priest, Thief, Sailor, Alchemist, Carpenter, Scribe, Cook, Miner, Weaver, Stable Hand, Diplomat, Entertainer, Gravedigger). Dimensions are rewritten as character personality traits across three tiers:

- **Temperament** — Warmth, Composure, Optimism, Humor, Empathy
- **Intellect** — Curiosity, Caution, Creativity, Articulation, Conviction
- **Drive** — Ambition, Loyalty, Authority

Categorical selectors become **Disposition** (social mask) and **Motivation** (inner drive). Prompt assembly writes in second-person character voice. Bookmarks become a **Roster** of up to 10 named characters.

### V5 — JSON Profile System

**Live demo: [scuffedepoch.com/zero-mood/](https://scuffedepoch.com/zero-mood/)**

Introduces the **JSON Profile system** — externalizing all vocabulary pools and prompt templates into loadable profile files. The hash pipeline is unchanged; a profile controls what the coordinates resolve into.

Occupation is replaced by **Mood** — 12 emotional states (Calm, Angry, Melancholy, Excited, Fearful, Playful, Weary, Resolute, Anxious, Content, Grieving, Defiant) as the user-selectable dimension. "All Moods" defaults to hash-derived selection.

A profile is a JSON object with `pools` (named string arrays keyed by `dimension_tier` and `mood_name`) and `templates` (prompt strings with `{placeholder}` substitution). The built-in "Auto" profile reproduces V3 behaviour exactly. Custom profiles can reshape the output entirely.

The determinism contract extends to: **same name + same seed + same mood + same profile = same output**.

Features: profile upload/validation, profile switching via dropdown, profile JSON inspector tab, and bookmarks that store mood override and profile reference.

## JSON Profile Builder Skill

The repository includes `zeropersona-profile-builder.skill` — a Claude skill for creating custom ZeroPersona V5 JSON profiles from source material. Feed it character descriptions, world-building documents, fiction excerpts, RPG sourcebooks, or any text describing character archetypes and the skill will:

1. Analyse the source material for character patterns and voice register
2. Extract vocabulary into dimension pools (13 axes × low/high/mid tiers)
3. Build disposition, motivation, and mood pools from the source
4. Synthesise templates matching the source domain's voice
5. Assemble and validate a complete ZeroPersona V5 profile

The skill follows the established ZeroFamily profile builder pattern (zeroenh, zeroprompt, zeroresponse, zeroedit) and outputs analysis summary, complete JSON, statistics, sample characters, and integration instructions.

## Example Profile: Sharona

The repository includes `sharona_persona_profile.json` — a complete ZeroPersona V5 profile built from the character bible for *Sharona: Huntress of the Forgotten Earth*. It demonstrates the profile system with a rich, domain-specific character:

- **12 dispositions**: Silent, Watchful, Feral, Wary, Fierce, Solitary, Restless, Still, Guarded, Wild, Patient, Coiled
- **8 motivations**: Survival, The next kill, Keeping moving, Protecting her companion, Following the herds south, Finding shelter before dark, Understanding the stars, Staying free
- **26 dimension pools** (61 entries) — all 13 dimensions with low/high in primal second-person voice
- **12 mood pools** — each written specifically for Sharona's world
- **3 templates** — primal briefing, narrative introduction, minimal imperative

Load the profile in the V5 demo, type any character name, and the pipeline generates characters that inhabit Sharona's world with her voice register but their own unique personality coordinates.

## Lineage

ZeroPersona descends from [ZeroSpecialist/ZeroSwarm](https://github.com/MushroomFleet) — a stigmergic AI specialist coordination system where 128-bit seed tuples generate deterministic agent personas through FNV-1a hashing and coherent noise fields. ZeroPersona strips away the swarm machinery (no epochs, no affinity, no signals, no evolutionary selection) and keeps the proven core: hash → noise → deterministic output. It points that core at the user instead of an automated orchestrator.

The `hash.ts` code is reused verbatim from ZeroSwarm. The noise field is adapted from 2D to 1D-per-axis. The prompt assembler and profile system are new. The mathematical foundation is identical.

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{zeropersona_jsx,
  title = {ZeroPersona-JSX: Deterministic Multi-Dimensional AI Persona Generation},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/ZeroPersona-JSX},
  version = {1.0.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)
