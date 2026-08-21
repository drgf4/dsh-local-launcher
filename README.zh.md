# dsh-local-launcher · 黑色小鲸鱼启动器

> [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的本地启动器 —— **黑色小鲸鱼启动器**。
> 双击桌面上的黑色小鲸鱼图标，默认浏览器会自动打开 DeepSeek Harness Web GUI。

[English](README.md) | 中文

## 这是什么？

`dsh-local-launcher` 是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（`dsh`）的本地部署 + 一键启动器封装，配**黑色小鲸鱼**图标。

DeepSeek Harness 是 [DeepSeek AI](https://deepseek.com) 开发的开源 agent harness（智能体框架），采用**一切皆插件**的架构，由 [Cordis](https://github.com/cordiverse/cordis) 驱动。

## 特性

- 🐋 **黑色小鲸鱼启动器** —— 桌面快捷方式带黑色小鲸鱼图标（`assets/dsh-whale.ico`）
- ⚡ **启动自动跳转浏览器** —— 启动后默认浏览器自动打开 Web GUI（`http://127.0.0.1:3080`）
- 🧩 DeepSeek Harness 完整源码（上游：[`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness)，MIT）

## 快速开始

### 环境要求

- [Node.js](https://nodejs.org) 18+（含 `pnpm`）

### 从源码运行

```sh
git clone https://github.com/drgf4/dsh-local-launcher
cd dsh-local-launcher
pnpm install
pnpm run build
pnpm dsh web
```

默认 Web UI 地址为 `http://127.0.0.1:3080`。

### 或直接用 npm 运行（无需克隆）

```sh
npx @deepseek-ai/dsh web
```

### 桌面启动器 + 自动打开浏览器（Windows）

桌面黑色小鲸鱼快捷方式指向一个小包装脚本：先启动服务，等服务就绪后自动用默认浏览器打开页面：

```js
// launch-dsh-web.cjs —— 把快捷方式指向这个包装脚本（按实际路径改 DSH_ROOT）
const { spawn } = require('node:child_process');
const http = require('node:http');
const path = require('node:path');

const DSH_ROOT = 'C:\\path\\to\\dsh-local-launcher';
const BIN = path.join(DSH_ROOT, 'apps', 'cli', 'lib', 'bin.js');
const FALLBACK_URL = 'http://127.0.0.1:3080';

let opened = false;
const child = spawn(process.execPath, [BIN, 'web'], { cwd: DSH_ROOT, stdio: ['inherit', 'pipe', 'inherit'] });

let buffered = '';
child.stdout.on('data', (chunk) => {
  const text = chunk.toString('utf8');
  try { process.stdout.write(text); } catch {}
  if (!opened) {
    buffered += text;
    const m = buffered.match(/https?:\/\/127\.0\.0\.1:\d+/);
    if (m) { opened = true; spawn('cmd', ['/c', 'start', '', m[0]], { stdio: 'ignore', detached: true }).unref(); }
  }
});

const timer = setInterval(() => {
  if (opened) return;
  const req = http.get(FALLBACK_URL, (res) => {
    res.resume();
    opened = true;
    spawn('cmd', ['/c', 'start', '', FALLBACK_URL], { stdio: 'ignore', detached: true }).unref();
  });
  req.on('error', () => {});
  req.setTimeout(1500, () => req.destroy());
}, 1000);

child.on('exit', (code, signal) => { clearInterval(timer); process.exitCode = code ?? (signal ? 1 : 0); });
```

> 快捷方式应运行 `node "路径\to\launch-dsh-web.cjs"`，并将工作目录设为仓库根目录。

## 目录结构

- `apps/` —— CLI 与 Web 应用
- `packages/` —— DeepSeek Harness 全部包（插件化架构）
- `assets/` —— 启动器用的黑色小鲸鱼图标
- `docs/` —— 文档

## 许可证

[MIT](LICENSE) —— 上游 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 为 MIT 协议，第三方依赖及其许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

## 上游

本仓库是 [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) 的本地启动器分发，非 DeepSeek AI 官方项目。
