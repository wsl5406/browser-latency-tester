# Browser Latency Tester ⚡

[English](README.md) | [简体中文](README_CN.md)

[![Live Demo](https://img.shields.io/badge/Live_Demo-GitHub_Pages-38bdf8?style=flat-square)](https://wsl5406.github.io/browser-latency-tester/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

A lightweight, zero-dependency, browser-native latency benchmark tool designed to measure **real HTTP/HTTPS round-trip time (RTT)** from your local browser environment directly to any web service or API endpoint.

Unlike cloud-based ping services (which test from remote data centers) or ICMP ping (which only tests low-level IP packet responses without TLS negotiation), **Browser Latency Tester** accurately reflects your actual browser browsing experience and respects your local proxy setup (such as Clash, Sing-box, V2Ray, or system TUN mode).

---

## 🌟 Key Features

- **100% Local Execution**: Initiated straight from your local browser stack. What you measure is what you experience.
- **Proxy & TUN Aware**: Seamlessly honors your local system proxy, rules, and TUN network interfaces.
- **Zero Dependencies**: Pure HTML5 and vanilla JavaScript. Runs out of the box with zero build steps or npm installations.
- **CORS Bypass Probe**: Employs `fetch(url, { mode: 'no-cors' })` with cache buster query parameters to probe any arbitrary HTTPS endpoint without triggering CORS blocking.
- **High-Precision Timing**: Leverages the browser `performance.now()` API to ensure sub-millisecond precision.
- **Preset Quick-Switching**:
  - **Fast 204 Empty Probes**: Test against lightweight heartbeat endpoints (e.g., Cloudflare `generate_204`, Google `generate_204`) to isolate raw network transport RTT (the exact mechanism used by proxy client latency tests).
  - **Global AI & Tech APIs**: Fast-check latency to popular AI and developer APIs (e.g., `api.openai.com`, `api.anthropic.com`, `generativelanguage.googleapis.com`, `api.github.com`).
  - **Full Web Pages**: Measure end-to-end user-perceived TTFB (Time to First Byte) including TLS handshake and backend server computation.
- **Statistical Aggregation**: Real-time computation of Current, Min, Max, and Average round-trip times across multi-round benchmarks.

---

## 📐 Understanding the Latency Numbers

### 1. Fast 204 Empty Probe vs. Full Web Page
- **Fast 204 Probe (Clash/Proxy Client Style)**:
  - Hits `http(s)://cp.cloudflare.com/generate_204` or `https://www.gstatic.com/generate_204`.
  - The server immediately returns an empty `204 No Content` response (0 bytes body, zero backend database/render overhead).
  - Measures pure connection latency + edge network speed.
- **Full Web Page (e.g., Google Search Homepage)**:
  - Involves TLS 1.3 cryptographic handshake (+1~2 RTTs).
  - Involves server-side computation, database lookups, template rendering, and cookie management (typically 100~150ms backend processing time).
  - Reflects the true time-to-first-byte (TTFB) experienced when navigating in a browser.

---

## 🚀 Quick Start

### Method 1: Direct File Access
Simply download or clone this repository and double-click `index.html` to open it in Chrome, Edge, Safari, or Firefox:
```bash
git clone https://github.com/wsl5406/browser-latency-tester.git
cd browser-latency-tester
start index.html
```

### Method 2: Host via Any Static Web Server
```bash
# Using Python
python -m http.server 8080

# Using Node.js npx
npx serve .
```
Then visit `http://localhost:8080` in your browser.

---

## 🛠️ How It Works

The core measurement loop utilizes the browser's native `fetch` with `cache: 'no-store'` and dynamic timestamp parameters:

```javascript
const target = new URL(url);
target.searchParams.set('_t', Date.now());

const t0 = performance.now();
await fetch(target.toString(), {
  method: 'GET',
  mode: 'no-cors',
  cache: 'no-store'
});
const latencyMs = Math.round(performance.now() - t0);
```

Because `no-cors` mode allows the request to be dispatched across origins while recording the network round trip upon header arrival, it serves as an effective, client-side HTTP latency probe for any public URL.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
