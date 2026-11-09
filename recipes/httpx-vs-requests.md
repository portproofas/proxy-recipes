---
title: "HTTPX vs Requests: choose and migrate"
description: "Choose a Python HTTP client for the application you need to run, then preserve its request policy. Redirects, timeout phases, environment routing and error"
---

# HTTPX vs Requests: choose and migrate

Choose a Python HTTP client for the application you need to run, then preserve its request policy. Redirects, timeout phases, environment routing and error handling can change when a working Requests integration moves to HTTPX.

## Install the versions used in the local fixture

*From the section "Use the same small response contract in both clients".*

```sh
python -m pip install "requests==2.34.2" "httpx==0.28.1"
```

## comparison_check.py — one controlled read with either client

*From the section "Use the same small response contract in both clients".*

```python
import json

import httpx
import requests


def read_result(response, chunks):
    status = response.status_code
    if status != 200:
        return {"status": status, "accepted": False, "reason": "status"}
    body = bytearray()
    for chunk in chunks:
        if len(body) + len(chunk) > 65536:
            raise ValueError("Response exceeds 64 KiB")
        body.extend(chunk)
    payload = json.loads(body)
    accepted = isinstance(payload, dict) and payload.get("ready") is True
    return {"status": status, "accepted": accepted,
            "reason": "accepted" if accepted else "content"}


def requests_check(url):
    status = None
    try:
        with requests.Session() as client:
            client.trust_env = False
            with client.get(url, timeout=(2, 3), allow_redirects=False,
                            stream=True, verify=True) as response:
                status = response.status_code
                return read_result(response, response.iter_content(8192))
    except requests.RequestException as error:
        return {"status": status, "accepted": False,
                "reason": type(error).__name__}
    except ValueError:
        return {"status": status, "accepted": False, "reason": "ValueError"}


def httpx_check(url):
    status = None
    try:
        with httpx.Client(trust_env=False, follow_redirects=False,
                          verify=True,
                          timeout=httpx.Timeout(3, connect=2, pool=1)) as client:
            with client.stream("GET", url) as response:
                status = response.status_code
                return read_result(response, response.iter_bytes(8192))
    except httpx.HTTPError as error:
        return {"status": status, "accepted": False,
                "reason": type(error).__name__}
    except ValueError:
        return {"status": status, "accepted": False, "reason": "ValueError"}
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [HTTPX vs Requests: choose and migrate](https://portproof.org/guides/httpx-vs-requests).

[← All recipes](../)