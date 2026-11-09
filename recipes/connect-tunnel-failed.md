---
title: "Fix CONNECT tunnel failed: 403, 407, 502 and 503"
description: "CONNECT tunnel failed, response 502 means the proxy returned 502 while your client was trying to open a tunnel. A 503 points to a service-unavailable respo"
---

# Fix CONNECT tunnel failed: 403, 407, 502 and 503

CONNECT tunnel failed, response 502 means the proxy returned 502 while your client was trying to open a tunnel. A 503 points to a service-unavailable response at that stage; a 403 means the CONNECT request was refused, and a 407 means the proxy wants valid credentials first. The number alone does not identify the failing service or tell you whether another attempt will work.

## Record CONNECT and destination status

*From the section "Record the tunnel status without exposing the password".*

```sh
curl_status=0
curl --disable --silent --show-error \
  --proxy 'http://gw.portproof.org:7000' \
  --proxy-user 'YOUR_FULL_PROXY_USERNAME' \
  --noproxy '' \
  --connect-timeout 5 --max-time 15 \
  --output /dev/null \
  --write-out 'connect_status=%{http_connect} target_status=%{response_code}\n' \
  'https://example.com/' || curl_status=$?
printf 'curl_exit=%s\n' "$curl_status"
```

## Illustrative diagnostic result

*From the section "Record the tunnel status without exposing the password".*

```text
connect_status=502 target_status=000
curl_exit=56
```

## Illustrative diagnostic result

*From the section "Record the tunnel status without exposing the password".*

```text
connect_status=200 target_status=403
curl_exit=0
```

## Support report without credentials

*From the section "Prepare a redacted support report".*

```text
Checked at (UTC):
Browser name and version:
Browser error shown (exact text):
Browser proxy source: system / PAC / extension / not recorded
curl version:
Proxy protocol, hostname and port (no credentials):
Destination hostname and port (no path, query or tokens):
connect_status:
target_status:
curl_exit:
Certificate error, if any (redacted):
Documented gateway code, if any:
Same intended proxy and destination in both checks: yes / no / not recorded
Small authorised endpoint succeeded: yes / no / not checked
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Fix CONNECT tunnel failed: 403, 407, 502 and 503](https://portproof.org/guides/connect-tunnel-failed).

[← All recipes](../)