# Entity-pattern example: CaseRecord

A sanitized fragment showing the entity-folder layout used in modern AppWorks entity projects.

## What's here

```
examples/entity/
└── Entities/
    └── CaseRecord#ef#/
        ├── CaseRecord#cws-nativeentity#.cws    ← entity definition
        └── bb/
            └── ActionBars#acbarbb#.cws         ← one building-block bundle
```

A real entity folder typically contains a `bb/` directory with 20–50 `.cws` files (forms, lists, layouts, action bars, activity flows, deadlines, categories, email templates, rules, lifecycles…). This example shows only two files — enough to demonstrate the shape without padding the template with bulk.

All GUIDs are placeholders (`00000000-0000-0000-0000-000000000XXX`). Dates are synthetic. Real entities have real GUIDs and timestamps written by PA.

## Why there's no Java here

Entity-pattern projects rarely include hand-written Java inside the entity folders. Business logic for an entity is expressed via:

- **Rules** (`#rule#.cws` files, edited in PA)
- **Lifecycles** (`#lifecbb#.cws`, edited in PA)
- **Activity flows** (`#acflw#.cws`, edited in PA)
- **External services** PA calls via REST/messaging

The "external services" bullet is where AI agents come in — and they live in a **sibling repository**, not here. See [`docs/pattern-entity.md`](../../docs/pattern-entity.md) for the sibling-repo architecture.

## What an agent task looks like for an entity project

The agent never edits files in *this* repo. A well-formed agent task is:

> *"In the case-tracker-services sibling repo, implement a webhook handler at `src/webhooks/case-created.ts`. It receives PA's CaseRecord-created event, looks up the assigned legal team based on case category, calls our internal billing API to pre-allocate budget, and returns a confirmation payload PA stores on the entity."*

The agent works exclusively in the sibling repo. This repo (the AppWorks model repo) is effectively read-only to agents.

## What to delete after forking

This whole `examples/` folder. It exists to teach the pattern, not to ship in your real project.
