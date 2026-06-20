# my-skills

Personal Claude skills library — a collection of specialized skills that extend Claude's behavior for specific domains and workflows.

Each skill acts as a focused expert mode: when invoked, it loads domain-specific rules, architecture patterns, and step-by-step workflows so Claude behaves like a specialist rather than a generalist.

## Structure

```
my-skills/
├── <skill-name>/
│   ├── SKILL.md          # Skill definition and instructions
│   └── references/       # Supporting reference docs (optional)
└── ...
```

## Install a skill

1. Download the `.skill` file from [Releases](../../releases)
2. Claude.ai → Settings → Skills → Install from file

## Build a skill locally

Requires the [skill-creator scripts](https://github.com/anthropics/claude-skills/tree/main/skill-creator/scripts).

```bash
python -m scripts.package_skill <skill-name> ./dist
```

## Skills

| Skill | Domain | Description |
|---|---|---|
| [guitar-riff-composer](./guitar-riff-composer/) | Music | Composes original guitar riffs with ASCII tablature, music theory context, and tone/FX suggestions. Supports any genre and player level. |
| [kmp-engineer](./kmp-engineer/) | Mobile / KMP | Acts as a Staff/Principal Engineer specialized in Kotlin Multiplatform and Compose Multiplatform. Applies clean architecture, typed error handling, injected dispatchers, and production-ready patterns across shared and platform-specific code. |
| [facade-floorplan-designer](./facade-floorplan-designer/) | Architecture | Diseñador conceptual de fachadas y plantas arquitectónicas para proyectos de casa propia. Traduce una descripción en lenguaje natural a alzados y planos esquemáticos SVG con proporciones razonables, incluyendo cálculo de áreas, recomendaciones de orientación solar y diseño bioclimático. |
| [mx-real-estate-advisor](./mx-real-estate-advisor/) | Real Estate | Asesor inmobiliario y financiero especializado en México. Calcula capacidad Infonavit, compara esquemas de financiamiento, analiza zonas por plusvalía y demanda, y recomienda desarrolladoras con historial verificable. |

---

## Skill details

### guitar-riff-composer

Triggered by requests to compose guitar riffs, licks, tabs, chord progressions, or any guitar musical idea — even casual ones like "dame algo para tocar" or "quiero un riff de blues".

**What it does:**
- Gathers genre, feel, tuning, and reference artist
- Composes a riff using appropriate scale and time signature
- Delivers: tablature (ASCII), music theory context, chord progression, tone/FX suggestion, and iteration options

**Reference files:** `scales.md`, `artist-styles.md`

---

### kmp-engineer

Triggered by any KMP/Compose Multiplatform question, design decision, or implementation task — including architecture reviews, `expect/actual` patterns, Ktor setup, kotlin-inject wiring, and iOS distribution via KMMBridge/SPM.

**Stack:** Kotlin Multiplatform · Compose Multiplatform · Ktor · kotlin-inject · Coroutines/Flow · kotlinx.serialization · Coil · KMMBridge + SPM

**What it does:**
- Enforces clean architecture (Domain → Data → Presentation) across shared modules
- Maximizes `commonMain` logic, minimizes platform-specific code
- Applies typed `DomainError` sealed interfaces and `runCatching` at data boundaries
- Injects `CoroutineDispatchers` everywhere for testability
- Produces ktlint and Detekt compliant code

**Reference files:** `architecture.md`, `compose.md`, `testing.md`, `code-quality.md`

---

### facade-floorplan-designer

Triggered by requests to design house facades and floor plans — descriptions of room programs, lot dimensions, architectural style, or any mention of "diseñar fachada", "planos de mi casa", "distribución de planta baja/alta", "boceto de fachada", or similar phrases in a Mexican residential construction context.

**What it does:**
- Gathers the architectural program (lot dimensions, room list per level, style, construction type)
- Calculates and displays area tables with typical ranges and COS (ground occupancy ratio)
- Generates SVG floor plans per level using `visualize:show_widget` following drawing conventions
- Generates a consistent facade elevation with proper proportions, openings, roof type, and material zones
- Provides climate-responsive design recommendations based on lot orientation
- Supports iteration — adjusts individual levels or the facade without regenerating everything

**Reference files:** `svg-template.md`, `architectural-styles.md`, `floorplan-drawing-guide.md`, `facade-elevation-guide.md`, `climate-design-principles.md`

---

### mx-real-estate-advisor

Triggered by questions about buying property in Mexico, Infonavit credit capacity, financing options, zone analysis, or any real estate investment question in a Mexican context.

**What it does:**
- Loads the user's financial profile (income, savings, Infonavit balance, existing assets)
- Calculates Infonavit credit capacity with visible formulas (VSM-based)
- Searches current listings, prices per m², and active developers using real portals (Lamudi, Inmuebles24, Vivanuncios)
- Delivers structured analysis: recommended zones, developers, financing comparison, risks, and specific project links
- Separates verified facts from estimates from advisor opinion

**Reference files:** `infonavit-calc.md`, `financing-calc.md`, `analysis-framework.md`
