---
title: "HTTP 429: identify the limit and retry carefully"
description: "A 429 response asks a client to slow down. First identify the service and request stage, then read its retry instructions and reduce the work generating re"
---

# HTTP 429: identify the limit and retry carefully

A 429 response asks a client to slow down. First identify the service and request stage, then read its retry instructions and reduce the work generating requests. Keep any retry within the service’s rules and your job’s attempt and time budgets.

## A bounded Retry-After decision

*From the section "Test the wait decision without making a request".*

```python
import math
import re
from datetime import datetime, timezone
from email.utils import format_datetime, parsedate_to_datetime


def retry_delay(value, *, now, remaining_seconds):
    """Return a permitted wait, or None to stop and review."""
    if now.tzinfo is None or not math.isfinite(remaining_seconds):
        raise ValueError("Use an aware clock and a finite time budget")
    if remaining_seconds < 0 or value is None:
        return None
    value = value.strip()
    if re.fullmatch(r"[0-9]+", value):
        # An enormous delay cannot fit this small-job example.
        if len(value) > 18:
            return None
        delay = int(value)
    else:
        try:
            date = parsedate_to_datetime(value)
            if date.tzinfo is None:
                return None
            date = date.astimezone(timezone.utc)
            # Accept the preferred HTTP date form; review other formats.
            if format_datetime(date, usegmt=True) != value:
                return None
            delay = max(0, math.ceil((date - now).total_seconds()))
        except (TypeError, ValueError, OverflowError):
            return None
    # Never shorten a server's wait to fit our job deadline.
    return delay if delay <= remaining_seconds else None


# Fictional response value and job budget; no network request is made.
wait = retry_delay("120", now=datetime.now(timezone.utc),
                   remaining_seconds=300)
print("Stop and review" if wait is None else f"Wait at least {wait}s")
```

Fictional 120-second wait and 300-second job budget. The function never turns an excessive wait into a shorter one. Keep automatic retries off when it returns None.

---

The explanation, failure modes and checks behind these snippets are in the full guide: [HTTP 429: identify the limit and retry carefully](https://portproof.org/guides/429-too-many-requests).

[← All recipes](../)