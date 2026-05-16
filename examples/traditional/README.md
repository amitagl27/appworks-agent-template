# Traditional-pattern example: ServiceOps

A sanitized fragment showing the Java generated/user-extendable split that AI agents target in traditional AppWorks projects.

## What's here

```
examples/traditional/
└── DB Metadata/
    └── Java Source ServiceOps/
        └── com/
            └── example/
                └── serviceops/
                    ├── Notification.java       ← user-extendable; agent edits HERE
                    └── NotificationBase.java   ← generated; agent must NOT touch
```

This mirrors the layout you'll find in a real traditional-pattern project's `DB Metadata/Java Source <ProjectName>/com/<package>/` folder.

## What an agent task looks like

Given a BPM process that has an activity `Send Notification` bound to `com.example.serviceops.Notification#send`, a well-formed agent task is:

> *"Implement `Notification.send(recipientId, channel, payload)` in `examples/traditional/DB Metadata/Java Source ServiceOps/com/example/serviceops/Notification.java`. It should look up the recipient's preferred channel if `channel` is null, dispatch via the matching connector, and return `"true"` on success or the error message on failure."*

The agent:
1. Reads `NotificationBase.java` to understand the schema (getters/setters exist for `RecipientId`, `Channel`, `Payload`, `Status`).
2. Implements `send()` in `Notification.java` — the method body is currently `// TODO implement body`.
3. Adds unit tests under `tests/`.
4. Commits. The pre-commit hook checks: no `.cws` touched, no `*Base.java` touched. Pass.

## What to delete after forking

This whole `examples/` folder. It exists to teach the pattern, not to ship in your real project.
