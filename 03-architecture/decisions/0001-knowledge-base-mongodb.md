# 0001. Store the knowledge base in MongoDB

- **Status:** Accepted
- **Date:** 2026-11-13, updated 2026-11-17

> **Scope note (2026-11-17):** SproutPal is currently being validated with a **demo**: a website with course topics and a few encyclopedia entries, shown to a potential partner garden center. The demo uses only the core of this decision: MongoDB on the VPS (points 1–5 and 8), with content loaded manually. Versioning, the listener service, backups and monitoring (points 6, 7 and 9) apply when SproutPal goes into production.

## Context

The SproutPal knowledge base has two parts (see [concept](../../01-product/concept.md)):

- **Course topics:** free-form lessons with any number of paragraphs, images and highlighted notes.
- **Encyclopedia entries:** species, varieties, rootstocks, pests and diseases. Entries of the same kind share the same fields, but the possible values differ by species.

Every topic and entry stands on its own. Entries refer to each other (for example a variety lists its rootstocks and pollinators), but no entry needs another entry's content embedded in it.

The application reads the knowledge base from a database. The first version is read-only for users and has no user accounts. The expected size is hundreds to a few thousand documents.

## Decision

1. The knowledge base is stored in a **document database: MongoDB**. Each course topic and each encyclopedia entry is one document.
2. **References** between documents are stored as lists of `{id, name}` pairs, for example `{"id": "rootstock:m9", "name": "M9"}`:
   - `id` is a stable identifier that does not change when an entry is renamed;
   - `name` is stored so the reference can still be displayed if the target is missing.
3. The client renders each reference as a link. If the target document does not exist, the name is still shown, but not as a link.
4. Reverse lists (for example "varieties suitable for the M9 rootstock" on the M9 page) are not stored; they are produced by queries over the referring documents.
5. When an entry is renamed, the `name` in every reference to it is updated as well.
6. There is **no editing interface** in the first version. Content is written into the database with AI assistance, through a small **write script**. An editing interface may be added later.
7. **Every change creates a new version, and every previous version is kept in full.**
   - Each document carries a version number and a `changeNote` describing the change. A MongoDB schema validation rule (`$jsonSchema`) makes `changeNote` mandatory, and the write path checks that a new version number comes with a new `changeNote`.
   - Previous versions are stored by **events, not by the writer**: a small listener service subscribes to MongoDB **change streams** with pre-images enabled (`changeStreamPreAndPostImages`) and copies the full previous version of every changed or deleted document into a `history` collection.
   - Because history is filled from database events, it is created no matter where the write comes from: the write script, a future editing interface or a manual fix.
   - If the listener sees a change without a new `changeNote`, it only **reports** it; the change stays live.
   - The listener runs in a **Docker container** with automatic restart. Its logs and metrics go to the existing **Grafana Cloud** setup (Loki, Prometheus), which raises alerts — including when the listener stops sending a heartbeat — and provides a dashboard.
8. For the first version, MongoDB is **self-hosted on a rented VPS** (Contabo), as a single-node replica set, together with the listener service.
9. **Backup:** the knowledge base content — course topics, encyclopedia entries and their `history` — is backed up to a **private GitHub repository** as plain JSON, one file per document, so changes are readable in git as well.
   - The **listener service** does it, driven by the same change stream events: changed documents are written to the backup repository and pushed. Pushes are batched (for example after a short quiet period) instead of one push per change.
   - **On every start**, the listener first makes a **full backup**, so changes missed while it was down are covered as well.
   - Backup is decoupled from history: if GitHub is unreachable, history is still written, and the backup is retried later.
   - User data (accounts, course progress) is **not backed up** for now: names and e-mail addresses come back from the SSO provider at the next sign-in, and losing progress is acceptable in the first version.

## Reasons

- The content is made of self-contained documents with a flexible structure. Forcing free-form course topics into relational tables would add complexity without benefit.
- At the expected size, both document and relational databases perform well, so the data model decides.
- Use the right tool for each problem: if garden planning later needs relational data (users, plots, plantings), it can use a separate relational database.

## Consequences

- Positive: the document structure matches the content; course topics can evolve without schema migrations.
- Negative: the `name` in a reference is a copy and can drift from the target's name if a rename is not propagated.
- Negative: broken references are not prevented, only displayed safely. A missing pollinator link still shows the name, but nothing reports the problem.
- Negative: if a relational database is added later, it will refer to knowledge base identifiers across databases, with the same broken-reference risk.
- Positive: every earlier version is kept in full, so documents can be restored, compared (diff) and shown "as of" a date; more features can be built on this later.
- Positive: history does not depend on the writer remembering to save it.
- Negative: change streams require a replica set (a single-node replica set is enough; MongoDB Atlas always runs one).
- Negative: the listener must be running. After downtime it resumes from its resume token, but only while the events are still in the oplog; a long outage can lose history.
- Negative: schema validation can require a `changeNote`, but cannot check that it was updated for this change; the write script has to ensure that.
- Negative: self-hosting means we are responsible for backups, updates and security. The `history` collection is not a backup: it lives in the same database on the same server.
- Positive: the content backup lives off the server, in a private repository, and needs no extra service or scheduled job; it is close to real time.
- Positive: the full backup on start also covers events lost during a long listener outage.
- Negative: the VPS needs write access to the backup repository (a deploy key limited to that one repository).
- Negative: if the VPS is lost, users lose their course progress.

## Open points

- How a restore from the backup repository works.
- Securing MongoDB on the VPS (authentication, no public database port).
- The exact format of course topic content (structured blocks or marked-up text).
- How the oplog window is sized.
- How content written with AI assistance reaches the database in production (a write API on the VPS, file import, or manual upload).
- Where images are stored (an external object storage, provider to be chosen) and how they are backed up.
