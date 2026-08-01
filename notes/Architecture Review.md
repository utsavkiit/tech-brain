# Architecture Review

An architecture review should improve the decision, reveal assumptions, and align owners—not reward document theater or transfer accountability to a committee.

## Before the review

- State the desired outcome and explicit non-goals.
- Name decision owners, stakeholders, and the deadline.
- Separate hard constraints from preferences.
- Document alternatives, including “do less” and “delay.”
- Identify invariants and expensive-to-reverse choices.

## Review lenses

### Behavior and data

- What are the consistency and ordering requirements?
- Which data is authoritative, derived, sensitive, or erasable?
- How do schema and API changes remain compatible?

### Scale and failure

- Where are the bottlenecks and bounded resources?
- How is [[Backpressure]] handled?
- What are the partial-failure, recovery, and rollback paths?
- What is the blast radius of each dependency?

### Operations and evolution

- How will user-impacting behavior be observed?
- Who owns the system across team boundaries?
- What is the migration path and safe intermediate state?
- Which assumptions will be validated first?

## Output

Record the decision, unanswered risks, predicted outcomes, and how the important assumptions will be tested. A later [[Incident Learning]] review can compare actual system behavior with this design model.

## Sources

No sources filed yet. These heuristics should be treated as preliminary synthesis or experience until supported.
