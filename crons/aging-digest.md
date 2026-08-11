Aging digest: surface my stalled items.

STEPS:
1. {BOARD}: items assigned to me with no activity (no comment, no status
   change) in {AGING_DAYS, default 5} days.
2. For each: title, days stalled, last activity, and ONE recommended
   action — do it now (if <15 min), schedule it, delegate it, or close it
   as no longer relevant.
3. {VAULT}: tasks tagged #task older than 14 days with no updates.

OUTPUT: table sorted by days stalled, descending. End with: "Reply with
item numbers + do/schedule/delegate/close and I'll act." If nothing is
stalled, deliver "Nothing aging. Clean board." and stop.
