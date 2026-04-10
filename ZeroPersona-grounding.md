# ZeroPersona — Grounding Document

## Origin and Lineage

ZeroPersona emerges from the ZeroFamily methodology, specifically from the proven ZeroSpecialist/ZeroSwarm stigmergic pipeline. That system demonstrated that a 128-bit seed tuple `(roleId, styleId, capabilitySalt, worldSeed)` hashed through FNV-1a and fed into coherent noise can produce a fully deterministic AI specialist persona — reconstructable on any machine, in any session, forever — without storing prompt text.

ZeroSpecialist solved a *swarm* problem: generating, benchmarking, and rotating populations of AI agents through evolutionary selection. It treats the specialist roster as an infinite continuous field sampled by an automated pipeline. The user never touches coordinates directly. The swarm does.

**ZeroPersona solves a different problem entirely.**

ZeroPersona is a *user-facing persona construction system*. The user's own natural-language input becomes the hash seed. The resulting coordinates define a multi-dimensional persona that shapes the AI's behavior, tone, expertise, and communication style. There is no swarm. There is no evolutionary selection. There is no stigmergic signal field. There is a human being typing a string, and that string deterministically resolving into a rich, coherent persona.

---

## What ZeroPersona Is

A pure-function pipeline that converts an arbitrary user string into a multi-dimensional persona definition used as (or injected into) an AI system prompt. The persona is not randomly generated — it is *navigated*. The same string always produces the same persona. Similar strings produce related personas. The user explores persona-space by varying their input.

### The Core Insight

In ZeroSpecialist, the seed tuple is four integers. In ZeroPersona, the seed is a **user-entered string**. The string is hashed to produce coordinates in a continuous multi-dimensional persona space. Those coordinates are fed through the same coherent noise field architecture (proven in ZeroSwarm) to produce behavioral dimensions. Those dimensions assemble into a natural-language system prompt.

The critical difference: the user *is* the seed source. This transforms the ZeroFamily hash pipeline from an infrastructure component into an interactive creative tool.

---

## What ZeroPersona Is Not

- **Not a swarm system.** There is no candidate pool, no benchmarking, no rotation, no epoch tracking, no stigmergic signals. Those belong to ZeroSpecialist.
- **Not a prompt template selector.** It does not pick from a finite list of personas. It generates from a continuous field.
- **Not a chatbot personality quiz.** The user doesn't answer questions. They enter a string. The string *is* the coordinate.
- **Not dependent on OpenRouter or any specific API.** ZeroPersona produces a system prompt string. What consumes that string is the host application's concern.

---

## Architecture: String → Hash → Coordinates → Dimensions → Persona

### Stage 1 — String to Hash

The user enters any string. Examples:

- `"battle-hardened sysadmin who speaks in haiku"`
- `"gentle maths tutor for a 12-year-old"`  
- `"unhinged inventor with a grudge against XML"`
- `"corporate compliance officer, secretly a poet"`

The string is normalized (lowercase, trimmed, whitespace-collapsed) and hashed via FNV-1a 64-bit to produce a single `bigint`. This is the **persona seed**.

```
userString → normalize() → fnv64(utf8Bytes) → personaSeed: bigint
```

The persona seed is deterministic and portable. The same string on any machine, any session, any platform produces the same seed.

### Stage 2 — Hash to Coordinates

