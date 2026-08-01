# Backpressure

Backpressure is the propagation of capacity constraints toward producers so demand can slow, shed, or reshape itself before queues and latency grow without bound.

## Why it matters

An overloaded system without feedback often fails slowly, then suddenly: queues grow, deadlines expire, retries multiply traffic, and work continues after callers no longer care. Backpressure makes saturation an explicit part of the protocol.

## Common mechanisms

- Bounded queues that reject or block new work
- Pull-based consumption or explicit demand signals
- Concurrency limits and admission control
- Rate limits and quotas
- Load shedding based on priority or deadline
- Windowing and acknowledgements in network protocols

## Tradeoffs

Backpressure does not create capacity. It decides how scarcity becomes visible and who absorbs it. Blocking can spread resource exhaustion upstream; rejection requires clients to behave responsibly; buffering only delays the decision. The end-to-end design must pair the signal with deadlines, retry policy, prioritization, and user-facing degradation.

## Questions

- Where is the first bounded resource?
- Can producers observe saturation before timeouts?
- Is rejected work safe to retry, and under what budget?
- Which requests should be protected during overload?
- Does cancellation stop downstream work?

## Connections

- [[Architecture Review]] should identify where load accumulates and how capacity signals propagate.
- [[Incident Learning]] can reveal overload amplification that was absent from the original design model.

## Sources

No sources filed yet. This note currently contains preliminary synthesis.
