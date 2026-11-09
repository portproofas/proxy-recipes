---
title: "PHP cURL proxy setup with authentication"
description: "Send an HTTPS request through an HTTP proxy in PHP. Keep the proxy password out of saved commands, verify the destination certificate and separate a reject"
---

# PHP cURL proxy setup with authentication

Send an HTTPS request through an HTTP proxy in PHP. Keep the proxy password out of saved commands, verify the destination certificate and separate a rejected tunnel from an HTTP error returned by the destination.

## Inspect PHP and its cURL library

*From the section "Check the PHP runtime and connection fields".*

```sh
php --version
php -r 'echo extension_loaded("curl") ? curl_version()["version"] . PHP_EOL : "curl extension is missing\n";'
```

## proxy-check.php

*From the section "Make one bounded HTTPS request".*

```php
<?php
declare(strict_types=1);

function fail(string $message): never {
    fwrite(STDERR, $message . PHP_EOL);
    exit(1);
}

if (PHP_SAPI !== 'cli' || !extension_loaded('curl')) {
    fail('Use PHP CLI with the curl extension enabled.');
}
$userLine = fgets(STDIN);
$passwordLine = fgets(STDIN);
$user = $userLine === false ? '' : rtrim($userLine, "\r\n");
$password = $passwordLine === false ? '' : rtrim($passwordLine, "\r\n");
if ($user === '' || $password === '') fail('Missing proxy credentials.');

$proxy = getenv('PROXY_SERVER') ?: 'http://gw.portproof.org:7000';
$url = getenv('PROXY_CHECK_URL') ?: 'https://api.portproof.org/v1/echo-ip';
$proxyParts = parse_url($proxy);
if (!is_array($proxyParts) || ($proxyParts['scheme'] ?? '') !== 'http'
    || empty($proxyParts['host']) || !isset($proxyParts['port'])
    || isset($proxyParts['user']) || isset($proxyParts['pass'])
    || isset($proxyParts['query']) || isset($proxyParts['fragment'])
    || !in_array($proxyParts['path'] ?? '', ['', '/'], true)) {
    fail('Use an HTTP proxy URL with host and port, without credentials or a path.');
}
$urlParts = parse_url($url);
if (!is_array($urlParts) || ($urlParts['scheme'] ?? '') !== 'https'
    || empty($urlParts['host']) || isset($urlParts['user']) || isset($urlParts['pass'])) {
    fail('Use an HTTPS check destination without URL credentials.');
}
$body = '';
$tooLarge = false;
$ch = curl_init($url);
if ($ch === false) fail('Could not initialise cURL.');
$configured = curl_setopt_array($ch, [
    CURLOPT_PROXY => $proxy,
    CURLOPT_PROXYTYPE => CURLPROXY_HTTP,
    CURLOPT_HTTPPROXYTUNNEL => true,
    CURLOPT_PROXYAUTH => CURLAUTH_BASIC,
    CURLOPT_PROXYUSERPWD => rawurlencode($user) . ':' . rawurlencode($password),
    CURLOPT_NOPROXY => '',
    CURLOPT_SSL_VERIFYPEER => true,
    CURLOPT_SSL_VERIFYHOST => 2,
    CURLOPT_CONNECTTIMEOUT => 10,
    CURLOPT_TIMEOUT => 30,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_WRITEFUNCTION => static function ($handle, string $chunk) use (&$body, &$tooLarge): int {
        if (strlen($body) + strlen($chunk) > 65_536) {
            $tooLarge = true;
            return 0;
        }
        $body .= $chunk;
        return strlen($chunk);
    },
]);
unset($user, $password, $userLine, $passwordLine);
if (!$configured) fail('Could not configure cURL.');
$caFile = getenv('PROXY_CA_FILE');
if ($caFile && !curl_setopt($ch, CURLOPT_CAINFO, $caFile)) {
    fail('Could not configure the trusted CA file.');
}
$ok = curl_exec($ch);
$error = curl_errno($ch);
$connect = (int) curl_getinfo($ch, CURLINFO_HTTP_CONNECTCODE);
$status = (int) curl_getinfo($ch, CURLINFO_HTTP_CODE);
unset($ch);

if ($connect === 407) fail('Proxy authentication rejected (CONNECT 407).');
if ($tooLarge) fail('Echo response exceeded 65,536 bytes.');
if ($ok === false) fail("Transfer failed: cURL $error; CONNECT $connect; HTTP $status.");
if ($connect !== 200) fail("Unexpected CONNECT status $connect.");
if ($status !== 200) fail("Destination returned HTTP $status.");
try {
    $data = json_decode($body, true, 16, JSON_THROW_ON_ERROR);
} catch (JsonException $error) {
    fail('Echo response was not valid JSON.');
}
$ip = is_array($data) ? ($data['ip'] ?? null) : null;
if (!is_string($ip) || filter_var($ip, FILTER_VALIDATE_IP) === false) {
    fail('Echo response did not contain a valid IP address.');
}
printf("HTTP %d; exit %s\n", $status, $ip);
```

## run-proxy-check.sh

*From the section "Enter the password without putting it in a command".*

```sh
#!/usr/bin/env bash
set +x
set -euo pipefail
unset proxy_user proxy_password
trap 'unset proxy_user proxy_password' EXIT
IFS= read -r -p 'Full proxy username: ' proxy_user </dev/tty
IFS= read -r -s -p 'Proxy password (hidden): ' proxy_password </dev/tty
printf '\n' >/dev/tty
printf '%s\n' "$proxy_user" "$proxy_password" | php proxy-check.php
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [PHP cURL proxy setup with authentication](https://portproof.org/guides/php-curl-proxy).

[← All recipes](../)