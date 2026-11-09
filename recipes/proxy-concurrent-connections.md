---
title: "Proxy concurrency: size your parallel connections"
description: "Size parallel work from a measured task, the destination’s permitted request rate and the limits on your client and proxy account. Workers, requests per se"
---

# Proxy concurrency: size your parallel connections

Size parallel work from a measured task, the destination’s permitted request rate and the limits on your client and proxy account. Workers, requests per second, open connections and sticky sessions count different things. Increasing one does not guarantee more useful results.

## An explicit HTTPX connection-pool configuration

*From the section "Set client limits without confusing them with permission".*

```python
import httpx

limits = httpx.Limits(
    max_connections=2,
    max_keepalive_connections=2,
)
timeout = httpx.Timeout(10.0, pool=2.0)

# Configuration only: no requests are sent by this example.
# A real worker also needs destination pacing, a bounded queue,
# a whole-task deadline, response limits and correct proxy settings.
with httpx.Client(limits=limits, timeout=timeout) as client:
    pass

```

Two is an illustrative local pool cap, not an account allowance, service recommendation or guaranteed requests-per-second value. Use the HTTPX proxy guide when adding a gateway.

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Proxy concurrency: size your parallel connections](https://portproof.org/guides/proxy-concurrent-connections).

[← All recipes](../)