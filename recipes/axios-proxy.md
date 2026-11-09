---
title: "Axios proxy setup for Node.js"
description: "Configure Axios to make an HTTPS request through an authenticated HTTP proxy. This Node.js example uses an explicit CONNECT agent, bounds the response and "
---

# Axios proxy setup for Node.js

Configure Axios to make an HTTPS request through an authenticated HTTP proxy. This Node.js example uses an explicit CONNECT agent, bounds the response and waiting time, and reports proxy and destination failures separately.

## Install the tested packages

*From the section "Use the Node HTTP adapter and one agent".*

```sh
npm install --save-exact axios@1.20.0 https-proxy-agent@9.1.0
node --version
npm ls axios https-proxy-agent
```

## axios-check.mjs

*From the section "Make one bounded HTTPS echo request".*

```javascript
import axios from 'axios';
import { HttpsProxyAgent } from 'https-proxy-agent';
import { isIP } from 'node:net';

function required(name) {
  const value = process.env[name];
  if (!value) throw new Error('Missing runtime setting');
  return value;
}

async function main() {
  const proxy = new URL(process.env.PROXY_SERVER ?? 'http://gw.portproof.org:7000');
  if (proxy.protocol !== 'http:' || !proxy.hostname || proxy.username ||
      proxy.password || proxy.pathname !== '/' || proxy.search || proxy.hash) {
    throw new Error('Invalid gateway URL');
  }
  const username = required('PROXY_USER');
  if (username.includes(':')) throw new Error('Invalid Basic username');
  proxy.username = encodeURIComponent(username);
  proxy.password = encodeURIComponent(required('PROXY_PASSWORD'));

  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), 20_000);
  const agent = new HttpsProxyAgent(proxy, {
    keepAlive: false, maxSockets: 1, signal: controller.signal,
  });
  let connectStatus;
  agent.on('proxyConnect', (response) => { connectStatus = response.statusCode; });
  try {
    const response = await axios.get('https://api.portproof.org/v1/echo-ip', {
      adapter: 'http', httpsAgent: agent, proxy: false,
      signal: controller.signal, timeout: 10_000,
      maxRedirects: 0, maxContentLength: 65_536, maxBodyLength: 0,
      responseType: 'text', transformResponse: [(body) => body],
      validateStatus: () => true,
    });
    if (connectStatus !== 200) {
      throw new Error('Proxy CONNECT status ' + connectStatus);
    }
    if (response.status !== 200) {
      throw new Error('Destination HTTP status ' + response.status);
    }
    const payload = JSON.parse(response.data);
    if (!payload || typeof payload.ip !== 'string' || !isIP(payload.ip)) {
      throw new Error('Invalid echo JSON');
    }
    console.log('HTTP', response.status, 'exit', payload.ip);
  } catch (error) {
    if (connectStatus && connectStatus !== 200) {
      console.error('Proxy CONNECT status', connectStatus);
    } else if (controller.signal.aborted) {
      console.error('Request deadline exceeded');
    } else if (axios.isAxiosError(error)) {
      console.error('Request failed:', error.code ?? 'AXIOS_ERROR');
    } else {
      console.error(error instanceof SyntaxError ? 'Invalid echo JSON' : error.message);
    }
    process.exitCode = 1;
  } finally {
    clearTimeout(timer);
    controller.abort();
    agent.destroy();
  }
}

main().catch(() => {
  console.error('Check gateway and runtime credential settings');
  process.exitCode = 1;
});
```

---

The explanation, failure modes and checks behind these snippets are in the full guide: [Axios proxy setup for Node.js](https://portproof.org/guides/axios-proxy).

[← All recipes](../)