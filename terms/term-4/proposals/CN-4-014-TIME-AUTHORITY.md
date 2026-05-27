# CN-4-014 — Time Authority

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine — extends to tz-database versioning for time zone determinism).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine — determinism), CN-4-002 (Event Store Contract — timestamp field, global_position for ordering), CN-4-006 (Isolation — fiscal zone as tenant property), CN-4-009 (Replay — ReplayClock).
> **Doctrine checks:** DC-017 (no `datetime.now()`), DC-018 (all time from clock protocol) — CN-4-019.
> **Boundaries:** Clock API design → Architect phase; replay mechanics → CN-4-009; event envelope → CN-4-002; tenant properties → CN-4-006; simulation time → CN-4-022 (advisor layer).

---

## 1. What This Doc Defines

CN-4-014 defines how BOS handles time deterministically. `datetime.now()` is forbidden in engine, primitive, fold, handler, and policy logic (Charter doctrine, DC-017/DC-018). This doc defines the clock protocol that replaces it, how fiscal time zones work, how time zone database changes are handled, and how time stays replay-safe.

---

## 2. The Clock Protocol

### The Problem

If engine logic calls `datetime.now()`, replay on a different day produces different state. A projection rebuilt on Monday shows different totals than one rebuilt on Tuesday. Determinism breaks. Legal defensibility breaks.

### The Solution: Three Injected Clocks

All time in event-producing logic comes from an **injected clock** — never from the system clock directly. The Kernel provides exactly three clock types:

| Clock type | When used | What it provides |
|-----------|----------|-----------------|
| **SystemClock** | Runtime (production) | Current wall-clock time. The only place where real time enters the system. |
| **FixedClock** | Tests | A fixed, configurable time. Tests are deterministic regardless of when they run. |
| **ReplayClock** | Replay (CN-4-009) | Time from event timestamps. Replay sees the time the event was originally recorded, not the actual current time. |

These are the only three Kernel clock types. There is no SimulationClock — what-if and profit simulation are advisor-layer concerns (CN-4-022, Law 3): they operate outside the event store, never write events, and may use hypothetical time internally without affecting Kernel determinism.

### The Rule

No engine, primitive, fold function, handler, or policy evaluator may access system time directly. All time is obtained through the clock protocol. The clock is injected at the boundary (like the tenant scope object, CN-4-006) and propagated through all layers — non-bypassable.

### Scope of the Clock Rule

The deterministic clock rule applies to all **replay-sensitive logic**: fold functions, projection handlers, policy evaluators, and any code that produces events. These must use the injected clock and must never depend on wall-clock time.

**Read-only queries** that ask "as of now" (e.g., a dashboard showing "current inventory") legitimately use SystemClock — but they are reads against projections, never event-producing logic. They do not affect the event stream or replay determinism.

DC-017 and DC-018 (CN-4-019) enforce this: static analysis detects `datetime.now()` or equivalent in replay-sensitive code paths; any use blocks the build.

---

## 3. Two Kinds of Time

| Kind | What it is | Where it lives | Example |
|------|-----------|---------------|---------|
| **Record time** | When the event was recorded in the store | Event envelope: `timestamp` field (CN-4-002) | "This sale event was recorded at 14:32:07" |
| **Business time** | When the business action occurred or takes effect | Event payload (engine-specific) | "This sale's effective date is 2026-03-31" (end-of-day batch) |

Record time is assigned by the SystemClock at event emission. Business time is declared by the engine in the payload. They may differ — a batch job processing end-of-day sales at 01:00 records business_date = yesterday, record_timestamp = today at 01:00.

Both are deterministic on replay: record time comes from the stored timestamp; business time comes from the stored payload. Neither depends on the current wall clock.

### Timestamps Are Not for Ordering

Record-time timestamps are **descriptive, not authoritative for ordering**. Ordering and determinism are provided by `global_position` (CN-4-002), never by timestamp monotonicity. Clock skew, NTP adjustments, and leap seconds mean timestamps may not be perfectly monotonic across events. This is acceptable — `global_position` is the total order; timestamps are metadata. This aligns with CN-4-009 §4 (point-in-time cutoff = global_position, timestamp resolved to position).

---

## 4. Fiscal Time Zone

### The Problem

A business in East Africa closes its day at a local time. An event arrives at 23:59:59 local. On replay, "which day does this belong to?" must have a deterministic answer.

### The Solution: Anchored at Registration, Version-Frozen

A tenant's fiscal time zone is **set at registration and stored as an event** (CN-4-006 tenant properties).

| Property | Detail |
|----------|--------|
| **Fiscal zone name** | IANA time zone identifier (e.g., `Africa/Dar_es_Salaam`) — stored as the zone name. |
| **Resolved UTC offset at registration** | The UTC offset active at the time of registration is stored alongside the zone name. |
| **Tz-database version** | The version of the IANA tz-database active at registration is recorded. |

### Tz-Database Version Freeze (D-009 Analog)

The IANA tz-database changes over time — countries modify DST rules, UTC offsets shift. If BOS naively looks up `Africa/Dar_es_Salaam` using the current tz-database, historical day boundaries may shift retroactively. This breaks determinism.

The resolution follows the same principle as D-009 (freeze doctrine for compliance packs):

**Historical events and tenant properties are interpreted under the tz-database version that was active at the time.** A later tz-database update applies only to new events and new tenant registrations.

Each event's day-boundary computation uses:
1. The tenant's registered fiscal zone name, AND
2. The tz-database version recorded at tenant registration (or at the most recent fiscal-zone-change event, if the tenant relocated).

Alternatively, the resolved UTC offset stored at registration provides a deterministic fallback even if the tz-database version is unavailable during replay.

