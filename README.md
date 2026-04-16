# monkey-c-garmin

An [agent skill](https://github.com/vercel-labs/skills) for **Monkey C / Garmin Connect IQ** development. Helps AI coding assistants write correct Monkey C code for Garmin wearable devices.

## What is this?

Monkey C looks like Java/JavaScript but has critical syntax differences — apostrophe comments (`'`), dollar-sign hex literals (`$FF`), `using` instead of `import`, PascalCase methods, reference-counted memory, and more. Without guidance, AI assistants default to Java/JS conventions and generate broken code.

This skill teaches your AI assistant how to write idiomatic Monkey C for Garmin Connect IQ apps.

## What's covered

- **Monkey C language** — syntax, operators, control flow, containers, type system, memory management, exceptions, annotations
- **App types** — Watch Faces, Data Fields, Widgets, Device Apps, Audio Content Providers
- **Application lifecycle** — AppBase, View lifecycle, input handling, view stack navigation
- **Graphics** — Dc drawing API, colors, fonts, round screen math, buffered bitmaps, AMOLED features
- **Toybox API** — all 37 modules (Sensor, Communications, Position, Activity, WatchUi, Graphics, BLE, ANT+, Weather, etc.)
- **Project structure** — manifest.xml, monkey.jungle, resource XML, barrels, build configuration
- **Sensors & comms** — heart rate, GPS, accelerometer, HTTP requests, phone messaging, BLE, ANT+
- **Background services** — temporal events, service delegates, complications
- **Data persistence** — Application.Storage, Application.Properties, settings UI
- **UX guidelines** — screen shapes, display types (MIP/AMOLED), device compatibility, design principles
- **Working templates** — complete code for watch face, data field, widget, device app, and common patterns

## Installation

```bash
# Install into a project
npx skills add TheoInTech/monkey-c-garmin -a claude-code

# Install globally
npx skills add TheoInTech/monkey-c-garmin -a claude-code -g
```

## Skill structure

```
monkey-c-garmin/
├── SKILL.md                                  # Core reference (~490 lines)
└── references/
    ├── language-syntax.md                    # Operators, control flow, containers, classes, modules
    ├── app-types-and-lifecycle.md            # 5 app types with full lifecycle details
    ├── ui-graphics-layouts.md               # Dc drawing API, layouts, input handling
    ├── toybox-api-reference.md              # All 37 Toybox modules with key methods
    ├── project-structure-and-build.md       # manifest.xml, jungle, resources, barrels, submission
    ├── sensors-comms-background.md          # Sensors, GPS, BLE, ANT+, HTTP, background services
    ├── data-persistence-and-properties.md   # Storage, Properties, settings UI
    ├── ux-guidelines-and-device-targets.md  # Screen types, UX patterns, device compatibility
    └── common-patterns-and-examples.md      # Complete working code templates
```

## How it works

The skill uses **progressive disclosure** to stay context-efficient:

1. **Activation** — the skill triggers when you work with `.mc` files, Connect IQ projects, `manifest.xml`, `.jungle` files, or mention Garmin/Monkey C
2. **SKILL.md loads** — covers syntax differences, naming conventions, app lifecycle, graphics essentials, and common pitfalls (everything needed for ~80% of tasks)
3. **References load on-demand** — when deeper knowledge is needed (e.g., BLE communication, full Toybox API, complete watch face template), the relevant reference file is loaded

## Compatible agents

Built on the [open skills standard](https://github.com/vercel-labs/skills). Works with:

- Claude Code
- Cursor
- GitHub Copilot
- Cline
- Windsurf
- And [40+ other agents](https://skills.sh)

## Links

- [Garmin Connect IQ Developer Docs](https://developer.garmin.com/connect-iq/overview/)
- [Garmin Monkey C Reference](https://developer.garmin.com/connect-iq/monkey-c/)
- [Garmin Connect IQ API Docs](https://developer.garmin.com/connect-iq/api-docs/)
- [Open Skills Standard](https://github.com/vercel-labs/skills)
- [Skills Directory](https://skills.sh)
