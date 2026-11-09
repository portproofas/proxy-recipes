---
title: "Go HTTP proxy setup with authentication"
description: "Use Go’s net/http client to send an HTTPS request through an authenticated HTTP proxy. The example keeps credentials out of destination headers, preserves "
---

# Go HTTP proxy setup with authentication

Use Go’s net/http client to send an HTTPS request through an authenticated HTTP proxy. The example keeps credentials out of destination headers, preserves TLS verification and puts explicit limits on connection time and response reading.

## proxy-check.go

*From the section "Use an explicit transport for one echo request".*

```go
package main

import (
	"context"
	"crypto/tls"
	"crypto/x509"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"net"
	"net/http"
	"net/url"
	"os"
	"strings"
	"time"
)

func check() error {
	server := os.Getenv("PROXY_SERVER")
	if server == "" {
		server = "http://gw.portproof.org:7000"
	}
	proxy, err := url.Parse(server)
	if err != nil || proxy.Scheme != "http" || proxy.Hostname() == "" ||
		proxy.User != nil || (proxy.Path != "" && proxy.Path != "/") ||
		proxy.RawQuery != "" || proxy.Fragment != "" {
		return errors.New("use an HTTP gateway URL without credentials or a path")
	}
	user, password := os.Getenv("PROXY_USER"), os.Getenv("PROXY_PASSWORD")
	if user == "" || password == "" || strings.Contains(user, ":") {
		return errors.New("check runtime proxy credentials; username cannot contain a colon")
	}
	proxy.User = url.UserPassword(user, password)

	transport := http.DefaultTransport.(*http.Transport).Clone()
	transport.Proxy = http.ProxyURL(proxy)
	transport.DialContext = (&net.Dialer{Timeout: 5 * time.Second}).DialContext
	transport.TLSHandshakeTimeout = 5 * time.Second
	transport.ResponseHeaderTimeout = 10 * time.Second
	transport.MaxConnsPerHost = 1
	transport.MaxIdleConns = 1
	transport.MaxResponseHeaderBytes = 16 << 10
	transport.OnProxyConnectResponse = func(_ context.Context, _ *url.URL,
		_ *http.Request, response *http.Response) error {
		if response.StatusCode != http.StatusOK {
			return fmt.Errorf("proxy CONNECT status %d", response.StatusCode)
		}
		return nil
	}
	// Optional approved CA file; default certificate verification stays enabled.
	if file := os.Getenv("PROXY_CA_BUNDLE"); file != "" {
		pem, err := os.ReadFile(file)
		if err != nil {
			return errors.New("cannot read approved CA bundle")
		}
		roots, err := x509.SystemCertPool()
		if err != nil {
			return errors.New("cannot load system trust store")
		}
		if !roots.AppendCertsFromPEM(pem) {
			return errors.New("invalid CA bundle")
		}
		transport.TLSClientConfig = &tls.Config{RootCAs: roots}
	}
	defer transport.CloseIdleConnections()
	client := &http.Client{
		Transport: transport, Timeout: 20 * time.Second,
		CheckRedirect: func(_ *http.Request, _ []*http.Request) error {
			return http.ErrUseLastResponse
		},
	}
	request, err := http.NewRequest(http.MethodGet, "https://api.portproof.org/v1/echo-ip", nil)
	if err != nil {
		return errors.New("invalid echo URL")
	}
	response, err := client.Do(request)
	if err != nil {
		var certificateError *tls.CertificateVerificationError
		switch {
		case errors.As(err, &certificateError):
			return errors.New("destination TLS certificate verification failed")
		case errors.Is(err, context.DeadlineExceeded):
			return errors.New("request deadline exceeded")
		default:
			// url.Error may contain URLs: expose only a recognised CONNECT status.
			var wrapped *url.Error
			if errors.As(err, &wrapped) && strings.HasPrefix(wrapped.Err.Error(), "proxy CONNECT status ") {
				return wrapped.Err
			}
			return errors.New("connection failed; check gateway, DNS and timeout settings")
		}
	}
	defer response.Body.Close()
	if response.StatusCode != http.StatusOK {
		return fmt.Errorf("destination HTTP status %d", response.StatusCode)
	}
	const limit = 64 << 10
	body, err := io.ReadAll(io.LimitReader(response.Body, limit+1))
	if err != nil {
		return errors.New("response read failed or exceeded its deadline")
	}
	if len(body) > limit {
		return errors.New("echo response exceeds 64 KiB")
	}
	var payload struct {
		IP string `json:"ip"`
	}
	if json.Unmarshal(body, &payload) != nil || net.ParseIP(payload.IP) == nil {
		return errors.New("invalid echo JSON")
	}
	fmt.Println("HTTP", response.StatusCode, "exit", payload.IP)
	return nil
}

func main() {
	if err := check(); err != nil {
		fmt.Fprintln(os.Stderr, "Check failed:", err)
		os.Exit(1)
	}
}
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Go HTTP proxy setup with authentication](https://portproof.org/guides/go-http-proxy).

[← All recipes](../)