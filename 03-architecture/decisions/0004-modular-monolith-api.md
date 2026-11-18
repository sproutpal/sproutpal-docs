# 0004. API as a modular monolith with command/query separation

- **Status:** Proposed
- **Date:** 2026-11-18

## Context

- The API (`sproutpal-api`, [ADR-0003](0003-tech-stack.md)) serves the knowledge base, users and course progress; later also garden planning.
- One developer, one VPS, a few thousand documents, and a demo as the next goal.
- The knowledge base is read far more often than it is written, and only one author writes. Writing and reading have different needs: writes need versioning and a `changeNote` ([ADR-0001](0001-knowledge-base-mongodb.md)), reads need fast queries and reverse lists.
- Microservices and CQRS were raised as options.

## Decision

1. **No microservices.** The API is **one deployable service**.
2. Inside, it is a **modular monolith** with clearly separated modules:
   - **KnowledgeBase:** course topics and encyclopedia entries;
   - **Users:** accounts and course progress;
   - later **Planner:** gardens, plots and plantings.

   A module owns its MongoDB collections and its code. Other modules use it only through its public interface, never its collections or internals.
3. **Commands and queries are separated in code** (lightweight CQRS):
   - **commands** change data (for example: save a new version of an entry, record course progress) and enforce the write rules;
   - **queries** only read and return data shaped for the caller (for example: a variety with its references, the varieties of a species).

   Each command and query has its own handler.
4. **One database** serves both commands and queries. There are no separate read models or read databases.

## Reasons

- Microservices solve independent development, deployment and scaling for several teams. With one developer they add repositories, pipelines, deployments, monitoring and network failure handling without a benefit.
- Clear module boundaries keep the code organised now, and a module can later be extracted into its own service if that becomes necessary. Merging services back is much harder.
- Separate command and query handlers match the different needs of writing and reading, keep each handler small and testable, and cost little.
- Full CQRS (separate read stores updated from events) is not needed: one MongoDB serves both sides well at this size. The change stream listener (ADR-0001) already gives an event-based path if read models are ever needed.

## Alternatives considered

- **Microservices** (for example a knowledge base service and a users service): rejected for now, see Reasons.
- **Full CQRS with event sourcing or separate read databases:** rejected as too complex for the size and the team.
- **A plain layered API without module boundaries:** simpler at first, but the boundaries would blur as the planner is added.

## Consequences

- Positive: one service to build, deploy and monitor; clear structure inside it.
- Positive: a module can be extracted later without rewriting it.
- Negative: module boundaries are kept by discipline and code structure, not by the network; they need attention in reviews.
- Negative: all modules scale together and share one failure domain.

## Open points

- Whether to use a mediator library for command and query dispatch, or plain handler classes.
- How module boundaries are checked (for example architecture tests).