This ensures: replay today, using today's tz-database, produces the same day boundaries as replay last year — because the historical tz-database version is referenced, not the current one.

### Fiscal Zone Change

If a business changes its fiscal zone (rare — e.g., relocation), a new event records the change with the new zone name, resolved offset, and the tz-database version at the time of change. Events before the change use the old zone; events after use the new zone. Freeze doctrine applies.

### Neutral by Design

The clock protocol and fiscal zone mechanism do not name any specific time zone as a constant. "Africa/Dar_es_Salaam" is a runtime value, not a concept-level constant. The Kernel knows "a tenant has a fiscal zone"; it does not know which zones exist or what their offsets are.

---

## 5. Time in Replay

During replay (CN-4-009):

| What happens | How time is handled |
|-------------|-------------------|
| **Fold/handler processes an event** | Time comes from the event's `timestamp` (record time) or payload fields (business time). The ReplayClock provides the event's timestamp. |
| **Logic needs "current time"** | The ReplayClock returns the timestamp of the event being processed — not the actual current time. |
| **Day boundary calculation** | Uses the tenant's fiscal zone as it was at the time of the event (zone name + tz-database version, per freeze). |
| **Time comparison** | Always between stored times — never against wall clock. |

This is what makes replay deterministic: time is data (stored in events and tenant properties), not environment (read from a clock or a database).

---

## 6. Time in the Command Bus

When the command bus (CN-4-004) processes a command and emits events:

1. The **SystemClock** provides the current time.
2. The `timestamp` field on the emitted event is set from the SystemClock.
3. The `pack_version_ref` is resolved at this time (D-009 freeze).
4. Business time (if any) is provided by the command submitter in the payload.
5. The current tz-database version is the one active in the running system.

The bus never uses ambient time — it uses the injected SystemClock. Tests can inject a FixedClock and verify command processing at any simulated time.

---

## 7. Three Whys

### Why does this matter?

If time is ambient (read from the system clock or the current tz-database), replay produces different state on different days or after a tz-database update. A projection rebuilt after a DST rule change shows different day boundaries than the original. The numbers change silently. Legal defensibility is destroyed — a court cannot trust figures that shift depending on when they are recalculated.

### Why does it belong in the Kernel?

Every engine uses time — for day boundaries, deadlines, aging, period logic. If each engine managed its own clock, one engine's `datetime.now()` would break replay for the entire system. The clock protocol is Kernel-level because time determinism must be system-wide — one violation anywhere breaks the guarantee everywhere.

### Why this design?

Injected clocks (SystemClock/FixedClock/ReplayClock) are the standard pattern for deterministic systems. The fiscal-zone-at-registration design with tz-database version freeze makes day boundaries a tenant property (data), not a server property (environment). Both choices eliminate ambient time from all logic paths. The alternative — "be careful with time" as a developer guideline — is unenforceable and will fail.

---

## 8. Example — End-of-Day Batch at Hoteli ya Bahari

A hotel in Zanzibar closes its business day at 23:00 local time. The system runs an end-of-day aggregation.

1. The scheduler (`system:scheduler`, CN-4-007) submits a command at 23:05 local time.
2. The SystemClock provides the current time: 2026-03-31T20:05:00Z.
3. The event's `timestamp` = 2026-03-31T20:05:00Z (record time).
4. The event's payload contains `business_date = 2026-03-31` (the day being closed).
5. The fiscal zone for this tenant is `Africa/Dar_es_Salaam` (UTC+3), registered with tz-database version 2025f, resolved offset +03:00.

Six months later, the IANA tz-database has been updated (hypothetically: East Africa shifted to UTC+2 in version 2026c). A projection rebuild replays this event:

- The ReplayClock provides 2026-03-31T20:05:00Z (from the stored timestamp).
- The handler reads `business_date = 2026-03-31` from the payload.
- Day boundary is computed from the tenant's fiscal zone using **tz-database version 2025f** (the version at registration) — not version 2026c. The offset is still +03:00 for this historical event.
- The result is identical to the original processing. No ambient time was used. No tz-database change affected the historical computation.

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| Event envelope timestamp field | CN-4-002 |
| Global_position (authoritative ordering) | CN-4-002 |
| Replay mechanics (when ReplayClock is used) | CN-4-009 |
| Freeze doctrine (pack versions; extends to tz-database versions) | D-009 |
| Doctrine checks DC-017/DC-018 | CN-4-019 |
| Tenant registration properties (where fiscal zone is stored) | CN-4-006 |
| Command bus (where SystemClock is used at emission) | CN-4-004 |
| Simulation/what-if time (advisor layer, not Kernel) | CN-4-022 |
| Clock API design (interface, injection mechanism) | Architect phase |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Clock API design (interface for SystemClock/FixedClock/ReplayClock) | Architect phase | Concept defines the three types and injection; Architect designs the API |
| Clock injection mechanism (dependency injection, context propagation) | Architect phase | Must be non-bypassable |
| Tz-database version storage and retrieval during replay | Architect phase | Concept says "version recorded at registration, used during replay"; Architect designs storage and lookup |
| Leap seconds / NTP synchronisation precision | Architect phase | Timestamps are descriptive; precision is infrastructure |
| Fiscal zone change event schema | CN-4-006 + Architect | The conceptual rules are defined here; the event schema is Architect's |
| **Platform cross-tenant time reference zone** | Term 1 + Architect | Platform-scope time queries (cross-tenant aggregation) must specify a reference zone explicitly (e.g., UTC or a platform-defined zone). Cross-tenant aggregation cannot assume a single fiscal day — tenants may have different fiscal zones. This is a governance/policy concern for Term 1 and a design concern for the Architect. |

---

*— End of CN-4-014 —*
