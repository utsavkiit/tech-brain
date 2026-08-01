# Incident Learning

The purpose of incident review is to improve the system's capacity to succeed under real conditions, not to produce a perfect retrospective narrative.

## Capture

- User impact and duration
- Detection and response timeline
- System conditions and contributing interactions
- What responders believed at each decision point
- Defenses that worked, failed, or were absent
- Recovery actions that created additional risk

## Analyze

Avoid a single “root cause” when multiple conditions were necessary. Ask why actions made sense locally, how the system permitted the failure to grow, and where signals arrived too late. Include ownership, incentives, documentation, and workload when they shaped the outcome.

## Convert to durable learning

- Fix immediate hazards, but also improve detection, containment, and recovery.
- Prefer systemic controls over reminders to “be careful.”
- Validate that corrective actions changed risk.
- Update related notes, such as [[Backpressure]], when an incident changes the system model.
- Feed invalidated assumptions back into [[Architecture Review]] and [[Engineering Strategy]].

## Quality test

Would someone unfamiliar with the incident make a better design or response decision after reading the review? If not, the artifact records history but has not yet created learning.

## Sources

No sources filed yet. These heuristics should be treated as preliminary synthesis or experience until supported.
