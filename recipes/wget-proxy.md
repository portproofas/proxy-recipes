---
title: "Wget proxy setup with authentication"
description: "Configure GNU Wget 1 to use an authenticated HTTP proxy, check an HTTPS destination and keep the password out of command history. Start with one bounded ec"
---

# Wget proxy setup with authentication

Configure GNU Wget 1 to use an authenticated HTTP proxy, check an HTTPS destination and keep the password out of command history. Start with one bounded echo request before downloading a permitted dataset.

## Check the installed executable

*From the section "Check that you are running GNU Wget 1".*

```sh
wget --no-config --version
```

## wget_check.py

*From the section "Run one bounded echo check".*

```python
import getpass
import ipaddress
import json
import os
from pathlib import Path
import selectors
import shutil
import subprocess
import tempfile
import time
import warnings
from urllib.parse import urlsplit


class CheckFailure(Exception):
    pass


def check():
    wget = shutil.which("wget")
    if not wget:
        raise CheckFailure("Install GNU Wget 1 and check your PATH")
    version = subprocess.run([wget, "--no-config", "--version"],
                             capture_output=True, text=True, timeout=5)
    if not version.stdout.startswith("GNU Wget 1."):
        raise CheckFailure("This example requires GNU Wget 1")
    server = os.environ.get("PROXY_SERVER", "http://gw.portproof.org:7000")
    parsed = urlsplit(server)
    if (parsed.scheme != "http" or not parsed.hostname or not parsed.port
            or parsed.username is not None or parsed.password is not None
            or parsed.path not in ("", "/") or parsed.query or parsed.fragment
            or any(ord(c) <= 32 or ord(c) == 127 for c in server)):
        raise CheckFailure("Use an HTTP gateway URL without credentials")
    user = os.environ.get("PROXY_USER") or input("Full proxy username: ")
    warnings.simplefilter("error", getpass.GetPassWarning)
    password = getpass.getpass("Proxy password: ")
    for value in (user, password):
        if (not value or value != value.strip()
                or any(ord(c) < 32 or ord(c) == 127 for c in value)):
            raise CheckFailure("This config method requires nonempty credentials without edge whitespace or controls")
    if ":" in user:
        raise CheckFailure("A Basic-auth username cannot contain a colon")
    env = {k: v for k, v in os.environ.items()
           if k.lower() not in {"http_proxy", "https_proxy", "ftp_proxy",
                                "all_proxy", "no_proxy", "wgetrc"}}
    with tempfile.TemporaryDirectory(prefix="wget-check-") as folder:
        config = Path(folder) / "wgetrc"
        fd = os.open(config, os.O_WRONLY | os.O_CREAT | os.O_EXCL, 0o600)
        with os.fdopen(fd, "w", encoding="utf-8") as stream:
            stream.write(f"use_proxy = on\nhttp_proxy = {server}\nhttps_proxy = {server}\n"
                         f"no_proxy =\nproxy_user = {user}\nproxy_password = {password}\n")
        command = [wget, f"--config={config}", "--quiet", "--no-netrc",
                   "--no-hsts", "--no-cookies", "--tries=1", "--timeout=5",
                   "--max-redirect=0", "--output-document=-"]
        if os.environ.get("PROXY_CA_BUNDLE"):
            command.append("--ca-certificate=" + os.environ["PROXY_CA_BUNDLE"])
        command.append("https://api.portproof.org/v1/echo-ip")
        with subprocess.Popen(command, stdout=subprocess.PIPE,
                              stderr=subprocess.DEVNULL, env=env) as process:
            deadline = time.monotonic() + 20
            data = bytearray()
            try:
                with selectors.DefaultSelector() as ready:
                    ready.register(process.stdout, selectors.EVENT_READ)
                    while True:
                        left = deadline - time.monotonic()
                        if left <= 0 or not ready.select(left):
                            raise CheckFailure("Overall download deadline reached")
                        chunk = os.read(process.stdout.fileno(), 4096)
                        if not chunk:
                            break
                        data.extend(chunk)
                        if len(data) > 65536:
                            raise CheckFailure("Echo body exceeds 64 KiB")
                process.wait(timeout=max(0.001, deadline - time.monotonic()))
                if process.returncode:
                    raise CheckFailure(f"Wget exited with status {process.returncode}")
            finally:
                if process.poll() is None:
                    process.kill()
                process.wait()
        payload = json.loads(data)
        if not isinstance(payload, dict) or not isinstance(payload.get("ip"), str):
            raise ValueError("Echo response has no IP string")
        address = ipaddress.ip_address(payload["ip"])
        print("Exit address:", address)


if __name__ == "__main__":
    try:
        check()
    except CheckFailure as error:
        print(error)
        raise SystemExit(1)
    except (OSError, ValueError, subprocess.SubprocessError, getpass.GetPassWarning) as error:
        print("Check failed:", type(error).__name__)
        raise SystemExit(1)

```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Wget proxy setup with authentication](https://portproof.org/guides/wget-proxy).

[← All recipes](../)