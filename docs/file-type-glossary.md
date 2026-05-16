# AppWorks file-type glossary

Every file or folder PA writes follows a `#suffix#` naming convention. This is the complete table of suffixes we've observed across real PA projects. Patterns are grouped by which project pattern uses them.

> If you encounter a suffix not listed here, **treat it as AppWorks-owned by default**. Update this file when you confirm what a new suffix is.

## Universal markers (both patterns)

| Suffix on filename | Meaning | Format |
|---|---|---|
| `#cws-prj#.cws`     | Project descriptor (root of the project) | XML |
| `#cws-qnr#.cws`     | Qualified Name Root (namespace container) | XML |
| `#cws-bpm#.cws`     | Business Process model | XML (with embedded SVG diagram) |
| `#cws-ma#.cws/`     | Mergeable Association — a *folder* containing many small `.cws` files for clean merging | folder |
| `#cws-tid#.cws`     | Translation IDentifier (a key for a translatable string) | XML |
| `#cws-trt#.cws`     | TRanslation Text (text for one identifier × one locale) | XML |
| `.identifiers`      | Filename → internal GUID map written by PA in every owned folder | properties |

## Traditional pattern (XForms,Java heavy development-style)

These appear in projects organised by artefact type — `BPM/`, `JS/`, `Webservices/`, `DB Metadata/`, etc.

| Suffix | Meaning |
|---|---|
| `#cws-xform#.cws`   | XForm UI definition |
| `#cws-webs#.cws`    | Web service / library definition |
| `#cws-wsds#.cws`    | Web service definition set |
| `#cws-wsi-r#.cws`   | Web service interface reference |
| `#cws-dbmd#.cws`    | Database metadata (drives the Java code generator) |
| `#cws-xmlsts#.cws`  | XML Store definition |
| `#cws-bi#.cws`      | Business Identifier |
| `#cws-sch#.cws`     | Scheduler |
| `#cws-ac-r#.cws`    | Adapter Connector reference |
| `#cws-jcd#.cws`     | Java Class Deployment descriptor |
| `#cws-wap#.cws`     | WS-AppServer Package |
| `#cws-wapjars#.cws` | WS-AppServer Jars bundle |

## Entity-based pattern (entity-modeling projects)

These appear in projects organised around entities — `Entities/`, `Layouts/`, `Processes/`, `Roles/`. Note the dual `#XXX#` (leaf) and `#XXXbb#` (bundle) convention: bare suffix is the leaf artefact; `bb` suffix is the container/bundle.

| Suffix | Meaning |
|---|---|
| `#ef#/`                          | Entity Folder — top-level container for an entity |
| `#cws-nativeentity#.cws`         | Entity definition (one per `#ef#` folder) |
| `#form#.cws` / `#formbb#.cws`    | Form / form bundle |
| `#list#.cws` / `#listbb#.cws`    | List / list bundle |
| `#layt#.cws` / `#laytbb#.cws`    | Layout / layout bundle |
| `#acbar#.cws` / `#acbarbb#.cws`  | Action Bar / action bar bundle |
| `#acflw#.cws` / `#acflwbb#.cws`  | Activity Flow / activity flow bundle |
| `#cat#.cws` / `#catbb#.cws`      | Category / category bundle |
| `#dedl#.cws` / `#dedlbb#.cws`    | Deadline / deadline bundle |
| `#emailt#.cws` / `#emailtbb#.cws`| Email Template / email template bundle |
| `#rule#.cws` / `#rulebb#.cws`    | Rule / rule bundle |
| `#ewsbb#.cws`                    | Entity Web Service bundle |
| `#fwso#.cws`                     | Flow With Sub Objects |
| `#lifecbb#.cws`                  | Lifecycle bundle |
| `#com.cordys.entity.services.*#.cws` | Pre-built entity service reference (HistoryLog, EmailConfig, etc.) |

## Mergeable Association pattern

PA's clever trick for git-friendly translations and similar collections. Instead of one giant XML file that merge-conflicts on every change, the contents are split across many tiny files inside a folder named `Name#cws-ma#.cws/`. Two developers editing different strings never collide.

Example structure:

```
TranslationInformation_<GUID>#cws-ma#.cws/
├── .identifiers
├── <GUID-1>#cws-tid#.cws    ← translation identifier 1
├── <GUID-2>#cws-tid#.cws    ← translation identifier 2
└── …

en-US_<GUID>#cws-ma#.cws/
├── .identifiers
├── <GUID-1>#cws-trt#.cws    ← en-US text for identifier 1
├── <GUID-2>#cws-trt#.cws    ← en-US text for identifier 2
└── …
```

This is well-engineered and largely undocumented in OT's public materials. Worth understanding before you assume PA's git output is messy.

## Folders to treat as black boxes

| Folder | Why |
|---|---|
| Anything ending in `#cws-ma#.cws` | Mergeable Association — owned by PA in full |
| Anything ending in `#ef#`         | Entity folder — owned by PA in full |
| `DB Metadata/<package>/` *(the WS-AppServer package folders)* | Mix of `*Base.java` (generated) and non-`Base.java` (user-extendable). Edit only the non-`Base` files. |
| `JavaArchive *` folders            | Compiled `.jar` outputs; never hand-edit |

## Recognising what's safe

Anywhere outside the patterns above is fair game for human/agent edits. Common safe paths in real projects:

```
JS/**/*.js
CSS/**/*.css
Web/**/*.htm
Web/**/*.html
DB Metadata/Java Source */com/**/*.java    (only files NOT ending in Base.java)
**/Webservices/*/Java Source*/**/*.java     (only files NOT ending in Base.java)
docs/**
tests/**
scripts/**
.github/workflows/**
```
