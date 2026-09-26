# 浏览器本机网络延迟测速工具 ⚡

[English](README.md) | [简体中文](README_CN.md)

一款轻量、零依赖、纯浏览器本地发起的 HTTP/HTTPS 网络延迟基准测试工具。用于精准测量从当前电脑网络栈（或代理客户端如 Clash、Sing-box、V2Ray、TUN 模式）到任意目标网站或 API 接口的**真实往返时间（RTT）**。

与传统的第三方云端测速网站（由远程机房代测）或系统 ICMP Ping（仅测试底层 IP 连通性，不包含 TLS 握手与真实 HTTP 握手）不同，**Browser Latency Tester** 测出的是您在浏览器中冲浪或调用 API 时的**真实体感延迟**。

---

## 🌟 核心特性

- **100% 本地发起**：直接在您的当前浏览器内发起请求，严格遵循您电脑当前的代理软件分流规则（DIRECT 直连 / 节点代理 / TUN 虚拟网卡）。
- **零依赖单文件**：纯 HTML5 与原生 JavaScript 编写，无需安装 Node.js、npm 或任何外部库，双击即用。
- **免 CORS 跨域探针**：利用 `fetch(url, { mode: 'no-cors' })` 配合时间戳防缓存机制，直接探测任何公开 HTTPS/HTTP 网址，不会被浏览器的同源策略拦截。
- **高精度微秒级计时**：采用浏览器底层高精度计时器 `performance.now()`，确保毫秒级计算的准确性。
- **一键快捷预设分类**：
  - **⚡ 204 空包极速探针**：直接测试 Cloudflare 与 Google 的 `generate_204` 心跳点（**这正是 Clash / Mihomo 节点测速使用的底层逻辑**），排除服务器网页渲染耗时，专门用于反映纯物理网络通路的延迟。
  - **🚀 主流 AI 与开发者 API 端点**：预置全球知名公网 API（如 `api.openai.com`、`api.anthropic.com`、`generativelanguage.googleapis.com`、`api.github.com` 等），一键排查接口连通性。
  - **🌐 完整网页首包测试**：测试访问 Google 搜索首页或普通门户的真实完整耗时（包含 TLS 1.3 加密握手与服务器后端处理生成 HTML 的时间）。
- **多轮测试统计聚合**：自动跑 5 轮基准测试，实时计算并显示：**当前延迟**、**最低延迟 (Min)**、**平均延迟 (Avg)**、**最高延迟 (Max)** 及每轮详细日志。

---

## 📐 为什么同一个节点，不同测速差异巨大？

### 1. 204 空包探针 vs 完整网页的区别
- **204 空包探针（Clash / 代理软件测速同款）**：
  - 目标地址如：`http(s)://cp.cloudflare.com/generate_204`。
  - 边缘服务器直接返回一个 `204 No Content` 空包（响应体大小为 0 字节，服务器 0 毫秒处理耗时）。
  - **反映的是纯网络专线通道的物理传输时间**（例如专线通常为 20~30ms）。
- **完整网页访问（如 Google 搜索首页）**：
  - 必须进行完整的 **TLS 1.3 安全加密握手**（密钥交换与证书校验，需增加 1~2 个网络往返 RTT）。
  - Google 服务器接收到请求后，需要**后台执行检索、计算 Cookie、排版生成 HTML 网页代码**（通常需要 100~150ms 的服务器端处理时间 TTFB）。
  - 因此：`底层专线 (30ms) + 加密握手 (60ms) + 谷歌后端计算 (140ms) ≈ 230ms`，这是访问一个复杂动态网站非常健康且符合物理规律的正常时间。

---

## 🚀 快速上手

### 方式 1：在线直接使用（免安装）
直接在任意浏览器打开 GitHub Pages 托管页面：
👉 **[https://wsl5406.github.io/browser-latency-tester/](https://wsl5406.github.io/browser-latency-tester/)**

### 方式 2：本地双击运行
克隆仓库后，直接双击 `index.html`：
```bash
git clone https://github.com/wsl5406/browser-latency-tester.git
cd browser-latency-tester
# Windows 直接打开：
start index.html
```

### 方式 3：静态服务器托管
```bash
# 使用 Python
python -m http.server 8080

# 使用 Node.js npx
npx serve .
```
然后在浏览器访问 `http://localhost:8080` 即可。

---

## 🛠️ 实现原理

页面核心探测逻辑如下：

```javascript
const target = new URL(url);
// 动态追加随机时间戳，防止浏览器读取本地 HTTP 缓存
target.searchParams.set('_t', Date.now());

const t0 = performance.now();
await fetch(target.toString(), {
  method: 'GET',
  mode: 'no-cors',
  cache: 'no-store'
});
// 计算从发起请求到收到首包响应的时间
const latencyMs = Math.round(performance.now() - t0);
```

---

## 📄 开源许可

本项目遵循 [MIT License](LICENSE) 开源协议。
