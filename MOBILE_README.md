# 在手机端运行 — 说明与操作指南

本文档说明我为本项目所做的修改、为何要做、以及如何在手机或其他移动设备上访问和使用该 3D 圣诞树应用。

---

## 我为你做了哪些修改（代码变更概述）

- 自动检测移动设备（`src/App.tsx` 中使用 `navigator.userAgent` 判断）。
- 在移动端降低渲染负载：
  - `foliage`（树叶粒子）从 15000 降到 2500。
  - `ornaments`（拍立得照片）从 300 降到 80。
  - `elements`（礼物/糖果等）降到 50，`lights` 降到 100。
- 根据设备调整 Canvas 像素比（DPR）：移动端使用 `dpr=1`，桌面端保留 `[1,2]`。
- 在移动端禁用昂贵的后期处理（`EffectComposer` / Bloom / Vignette）。
- 降低粒子光点（Sparkles）数量（移动端 150，桌面端 600）。
- 微调 `OrbitControls` 的 `rotateSpeed` 以改善触摸体验。
- 在 `package.json` 中添加脚本 `dev:host`，方便在局域网内绑定并由手机访问：
  - `npm run dev:host` 等同 `vite --host`。
- 注：我也临时注释掉了默认 `Environment preset="night"`（若网络环境不能加载外部 HDR 文件，会导致页面报错/空白）。如果你想恢复 HDR 渲染，可把 HDR 下载到 `public/hdri/` 并修改为本地引用（说明见下）。

---

## 如何在手机上实时访问（开发调试）

以下为最常用的两种方法，按需选择：

### 方法 A — 局域网访问（同一 Wi‑Fi）

1. 在 Windows 开启终端（PowerShell），在项目根目录运行：

```powershell
npm.cmd run dev -- --host
# 或使用脚本
npm run dev:host
```

2. 终端会输出 `Network` 地址，例如 `http://192.168.1.50:5173/`。
3. 在手机（连接同一 Wi‑Fi）打开该地址即可查看页面。注意：手机在 HTTP 上通常无法使用摄像头（getUserMedia），因为大多数移动浏览器要求 HTTPS 才能访问摄像头（localhost 是例外）。
4. 如果手机无法访问：
   - 检查 PC 与手机是否在同一局域网。使用 `ipconfig`（PC）确认 IP。
   - 若被防火墙阻止，临时在管理员 PowerShell 中允许 5173 端口：

```powershell
New-NetFirewallRule -DisplayName "Allow Vite 5173" -Direction Inbound -Action Allow -LocalPort 5173 -Protocol TCP
# 测试完成后删除
Remove-NetFirewallRule -DisplayName "Allow Vite 5173"
```

### 方法 B — 使用 ngrok（推荐用于摄像头 / HTTPS）

1. 在 https://ngrok.com/ 注册并下载 ngrok。安装并登录（按 ngrok 官方说明）。
2. 在项目根运行（终端）：

```powershell
ngrok http 5173
```

3. ngrok 会输出一个 `https://xxxxxx.ngrok.io` 地址，在手机上打开该 HTTPS 地址即可——摄像头权限通常在 HTTPS 下可用。

---

## 如何恢复 HDR（可选：如果你想要更高级的环境光）

1. 在 `public` 下创建 `hdri` 文件夹（项目根）：

```powershell
mkdir .\public\hdri
```

2. 下载 HDR 到该目录（示例）：

```powershell
Invoke-WebRequest -Uri "https://cdn.jsdelivr.net/gh/pmndrs/drei-assets@456060a26bbeb8fdf79326f224b6d99b8bcce736/hdri/dikhololo_night_1k.hdr" -OutFile .\public\hdri\dikhololo_night_1k.hdr
```

3. 在 `src/App.tsx` 中把 Environment 行改为：

```tsx
<Environment files="/hdri/dikhololo_night_1k.hdr" background={false} />
```

这会从本地加载 HDR，避免依赖外部 CDN 导致超时或 CORS 问题。

---

## 推荐的移动端配置（如果你希望默认以移动优先）

在 `src/App.tsx` 中 `CONFIG.counts` 你可以按需调整：

```ts
counts: {
  foliage: 2000,
  ornaments: 60,
  elements: 40,
  lights: 80
}
```

并在 Canvas 中强制使用 `dpr={1}`，以及在运行时禁用 `EffectComposer`。

---

## 使用者快速入门（一步到位）

1. 克隆并进入项目目录。
2. 安装依赖：

```powershell
npm install
```

3. 局域网访问（简单）或使用 ngrok（支持摄像头/HTTPS）：

- 局域网：
```powershell
npm run dev:host
# 终端查看 Network 地址并在手机浏览器打开
```

- ngrok（获取 HTTPS 并允许摄像头）：
```powershell
npm run dev:host
# 在另一个终端运行
ngrok http 5173
# 在手机上打开 ngrok 给出的 https 地址
```

4. 推荐手机测试顺序：
   - 打开页面，确认能看到画面。
   - 允许摄像头权限（若使用 MediaPipe 手势功能）并观察右下角 `DEBUG`（或页面上的状态）提示。
   - 若卡顿：返回 PC 停止服务器，降低 `CONFIG.counts` 数值并重启。

---

## 我可以帮你进一步做的事（可选）

- 我可以把 HDR 自动下载并恢复 Environment 为本地引用（如果你允许我尝试下载）。
- 我可以把一个额外的 `mobile`/`prod-mobile` 构建配置添加到 `vite`，以输出更轻量的资源包。 
- 我可以把一个开关 UI 加入页面，让你在运行时切换“移动模式/高质量模式”。

如果你希望我继续（例如：自动下载 HDR 并恢复，或添加开关），告诉我我会继续操作。祝试验顺利！