The 64-bit persona seed is decomposed into N independent coordinate channels using bit extraction and child-hash derivation (identical technique to ZeroSpecialist's `hashToFloatChannel` and `deriveChildHash`).

Each coordinate maps to one axis of persona-space. The coordinate system is not 2D like ZeroSpecialist's `(roleId, styleId)` — it is **N-dimensional**, where N is the number of persona trait axes.

```
personaSeed → deriveChildHash(seed, axisIndex) → hashToFloat() → coordinate[axisIndex]
```

Each coordinate is a float in [0, 1], representing a position along one persona trait axis.

### Stage 3 — Coordinates to Dimensions (Coherent Noise Field)

Each coordinate is fed through the same octave-layered coherent noise architecture from ZeroSwarm's capability field. This ensures:

- **Local coherence:** Similar strings produce similar personas (because nearby coordinates in the noise field produce correlated values).
- **Global variation:** Distant strings produce unrelated personas.
- **No discontinuities:** The persona space is smooth and explorable.

The noise field maps raw coordinates to **behavioral dimension values** — each a float in [0, 1].

### Stage 4 — Dimensions to Persona Prompt

The behavioral dimension values are thresholded and mapped to natural-language instruction fragments, then assembled into a complete system prompt. This is the same proven assembly technique from ZeroSwarm's `prompt-generator.ts`, adapted for the richer persona dimension set.

---

## The Persona Dimension Space

ZeroSpecialist uses 5 behavioral dimensions (verbosity, assertiveness, formality, domainFocus, reasoningDepth) plus 3 categorical selectors (role archetype, style modifier, reasoning approach). This was sufficient for swarm agent differentiation.

ZeroPersona expands to a richer multi-dimensional space designed for human-facing persona construction. The dimensions are organized into three tiers:

### Tier 1 — Communication Axes (how the persona speaks)

| Axis | Low (0.0) | Mid (0.5) | High (1.0) |
|------|-----------|-----------|------------|
| **Verbosity** | Terse, telegraphic | Balanced, clear | Expansive, thorough |
| **Formality** | Casual, colloquial | Professional neutral | Academic, ceremonial |
| **Directness** | Socratic, circuitous | Guided, structured | Blunt, no preamble |
| **Warmth** | Clinical, detached | Friendly, approachable | Effusive, encouraging |
| **Humor** | Dead serious | Dry wit, occasional | Playful, irreverent |

### Tier 2 — Cognitive Axes (how the persona thinks)

| Axis | Low (0.0) | Mid (0.5) | High (1.0) |
|------|-----------|-----------|------------|
| **Abstraction** | Concrete, example-first | Mixed | Theoretical, principle-first |
| **Certainty** | Hedging, probabilistic | Balanced confidence | Definitive, authoritative |
| **Creativity** | Conservative, by-the-book | Pragmatic innovation | Lateral, experimental |
| **Depth** | Surface-level, quick answers | Moderate exploration | Deep-dive, exhaustive |
| **Pedagogy** | Assumes expertise | Adaptive scaffolding | Assumes novice, teaches everything |

### Tier 3 — Identity Axes (who the persona is)

| Axis | Low (0.0) | Mid (0.5) | High (1.0) |
|------|-----------|-----------|------------|
| **Domain Affinity** | Generalist | Leaning specialist | Deep domain expert |
| **Temporal Frame** | Historical, retrospective | Present-focused | Forward-looking, speculative |
| **Agency** | Reactive, waits for direction | Collaborative, suggests | Proactive, takes initiative |

### Categorical Selectors (derived from hash, not noise field)

In addition to the continuous axes, the persona seed deterministically selects categorical values:

- **Domain Archetype** (1 of N): The broad expertise family — engineering, humanities, sciences, arts, business, systems, education, medicine, law, craft. Selected by `personaSeed % N`.
- **Voice Register** (1 of M): The linguistic personality type — academic, practitioner, mentor, provocateur, storyteller, analyst, coach, architect. Selected by `deriveChildHash(personaSeed, REGISTER_SALT) % M`.
- **Reasoning Style** (1 of K): The default approach — deductive, inductive, abductive, analogical, dialectical, empirical, narrative, systematic. Selected by `deriveChildHash(personaSeed, REASONING_SALT) % K`.

The categorical selectors are drawn from vocabulary pools. When a JSON profile is provided (following ZeroSwarm's profile pattern), the pools are externalizable. Default hardcoded pools ship with the system.

---

## The User String as Creative Interface

The beauty of string-as-seed is that the user gets two things simultaneously:

1. **Semantic intent** — they write what they want, and the words have meaning to them.
2. **Deterministic coordinates** — the hash doesn't care about semantics, only bytes. But coherent noise ensures that strings which *feel* similar (because they share substrings) often produce personas that *are* similar.

This creates a satisfying exploration dynamic. The user can:

- Type a description and get a persona that *feels right* because the noise field produces coherent results.
- Tweak the string to nudge the persona ("add more warmth" → change the string → get a nearby persona).
- Share persona strings with others — a string is a portable, reproducible persona reference.
- Bookmark strings that work well (see: Top Ten system below).

The string does not need to describe the desired persona. `"banana"` produces a valid persona. `"42"` produces a valid persona. But descriptive strings create a better user experience because the user can remember what they intended.

---

## Top Ten — Persona Coordinate Memory

If a user finds a persona they like, they should be able to keep it. The Top Ten system stores up to 10 persona references for quick recall.

### What Is Stored

```typescript
interface PersonaBookmark {
  label: string;           // User-assigned name ("My Code Reviewer")
  inputString: string;     // The original user string
  personaSeed: bigint;     // The computed hash (for fast reconstruction)
  coordinates: number[];   // The N-dimensional coordinate vector
  createdAt: number;       // Timestamp of bookmark creation
  useCount: number;        // How many times recalled
}
```

### Why Store Coordinates?

The `inputString` alone is sufficient to reconstruct the persona (hash it again). But storing the pre-computed `coordinates` array allows the UI to display a visual fingerprint of the persona without re-running the hash pipeline. It also enables coordinate-space operations: "show me personas similar to this one" becomes a nearest-neighbor query in coordinate space.

### Storage

For SYNTH-Framework: persisted via Tauri's filesystem API to `%APPDATA%/synth-framework/personas.json`.

For standalone JSX component: persisted via the host application's storage mechanism (props callback). The component itself is storage-agnostic — it emits bookmark events and accepts a bookmarks array as props.

---

## Standalone JSX Component Architecture

ZeroPersona is designed as a **self-contained React component** that can be dropped into any project. The SYNTH-Framework integration is one consumer. The component is the product.

### Component Boundary

```
┌─────────────────────────────────────────────────┐
│  ZeroPersona (standalone JSX component)          │
│                                                   │
│  Props In:                                        │
│    worldSeed: bigint                              │
│    profile?: PersonaProfile (vocabulary override)  │
│    bookmarks?: PersonaBookmark[]                  │
│    onPersonaGenerated: (persona: Persona) => void │
│    onBookmarkSave: (bookmark: PersonaBookmark) => void │
│    onBookmarkDelete: (label: string) => void      │
│    theme?: 'light' | 'dark' | 'system'           │
│                                                   │
│  Internal:                                        │
│    String input → hash → coordinates → dimensions │
│    → prompt assembly → Persona object emitted     │
│    Bookmark management UI                         │
│    Coordinate visualizer (radar chart / fingerprint)│
│    Dimension inspector (shows all axis values)    │
│                                                   │
│  Emits:                                           │
│    Persona { systemPrompt, dimensions, seed, ... }│
│    BookmarkEvent { action, bookmark }             │
│                                                   │
│  Zero external dependencies beyond React.         │
│  All hash/noise functions are internal pure fns.  │
│  No API calls. No storage. No side effects.       │
└─────────────────────────────────────────────────┘
```

### The Persona Output Object

```typescript
interface Persona {
  inputString: string;          // What the user typed
  personaSeed: bigint;          // The 64-bit hash
  coordinates: number[];        // N-dimensional coordinate vector [0,1]
  dimensions: PersonaDimensions; // Named dimension values
  categoricals: {
    domainArchetype: string;
    voiceRegister: string;
    reasoningStyle: string;
  };
  systemPrompt: string;         // The assembled natural-language prompt
  identityTag: string;          // 8-hex fingerprint for display
  timestamp: number;            // Generation timestamp (not used in hash)
}

interface PersonaDimensions {
  // Tier 1 — Communication
  verbosity: number;
  formality: number;
  directness: number;
  warmth: number;
  humor: number;
  // Tier 2 — Cognitive
  abstraction: number;
  certainty: number;
  creativity: number;
  depth: number;
  pedagogy: number;
  // Tier 3 — Identity
  domainAffinity: number;
  temporalFrame: number;
  agency: number;
}
```

### Integration with SYNTH-Framework

In SYNTH, the `ZeroPersona` component replaces (or supplements) LAINA's static system prompt. The user can:

1. Open the persona panel in the speech bubble UI.
2. Type a persona string.
3. See the generated persona dimensions visualized.
4. Accept → the persona's `systemPrompt` is injected into the Claude Code prompt assembly pipeline (replacing the `{PERSONA_CONTEXT}` placeholder or appending to LAINA's base prompt).
5. Bookmark → stored in `personas.json` via Tauri filesystem.

LAINA's base persona remains the default. ZeroPersona is an overlay — activated when the user wants a different voice.

### Integration with Any Other Project

```tsx
import { ZeroPersona } from './zero-persona';

function MyApp() {
  const [activePersona, setActivePersona] = useState(null);

  return (
    <ZeroPersona
      worldSeed={12345n}
      onPersonaGenerated={(persona) => {
        setActivePersona(persona);
        // Use persona.systemPrompt with your AI API call
      }}
      onBookmarkSave={(bookmark) => {
        // Persist to your storage
      }}
    />
  );
}
```

---

## Pure Function Library: `zero-persona/`

The core logic lives in a pure-function directory with zero UI imports — identical discipline to ZeroSwarm's `zero-specialist/`.

```
zero-persona/
  types.ts              ← All interfaces
  hash.ts               ← FNV-1a 64-bit (reused from ZeroSwarm, exact same code)
  normalize.ts          ← String normalization (lowercase, trim, collapse whitespace)
  coordinate-mapper.ts  ← Seed → N-dimensional coordinates via child hash derivation
  noise-field.ts        ← Octave-layered coherent noise (adapted from capability-field.ts)
  dimension-resolver.ts ← Noise values → named PersonaDimensions
  categorical-selector.ts ← Hash-based categorical picks from vocabulary pools
  prompt-assembler.ts   ← Dimensions + categoricals → natural-language system prompt
  profile.ts            ← Optional JSON profile loader/validator
  index.ts              ← Public API barrel export
```

### What Is Reused from ZeroSwarm

| Module | Reuse Level | Notes |
|--------|-------------|-------|
| `hash.ts` | **Verbatim** | FNV-1a, packBytes, hashToFloat, deriveChildHash — identical code |
| `noise-field.ts` | **Adapted** | Same coherentNoise algorithm, but operating on N axes instead of 2D `(roleX, styleY)` grid. The gridHash, smoothstep, and octave layering are identical. |
| `prompt-assembler.ts` | **New** | ZeroSwarm's prompt-generator.ts is swarm-specific. ZeroPersona needs a new assembler for the richer dimension set. Same *pattern* (threshold → instruction fragment → template assembly) but different vocabulary and template structure. |
| `profile.ts` | **Adapted** | Same concept (externalize vocabulary pools via JSON), different schema (persona dimensions instead of specialist dimensions). |

### What Is New

| Module | Purpose |
|--------|---------|
| `normalize.ts` | String normalization. ZeroSwarm never needed this — seeds were integers. |
| `coordinate-mapper.ts` | Maps a single 64-bit hash to N independent coordinate floats. ZeroSwarm had explicit `roleId`/`styleId` integers — ZeroPersona derives all coordinates from one hash. |
| `dimension-resolver.ts` | Maps raw noise values to the 13-axis PersonaDimensions schema. |
| `categorical-selector.ts` | Selects domain archetype, voice register, reasoning style from pools. |

---

## Hash Determinism Contract

The same contract as ZeroSwarm, restated for ZeroPersona:

> Given the same `(inputString, worldSeed)` pair, `generatePersona()` produces a byte-identical `Persona` object across all machines, all execution orders, and all sessions.

No `Math.random()`. No `Date.now()` in the generation path. No external state. The `timestamp` field on the Persona object is metadata, not a hash input.

The `worldSeed` exists so that different installations (or different projects using ZeroPersona) can produce different persona spaces from the same input strings. Two users typing `"helpful assistant"` with different world seeds get different personas. Same world seed, same string → same persona.

---

## Coherent Noise Adaptation

ZeroSwarm's coherent noise operates on a 2D integer grid `(roleX, styleY)`. ZeroPersona adapts this to operate on a 1D coordinate per dimension, with the dimension index serving as the salt (exactly as ZeroSwarm uses salt spacing of 1000 between dimensions).

```
For each dimension axis i:
  coordinate[i] = hashToFloat(deriveChildHash(personaSeed, i))
  rawValue[i]   = coherentNoise1D(coordinate[i], salt=i*1000, worldSeed, octaves=4)
  normalized[i] = (rawValue[i] + 1) / 2  // map [-1,1] → [0,1]
```

The 1D coherent noise function is a simplification of the 2D version: instead of bilinear interpolation over a 2D grid, it uses linear interpolation over a 1D grid. The gridHash, smoothstep, and octave parameters are identical.

This preserves the critical property: **nearby coordinates produce correlated dimension values**. A persona seed that maps to coordinate 0.500 on the warmth axis will produce a similar warmth value to a seed at coordinate 0.502. Strings that hash to nearby regions of persona-space produce related personas.

---

## Prompt Assembly Strategy

The assembled system prompt follows this structure:

```
You are a {voiceRegister} with deep expertise in {domainArchetype}.

{communication_instructions}

{cognitive_instructions}

{identity_instructions}

Your reasoning approach is {reasoningStyle}. {reasoning_elaboration}

{optional_profile_addendum}
```

Each instruction block is composed from the dimension values:

```typescript
function communicationInstructions(dims: PersonaDimensions): string {
  const parts: string[] = [];
  
  // Verbosity
  if (dims.verbosity < 0.33) parts.push("Keep responses concise and to the point.");
  else if (dims.verbosity > 0.66) parts.push("Provide thorough, detailed explanations.");
  
  // Formality
  if (dims.formality < 0.33) parts.push("Use casual, conversational language.");
  else if (dims.formality > 0.66) parts.push("Maintain formal, precise language.");
  
  // ... etc for each communication axis
  
  return parts.join(" ");
}
```

The three-tier threshold (low < 0.33, mid 0.33–0.66, high > 0.66) matches ZeroSwarm's proven pattern. Mid-range values produce no instruction for that axis, allowing natural default behavior — this is intentional. Not every persona needs an opinion on every dimension.

---

## Profile System

Following ZeroSwarm's JSON profile pattern, ZeroPersona supports optional vocabulary pool externalization:

```typescript
interface PersonaProfile {
  id: string;
  name: string;
  version: number;
  domainArchetypes: string[];       // ["engineering", "humanities", ...]
  voiceRegisters: string[];         // ["academic", "practitioner", ...]
  reasoningStyles: string[];        // ["deductive", "inductive", ...]
  dimensionInstructions: {
    [dimensionName: string]: {
      low: string[];    // Instruction pool for values < 0.33
      mid: string[];    // Instruction pool for values 0.33–0.66 (optional)
      high: string[];   // Instruction pool for values > 0.66
    };
  };
  promptTemplates: string[];        // Template variants for final assembly
}
```

When no profile is provided, hardcoded defaults are used. When a profile is active, the hash selects from profile pools instead — the hash arithmetic is identical, only the vocabulary changes. Same seed + same profile = same persona, deterministically.

---

## Visual Component: Coordinate Fingerprint

The JSX component includes a visual representation of the persona's coordinate vector. This serves as both a fingerprint (recognizable at a glance) and an editing affordance (inspect which dimensions are active).

**Radar Chart:** A 13-axis radar chart showing all dimension values. Each axis labeled. The filled shape is the persona's "signature". Two personas can be visually compared by overlaying their radar charts.

**Compact Fingerprint:** A small colored bar or glyph derived from the coordinate vector — suitable for display alongside a bookmark label. Uses the same `identityTag` (8-hex) as ZeroSwarm for text representation.

---

## Relationship to ZeroSpecialist/ZeroSwarm

| Concern | ZeroSpecialist/ZeroSwarm | ZeroPersona |
|---------|--------------------------|-------------|
| Seed source | 4-integer tuple (automated) | User string (manual) |
| Hash function | FNV-1a 64-bit | FNV-1a 64-bit (identical) |
| Coordinate space | 2D (roleId, styleId) | 13D (persona axes) |
| Noise field | 2D coherent noise | 1D coherent noise per axis |
| Behavioral dimensions | 5 continuous + 3 categorical | 13 continuous + 3 categorical |
| Prompt assembly | Swarm-optimized templates | Human-facing persona templates |
| Selection | Automated sample-benchmark-rotate | User chooses, bookmarks favorites |
| Storage | WinnerRecord (128 bits) | PersonaBookmark (string + metadata) |
| Affinity | Pairwise agent collaboration | Not applicable |
| Epochs | Swarm rotation cycles | Not applicable |
| Stigmergic signals | Coordination field | Not applicable |
| Target consumer | Swarm orchestrator | Any AI API call needing a system prompt |

ZeroPersona is a **sibling**, not a child, of ZeroSpecialist. They share the same hash and noise infrastructure but serve fundamentally different use cases. A project could use both: ZeroSpecialist for automated agent management, ZeroPersona for user-facing persona selection.

---

## Implementation Priority

### Phase 1 — Core Pipeline (MVP)

1. `hash.ts` — copy from ZeroSwarm verbatim
2. `normalize.ts` — string normalization
3. `coordinate-mapper.ts` — seed → N coordinates
4. `noise-field.ts` — 1D coherent noise adapted from ZeroSwarm
5. `dimension-resolver.ts` — noise → PersonaDimensions
6. `categorical-selector.ts` — hash → categorical picks
7. `prompt-assembler.ts` — dimensions → system prompt string
8. `types.ts` — all interfaces
9. `index.ts` — barrel export

**Deliverable:** `generatePersona(inputString, worldSeed) → Persona` as a pure function.

### Phase 2 — JSX Component

1. `ZeroPersona.tsx` — main component with string input, dimension display, prompt preview
2. Radar chart visualization
3. Bookmark management UI (save, recall, delete)
4. Compact fingerprint display
5. Theme support (light/dark/system)

**Deliverable:** Importable `<ZeroPersona />` component.

### Phase 3 — SYNTH Integration

1. Mount ZeroPersona in SYNTH speech bubble UI
2. Wire `onPersonaGenerated` to Claude Code prompt assembly
3. Wire bookmark persistence to Tauri filesystem
4. Add persona toggle (LAINA default vs ZeroPersona override)

### Phase 4 — Profile System

1. JSON profile schema and validator
2. Profile loader
3. Default profile constant
4. Profile-aware prompt assembler path
5. Profile UI in settings

---

## Testing Contract

Identical discipline to ZeroSwarm's 9-layer audit, scoped to ZeroPersona's pipeline:

1. **Determinism:** Same `(inputString, worldSeed)` → identical Persona across 10 trials.
2. **Normalization idempotence:** `normalize(normalize(s)) === normalize(s)`.
3. **Coordinate range:** All coordinates in [0, 1].
4. **Dimension range:** All dimension values in [0, 1].
5. **Cross-string uniqueness:** 100 random strings → ≥ 90% unique system prompts.
6. **Prompt structure:** Every generated prompt contains domain archetype, voice register, reasoning style.
7. **Profile determinism:** Same `(inputString, worldSeed, profile)` → identical Persona.
8. **No anti-patterns:** Zero instances of `Math.random()`, `Date.now()`, or external state reads in the generation path.
9. **Bookmark round-trip:** Save bookmark → reconstruct from `inputString` → identical Persona.

---

## Summary

ZeroPersona takes the proven ZeroFamily hash-to-behavior pipeline and points it at the user. Instead of an automated swarm sampling an infinite specialist space, a human being navigates an infinite persona space by typing a string. The string is the seed. The seed is the coordinate. The coordinate is the persona. The persona is the prompt.

The system is a standalone JSX component backed by a pure-function library. It integrates into SYNTH-Framework as a persona overlay, and into any other React project via props. No API dependencies. No storage assumptions. No swarm machinery. Just a string in, a persona out.
