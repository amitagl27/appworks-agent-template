# Traditional-pattern AppWorks projects

Projects that look like this:

```
ProjectRoot/
├── BPM/
│   ├── ProcessA#cws-bpm#.cws
│   └── ProcessB#cws-bpm#.cws
├── BusinessIdentifier/
│   └── *#cws-bi#.cws
├── CSS/
│   ├── theme.css
│   └── orange.css
├── DB Metadata/
│   ├── ProjectName#cws-dbmd#.cws
│   ├── Java Source ProjectName/
│   │   └── com/example/serviceops/
│   │       ├── Notification.java         ← user-extendable
│   │       ├── NotificationBase.java     ← generated
│   │       ├── Order.java
│   │       ├── OrderBase.java
│   │       └── …
│   └── JavaArchive ProjectName/
│       └── ProjectName.jar               (build output)
├── Images/
│   ├── logo.png
│   └── icon.gif
├── JS/
│   ├── ExportToExcel.js
│   ├── flash_detect.js
│   └── validation.js
├── Jars/
│   └── Uploadjars#cws-jcd#.cws
├── Scheduler/
│   └── DailyJob#cws-sch#.cws
├── UserInterface/
│   ├── LandingPage/
│   │   └── Home#cws-xform#.cws
│   └── *.cws (xforms)
├── Web/
│   └── *#cws-webs#.cws  + static .htm files
├── Webservices/
│   ├── CustomWebservice/
│   ├── ExternalREST/
│   └── *#cws-wsds#.cws
├── XMLStore/
│   └── *#cws-xmlsts#.cws
├── ProjectName#cws-prj#.cws
└── runtimeQNR for project#cws-qnr#.cws
```

This is the older, more freeform structure that's been part of Cordys/AppWorks for over a decade. It carries a **large agent-editable surface** — typically hundreds of files of Java, JS, CSS, HTML.

## Where AI agents help most in this pattern

### 1. Java business-object methods

The biggest single agent win. PA generates a `XxxBase.java` (do not touch) and a `Xxx.java` (user-extendable) per business object. Method skeletons in `Xxx.java` have bodies like `// TODO implement body`. That's exactly the surface an AI agent excels at: tightly-scoped, well-typed, test-driven.

Example workflow:

1. Modeler defines a new business object `Order` in PA — commits `Order#cws-dbmd#.cws` and the generator writes `OrderBase.java` (full schema) and `Order.java` (skeleton).
2. Developer pulls. Asks the agent: *"Implement `submitOrder()` in `Order.java`. It should validate the line items against pricing rules in `PricingService`, persist the order, and emit an event to the billing webservice."*
3. Agent reads `OrderBase.java` for the schema, finds `PricingService` in a sibling Java source folder, writes `submitOrder()` in `Order.java`, adds unit tests.
4. Commit. CI passes (no `.cws` or `*Base.java` touched). Push.

### 2. JavaScript for xforms

JS files in `JS/`, and inline scripts referenced by `#cws-xform#.cws` files. The xform XML is off-limits, but the `.js` files it loads are agent-editable. Common asks: validation functions, formatters, browser-side helpers.

### 3. CSS theme customization

Pure agent-editable. PA doesn't touch `.css` files unless they're referenced via `#cws-webs#.cws` definitions (and even then, only the definition file is PA-owned; the actual `.css` is fair game).

### 4. Static web assets

`.htm`, `.html`, images, JS libraries dropped into `Web/`. Anything that's not `.cws`.

### 5. External integration code

`Webservices/CustomWebservice/` and `Webservices/ExternalREST/` typically hold custom Java code that calls third-party APIs. Once the `#cws-wsds#.cws` definition exists (modeler's job), the implementation Java is yours.

## Where agents must NOT help

- BPM process logic (those are `#cws-bpm#.cws` files — model-level changes)
- xForm UI structure (`#cws-xform#.cws` — open in PA's xform designer)
- Schedulers, business identifiers, adapter connectors, XML Store definitions (all `.cws`)
- Database metadata — even though it drives Java generation, the metadata itself is modeled in PA
- Any `Base.java` — fully generated, regenerated on metadata change

## A canonical example

See [`examples/traditional/`](../examples/traditional/) for a sanitized fragment. It shows a `Notification.java` / `NotificationBase.java` pair with a realistic method skeleton ready for an agent to implement.

## Common foot-guns specific to this pattern

1. **`Base.java` looks editable.** It compiles. Your edits even seem to work. They survive until the next time a developer regenerates from metadata — then your code is silently gone. The CLAUDE.md rule against `*Base.java` exists for exactly this reason.

2. **`.js` files are not always agent-safe.** A `.js` file inside a `#cws-ma#.cws/` folder (rare but exists) is part of a mergeable association — off-limits. Check the *folder name* before assuming a file extension means "safe."

3. **`xforms` mention JavaScript.** Inline JavaScript inside a `#cws-xform#.cws` file is part of the xform XML — do not edit it directly. Edit the referenced external `.js` file instead, or have the modeler change the xform.
