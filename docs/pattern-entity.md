# Entity-based AppWorks projects

Projects that look like this:

```
ProjectRoot/
├── EMail/
│   └── *EmailConfig*#.cws
├── Entities/
│   ├── CaseRecord#ef#/
│   │   ├── CaseRecord#cws-nativeentity#.cws
│   │   ├── bb/                              ← "bundle" — child container
│   │   │   ├── Actions#acbarbb#.cws
│   │   │   ├── ActivityFlow#acflwbb#.cws
│   │   │   ├── Categories#catbb#.cws
│   │   │   ├── Deadlines#dedlbb#.cws
│   │   │   ├── EmailTemplates#emailtbb#.cws
│   │   │   ├── Forms#formbb#.cws
│   │   │   ├── Lists#listbb#.cws
│   │   │   ├── Layouts#laytbb#.cws
│   │   │   ├── Lifecycle#lifecbb#.cws
│   │   │   ├── Rules#rulebb#.cws
│   │   │   ├── DefaultLayout#layt#.cws
│   │   │   ├── DefaultList#list#.cws
│   │   │   ├── Create#form#.cws
│   │   │   ├── … and many sub-entity folders (#ef#)
│   │   └── *HistoryLog*#.cws
│   ├── Litigation#ef#/
│   ├── Masterdata/
│   └── Subentities/
├── Layouts/                  (cross-entity layouts)
├── Processes/                (#cws-bpm# files, plus folders)
├── Roles/
├── Runtime/
├── User Interfaces/
├── Web/                      (typically empty or sparse)
├── TranslationInformation_<GUID>#cws-ma#.cws/
└── en-US_<GUID>#cws-ma#.cws/
```

This is the modern, structured pattern introduced with AppWorks' entity modeling. Everything is composed around domain entities. **Almost every file is a `.cws`** — PA owns the entire repo.

## Where AI agents help most in this pattern

The honest answer: **almost nothing inside this repo**.

Total agent-editable surface across the entire entity-pattern projects we surveyed: **zero non-`.cws` files**.

This is not a limitation of agents — it's a design choice in AppWorks' entity model. The behaviour of an entity (its lifecycle, rules, forms, lists, actions, deadlines, emails) is all expressed declaratively in PA's modeling tools and serialised to `.cws` XML. There is no "implementation" layer in the same way traditional WS-AppServer projects had Java method bodies.

So: the agent's job in entity-pattern projects is to work in a **sibling repository** — a separate codebase that PA calls into via REST, messaging, or external service references.

## The sibling-repo architecture

```
your-org-github/
├── case-tracker/                  ← THIS template fork; PA-owned model
│   ├── Entities/
│   ├── Processes/
│   └── … (all .cws)
│
└── case-tracker-services/         ← Sibling repo; AI-editable
    ├── README.md
    ├── package.json (or pom.xml, etc.)
    ├── src/
    │   ├── webhooks/               PA event subscribers
    │   │   ├── case-created.ts
    │   │   └── deadline-approaching.ts
    │   ├── rules/                  Externalised business rules
    │   ├── integrations/           3rd-party API calls
    │   └── reports/                Custom reporting endpoints
    └── tests/
```

The sibling repo is a regular code repo — TypeScript, Go, Java, whatever your team prefers. Claude Code lives in there full-time. The AppWorks repo (this one) is effectively read-only-by-agents.

## What the two repos exchange

| From PA repo → sibling | From sibling → PA |
|---|---|
| Entity events (case created, status changed, deadline missed) via PA's REST/webhook adapters | REST calls to the sibling for business rule evaluation, document generation, external lookups |
| BPM process activities that call out to external services | Activity completion callbacks back to PA |

## Where agents *can* contribute inside this repo (limited)

Even though the model files are off-limits, an agent can usefully:

1. **Document the model.** Generate human-readable summaries of what each entity does by reading (but not editing) the `.cws` XML — useful for onboarding new developers.
2. **Maintain `docs/`** in this repo: data dictionaries, ERD-style diagrams (Mermaid), workflow descriptions.
3. **Update the `.github/workflows/`** for build/release automation.
4. **Maintain the sibling-repo integration contracts** — OpenAPI specs, JSON schemas, message contracts that both sides honor.

## A canonical example

See [`examples/entity/`](../examples/entity/) for a sanitized `CaseRecord#ef#/` skeleton with the typical `bb/` folder layout. It's shape-only — every GUID and string has been sanitized — but it accurately reflects what you'll see when PA commits a real entity.

## Common foot-guns specific to this pattern

1. **The temptation to add Java alongside the entity.** Some developers, frustrated by the lack of an implementation layer, try to drop Java classes inside the entity folders. PA does not pick them up — they're either ignored or actively cleaned up on the next sync. Use a sibling repo.

2. **The `bb/` folder is not a "bundle" you can edit.** It's PA's serialization layout — short for "behavior bundle". Treat it as part of the entity folder (`#ef#`) and off-limits.

3. **Sub-entities are still entities.** Recursive `#ef#` folders inside a parent entity's `bb/` directory (`bb/Contents#ef#`, `bb/Discussions#ef#`) are real entities with their own lifecycles. Same rules apply.

4. **Translation collisions are NOT impossible.** PA's mergeable-association folders (`#cws-ma#.cws/`) shard translations across many tiny files, but two developers translating the same string in the same locale will still conflict on the same `#cws-trt#.cws` file. The shard is *per-string-per-locale*, not finer.
