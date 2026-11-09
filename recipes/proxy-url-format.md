---
title: "Proxy format: URLs, list formats and ports"
description: "Proxies are shared as URLs such as http://user:pass@host:port or as list lines such as host:port:user:pass. Both carry the same four fields: protocol, host"
---

# Proxy format: URLs, list formats and ports

Proxies are shared as URLs such as `http://user:pass@host:port` or as list lines such as `host:port:user:pass`. Both carry the same four fields: protocol, host, port and credentials. Start from the original connection values, encode credential components once when they go inside a URL, and keep raw credentials for clients with separate authentication fields.

## Proxy URL shape — placeholders only

*From the section "Identify the four fields before joining them".*

```text
http://ENCODED_USER:ENCODED_PASSWORD@gw.portproof.org:7000
```

This shows structure, not a usable credential. A complete credential-bearing URL is a secret; keep it out of screenshots, logs and source control.

## Offline serialization check — synthetic values only

*From the section "Encode raw username and password components once".*

```python
from urllib.parse import quote, unquote, urlsplit

raw_user = "user:name"
raw_password = "p@ss/#:%40 +"
host = "gw.portproof.org"
port = 7000

proxy_url = (
    f"http://{quote(raw_user, safe='')}:"
    f"{quote(raw_password, safe='')}@{host}:{port}"
)
parts = urlsplit(proxy_url)
assert unquote(parts.username) == raw_user
assert unquote(parts.password) == raw_password
assert parts.hostname == host
assert parts.port == port
# Pass proxy_url privately to a client that accepts this format.
# Do not print a URL assembled from real credentials.

```

This checks string construction and a local parse round trip. It sends no network request and does not establish that a gateway or particular client accepted the credentials.

## Separate fields for a supported Playwright proxy configuration

*From the section "Keep separate authentication fields raw".*

```javascript
// Configuration fragment; credentials are original values from your environment.
const proxy = {
  server: "http://gw.portproof.org:7000",
  username: process.env.PROXY_USER,
  password: process.env.PROXY_PASS,
};

```

Set and validate both environment values before creating the browser. Environment storage is not permission to log the object. See the full Playwright guide for a bounded request and cleanup.

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Proxy format: URLs, list formats and ports](https://portproof.org/guides/proxy-url-format).

[← All recipes](../)