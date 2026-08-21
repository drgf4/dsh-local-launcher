# dsh-local-launcher · 黑色小鲸鱼启动器

> A local launcher for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) — the **black whale launcher**.
> Double-click the black whale icon on the desktop and your default browser opens the DeepSeek Harness Web GUI automatically.

English | [中文](README.zh.md)

## What is this?

`dsh-local-launcher` is a local deployment of [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`) packaged as a one-click launcher, featuring the **black whale** icon.

DeepSeek Harness is an open-source agent harness developed by [DeepSeek AI](https://deepseek.com). It uses an architecture where **everything is a plugin**, powered by [Cordis](https://github.com/cordiverse/cordis).

## Features

- 🐋 **Black whale launcher** — desktop shortcut with the black whale icon (`assets/dsh-whale.ico`)
- ⚡ **Auto-open browser** — after launch, the Web GUI opens in your default browser automatically at `http://127.0.0.1:3080`
- 🧩 Full DeepSeek Harness source (upstream: [`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness), MIT)

## Quick start

### Prerequisites

- [Node.js](https://nodejs.org) 18+ (with `pnpm`)

### Run from source

```sh
git clone https://github.com/drgf4/dsh-local-launcher
cd dsh-local-launcher
pnpm install
pnpm run build
pnpm dsh web
```

The Web UI is served at `http://127.0.0.1:3080` by default.

### Or run from npm (no checkout needed)

```sh
npx @deepseek-ai/dsh web
```

### Desktop launcher with auto-open browser (Windows)

The desktop black whale shortcut points to a small wrapper script that starts the server and opens the browser once it is ready:

```js
// launch-dsh-web.cjs — point your shortcut at this wrapper (adjust DSH_ROOT)
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

> The shortcut should run `node "path\to\launch-dsh-web.cjs"` with the working directory set to the checkout root.

## Repository layout

- `apps/` — CLI and web apps
- `packages/` — all DeepSeek Harness packages (plugin architecture)
- `assets/` — black whale icons used by the launcher
- `docs/` — documentation

## License

[MIT](LICENSE) — upstream [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) is MIT-licensed; third-party notices are disclosed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Upstream

This repository is a local launcher distribution of [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness). Not an official DeepSeek AI project.
