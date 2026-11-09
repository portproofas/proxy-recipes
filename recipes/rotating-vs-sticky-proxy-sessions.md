---
title: "Sticky proxy sessions: rotation and duration"
description: "Use a saved sticky configuration to request reuse of an available binding for related work. It does not reserve an exit device or keep its public IP fixed."
---

# Sticky proxy sessions: rotation and duration

Use a saved sticky configuration to request reuse of an available binding for related work. It does not reserve an exit device or keep its public IP fixed. Timed modes are evaluated on later connections using the saved SID; device availability and the network still determine whether that route can continue.

## Create one session name for one job

*From the section "Keep one name for one job".*

```python
from uuid import uuid4

session_name = "qa_" + uuid4().hex
connection_settings = {
    "pool": "residential",
    "country": "any",
    "rotation": "sticky",
    "session": session_name,
    "protocol": "http",
}
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Sticky proxy sessions: rotation and duration](https://portproof.org/guides/rotating-vs-sticky-proxy-sessions).

[← All recipes](../)