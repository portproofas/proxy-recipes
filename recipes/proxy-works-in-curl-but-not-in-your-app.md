---
title: "HTTPS_PROXY not working? Check your client"
description: "A proxy that works in curl can still be ignored by your application. Check the process that sends the request: which variables it received, whether its HTT"
---

# HTTPS_PROXY not working? Check your client

A proxy that works in curl can still be ignored by your application. Check the process that sends the request: which variables it received, whether its HTTP client reads them, and whether a host exclusion or an explicit setting takes precedence.

## Match the variable to the destination

*From the section "Match the variable to the destination".*

```sh
export https_proxy='http://gw.portproof.org:7000'
```

## Match the variable to the destination

*From the section "Match the variable to the destination".*

```python
import os

for lower in ("http_proxy", "https_proxy", "all_proxy", "no_proxy"):
    upper = lower.upper()
    low = os.environ.get(lower)
    high = os.environ.get(upper)
    print(f"{lower}: {'set' if low else 'unset or empty'}")
    print(f"{upper}: {'set' if high else 'unset or empty'}")
    if low is not None and high is not None and low != high:
        print(f"{lower}/{upper}: values differ")
```

## Set the client that actually opens the connection

*From the section "Set the client that actually opens the connection".*

```sh
# HTTPS_PROXY is supplied through your existing secret configuration.
NODE_USE_ENV_PROXY=1 node app.mjs
```

## Put the settings inside the worker

*From the section "Put the settings inside the worker".*

```sh
# worker-image is your application's existing image.
docker run --rm \
  --env HTTPS_PROXY \
  --env NO_PROXY \
  worker-image
```

## Compare an explicit curl check with the application

*From the section "Compare an explicit curl check with the application".*

```sh
curl --disable --silent --show-error \
  --proxy 'http://gw.portproof.org:7000' \
  --proxy-user 'YOUR_FULL_PROXY_USERNAME' \
  --noproxy '' \
  --connect-timeout 5 --max-time 15 \
  --output /dev/null \
  --write-out 'connect_status=%{http_connect} target_status=%{response_code}\n' \
  'https://example.com/'
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [HTTPS_PROXY not working? Check your client](https://portproof.org/guides/proxy-works-in-curl-but-not-in-your-app).

[← All recipes](../)