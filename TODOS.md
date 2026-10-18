# TODOS

## Derive item status from item_events; drop the denormalized status column
**What:** Remove `items.status` (and `claimer_id`'s dual-write responsibility); derive current status and claimer from the latest row in `item_events` (`type ∈ found_snap|claim|restoration_note|sold`).

**Why:** The claim/note/sold writes currently update both `items` and `item_events` — two sources of truth for one fact. The events table was chosen precisely to be the timeline of record; the denormalized column exists only because it was faster for the hackathon demo. It will drift the first time a write path forgets the second update.

**Pros:** Single source of truth; timeline and list badges can never disagree; simpler write paths (insert one row).

**Cons:** Every status read becomes a latest-event query (add an index on `(item_id, created_at desc)`); one migration + touching every read site; ~1-2 hours.

**Context:** Demo shipped 2026-10-18 with the dual-write as an accepted shortcut. Start by writing the derive-status query, verifying it matches `items.status` on the seeded data, then flip reads, then drop the column.

**Depends on / blocked by:** Nothing — do it any time after the hackathon.
