---
title: "PowerShell proxy setup and authentication"
description: "Send an HTTPS request through an authenticated HTTP proxy from PowerShell 7.6. Use a password prompt, keep destination TLS checks enabled and verify the ec"
---

# PowerShell proxy setup and authentication

Send an HTTPS request through an authenticated HTTP proxy from PowerShell 7.6. Use a password prompt, keep destination TLS checks enabled and verify the echo response before adding the proxy to a larger script.

## proxy-check.ps1 · PowerShell 7.6

*From the section "Send one request with explicit proxy credentials".*

```powershell
$ErrorActionPreference = 'Stop'
$proxy = [uri]'http://gw.portproof.org:7000'
$target = [uri]'https://api.portproof.org/v1/echo-ip'
$credential = Get-Credential -Message 'Enter the full proxy username and proxy password'
try {
    if ($null -eq $credential -or [string]::IsNullOrWhiteSpace($credential.UserName)) {
        throw 'Enter the generated proxy username.'
    }
    $response = Invoke-WebRequest -Uri $target -Method Get -Proxy $proxy -ProxyCredential $credential -ConnectionTimeoutSeconds 10 -OperationTimeoutSeconds 10 -MaximumRedirection 0 -MaximumRetryCount 0 -ErrorAction Stop
    if ($response.StatusCode -ne 200) { throw 'The echo endpoint did not return HTTP 200.' }
    $payload = $response.Content | ConvertFrom-Json -ErrorAction Stop
    $address = $null
    if ($payload.ip -isnot [string] -or -not [System.Net.IPAddress]::TryParse($payload.ip, [ref]$address)) {
        throw 'The echo body did not contain a valid IP address.'
    }
    'HTTP 200; exit IP: ' + $address.ToString()
} catch {
    # Do not print complete exceptions or response bodies into shared logs.
    if ($null -ne $_.Exception.Response) {
        Write-Host ('HTTP status: ' + [int]$_.Exception.Response.StatusCode)
    }
    Write-Host 'Check failed. Check proxy credentials, gateway status, TLS and echo response.'
    exit 1
} finally {
    $credential = $null
}

```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [PowerShell proxy setup and authentication](https://portproof.org/guides/powershell-proxy).

[← All recipes](../)