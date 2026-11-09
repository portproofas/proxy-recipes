---
title: "Python urllib proxy setup"
description: "To use an urllib proxy, build an opener with ProxyHandler, mapping the destination scheme to the gateway URL, and send requests through that opener. This g"
---

# Python urllib proxy setup

To use an urllib proxy, build an opener with ProxyHandler, mapping the destination scheme to the gateway URL, and send requests through that opener. This guide uses Python’s standard library to send an HTTPS request through an authenticated HTTP proxy. Keep the password out of source code, check routing exceptions and distinguish gateway failures from destination responses.

## proxy_check.py · Python standard library

*From the section "Make one HTTPS request through the HTTP proxy".*

```python
import getpass
import http.client
import ipaddress
import json
import ssl
import warnings
from urllib.error import HTTPError, URLError
from urllib.parse import quote, urlsplit
from urllib.request import (
    HTTPRedirectHandler, HTTPSHandler, ProxyHandler,
    build_opener, proxy_bypass,
)

TARGET = "https://api.portproof.org/v1/echo-ip"
GATEWAY = "gw.portproof.org:7000"

class NoRedirect(HTTPRedirectHandler):
    def redirect_request(self, req, fp, code, msg, headers, newurl):
        return None

def main():
    try:
        # Stop if a terminal cannot hide the password.
        warnings.simplefilter("error", getpass.GetPassWarning)
        username = input("Full proxy username: ").strip()
        password = getpass.getpass("Proxy password: ")
        if not username or not password:
            raise ValueError("Empty credential")
        if proxy_bypass(urlsplit(TARGET).netloc):
            print("A direct-routing exception matches the target. Review NO_PROXY or system proxy exceptions.")
            return 1
        credential = quote(username, safe="") + ":" + quote(password, safe="")
        proxy = "http://" + credential + "@" + GATEWAY
        context = ssl.create_default_context()
        context.set_alpn_protocols(["http/1.1"])
        opener = build_opener(
            ProxyHandler({"https": proxy}),
            HTTPSHandler(context=context),
            NoRedirect(),
        )
        with opener.open(TARGET, timeout=10) as response:
            if response.status != 200:
                raise ValueError("Unexpected response")
            body = response.read(16_385)
        if len(body) > 16_384:
            raise ValueError("Response too large")
        payload = json.loads(body)
        if not isinstance(payload, dict) or not isinstance(payload.get("ip"), str):
            raise ValueError("Missing IP")
        print("HTTP 200; exit IP:", ipaddress.ip_address(payload["ip"]))
        return 0
    except HTTPError as error:
        print("HTTP status:", error.code)
        error.close()
    except getpass.GetPassWarning:
        print("Run from an interactive terminal that supports hidden password input.")
    except (URLError, OSError, ValueError, http.client.HTTPException):
        print("Check failed. Check proxy settings, credentials, TLS and the echo response.")
    except (EOFError, KeyboardInterrupt):
        print("Check cancelled.")
    return 1

if __name__ == "__main__":
    raise SystemExit(main())

```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Python urllib proxy setup](https://portproof.org/guides/python-urllib-proxy).

[← All recipes](../)