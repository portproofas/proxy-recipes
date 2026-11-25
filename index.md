---
title: Proxy recipes
---

# Proxy recipes

Short, tested snippets for sending requests through an authenticated HTTP proxy. Each one sets explicit timeouts, keeps credentials out of the command line where the client allows it, and tells a proxy failure apart from a destination failure.

The examples use the [Portproof](https://portproof.org) gateway host and its echo endpoint, so you can run them as written with an account. To use another provider, replace the host, port and username. Nothing else in the snippets is provider-specific.

## Clients

- [Axios proxy setup for Node.js](recipes/axios-proxy) — [full guide](https://portproof.org/guides/axios-proxy)
- [Go HTTP proxy setup with authentication](recipes/go-http-proxy) — [full guide](https://portproof.org/guides/go-http-proxy)
- [PHP cURL proxy setup with authentication](recipes/php-curl-proxy) — [full guide](https://portproof.org/guides/php-curl-proxy)
- [PowerShell proxy setup and authentication](recipes/powershell-proxy) — [full guide](https://portproof.org/guides/powershell-proxy)
- [Python urllib proxy setup](recipes/python-urllib-proxy) — [full guide](https://portproof.org/guides/python-urllib-proxy)
- [Wget proxy setup with authentication](recipes/wget-proxy) — [full guide](https://portproof.org/guides/wget-proxy)
- [HTTPX vs Requests: choose and migrate](recipes/httpx-vs-requests) — [full guide](https://portproof.org/guides/httpx-vs-requests)
- [HTTPS_PROXY not working? Check your client](recipes/proxy-works-in-curl-but-not-in-your-app) — [full guide](https://portproof.org/guides/proxy-works-in-curl-but-not-in-your-app)
- [Proxy format: URLs, list formats and ports](recipes/proxy-url-format) — [full guide](https://portproof.org/guides/proxy-url-format)
- [Fix CONNECT tunnel failed: 403, 407, 502 and 503](recipes/connect-tunnel-failed) — [full guide](https://portproof.org/guides/connect-tunnel-failed)
- [HTTP 429: identify the limit and retry carefully](recipes/429-too-many-requests) — [full guide](https://portproof.org/guides/429-too-many-requests)
- [Proxy concurrency: size your parallel connections](recipes/proxy-concurrent-connections) — [full guide](https://portproof.org/guides/proxy-concurrent-connections)
- [Sticky proxy sessions: rotation and duration](recipes/rotating-vs-sticky-proxy-sessions) — [full guide](https://portproof.org/guides/rotating-vs-sticky-proxy-sessions)

## Conventions

- `YOUR_FULL_PROXY_USERNAME` and the `PROXY_USER` variable stand for the full proxy username from your dashboard. The password comes from an environment variable or a prompt, never from shell history.
- `https://api.portproof.org/v1/echo-ip` returns the IP address it sees (your exit IP) as JSON. Any echo service works the same way.
- Port 7000 is HTTP CONNECT and port 7001 is SOCKS5. The snippets use port 7000.

Corrections and new clients are welcome as [issues or pull requests](https://github.com/portproofas/proxy-recipes).
