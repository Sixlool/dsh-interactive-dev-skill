---
name: dsh-interactive-dev-skill
description: 'Interactive development workflow: use structured multiple-choice questions (ask_user_question) to let users decide requirements, then document first, implement, and visually verify before shipping.'
description-zh: '交互式开发工作流（选择题需求 → 文档先行 → 视觉预览验证）'
---

# DeepSeek Harness 交互式开发工作流 Skill

> 把「让用户做选择题」贯穿始终的 DSH 插件 / 功能开发方法论。
> 源自一次完整的桌宠插件实战（DSH Desktop Pet）：分阶段用结构化选择题收敛需求，先文档后实现，视觉预览验证后再实装。

## 何时使用

开发 DSH 插件（动态 Cordis 插件）、自定义功能，或任何**需求模糊、需要用户拍板**的开发任务。
核心特征：用户有想法但细节未定 → 用结构化选择题快速收敛，而不是开放式追问。

## 核心理念

1. **一次只问一组题**（`ask_user_question`），每组 ≤ 5 题，每题 3–4 个选项
2. **每题第一项为推荐项**，标注 `(Recommended)`；描述写清 tradeoff，让用户 10 秒内能选
3. **只有真正需要才开多选**（multi_select）；不需要用户选技术方案（Host/Client、Slot）——那是实现细节，由 agent 决定
4. **文档先行**：需求基线 → 架构设计 → 逐条对照实现，避免边做边猜
5. **先预览后实装**：图形/UI 类交付物必须用无头浏览器渲染预览，视觉确认后才写进插件

## 工作流

### Phase 0 — 需求收集（选择题五问）

维度模板（按项目替换）：

| 维度 | 问题示例 | 选项示例 |
|---|---|---|
| 外观/形象 | 用什么画风？ | SVG 自绘 / 立绘抠图 / 抽象图形 |
| 交互行为 | 需要哪些交互？(多选) | 拖拽 / 点击互动 / 对话气泡 / 状态联动 |
| 状态联动 | 要不要感知运行状态？ | 感知关键状态 / 感知全部 / 纯装饰 |
| 持久化记忆 | 开关与状态怎么记？ | 默认开启+记住上次 / 不记忆 |
| 呈现位置 | 默认显示在哪？ | 右下角可拖走 / 固定角落 / 跟随内容 |

**产出**：需求基线文档 `requirements.md`（编号 R1..Rn + 待实现阶段确认的技术点）。

### Phase 1 — 架构设计（先查契约，再写设计）

1. `cordis_inspect_list` 列出全部 Provider（Host/Client 各端）
2. **只查要用的契约**：
   - `Slots.listSubTree` — 找 UI 挂载点（overlay / sidebar.footer.action / settings…），确认注册协议
   - `Event.listEvents` — 找状态信号（agent/status、workflow/start|end…），确认 payload
   - `Service.listService` / `Builtin.listBuiltins` — 确认可用能力与全局（尤其 Client 端：**没有 window/localStorage**）
3. 写设计文档 `design.md`：平台分工表（Host/Client）、Slot 选择与理由、事件信号映射、RPC 契约、持久化方案
4. 未定项 → **决策点提问**（一次一个决策组，每组 ≤ 2 题，如：持久化深度 A/B、按钮位置 A/B/C）

### Phase 2 — 实现（动态 Cordis 插件）

- `cordis_define`：新插件 `kind: new`（idPrefix 3–6 小写字母）；改版 `kind: existing` 追加**不可变包**
- 纯 JavaScript：无 JSX / TS / import；`React.createElement`；`inject` 声明硬依赖；`ctx.get` 读可选服务
- 生命周期可逆：`ctx.on` / `ctx.effect` / `ctx.timeout`、`slots.inject` 包裹注册；stop/update 自动清理
- `cordis_run`：首次 `run`、切换 `update`；`awaiting-approval` 需用户在 UI 批准，批准前不要重试

### Phase 3 — 视觉预览验证（图形/UI 必做）

1. 把设计画成静态 `preview.html`（SVG / 立绘 / 布局，含各状态并排）
2. `vision_html_screenshot` 无头渲染成 PNG
3. `vision_describe` 逐项检查：比例 / 重叠 / 遮挡 / 错乱 / 美观
4. 修改 → 重渲染 → 复查，直到 OK
5. **满意才实装**（cordis_define + update）

> 这步是"改完还是丑"的克星：先让 AI 和用户看到效果，再进代码。

### Phase 4 — 修复迭代

- 运行失败：`cordis_inspect_self(pluginId, packageId)` 读源码 + 诊断
- 修复 = 追加新包 → `update`；**不要覆盖旧包**
- 先定位再改：分不清原因时给 Client 加诊断输出（错误打进浏览器 console / 显示占位），让用户反馈错误信息

### Phase 5 — 发布（GitHub）

1. 整理成标准仓库：`package.json`（host/client 入口声明）+ `lib/`（源码）+ `res/`（资源）+ `README.md` + `LICENSE`
2. `git init` → commit → `gh repo create --source . --push`（先创建 remote，再单独 push 以便带 git TLS 参数）

## 提问规范细节

- `ask_user_question`：`id` 稳定可回显；`question` 具体到用户能直接答；`options` 3–4 个，推荐项放第一并标 `(Recommended)`，`description` 一句话讲清代价
- 材料索取：明确让用户**拖图上传**（沙箱常无法下载外链图片）；收到图片后用 `vision_describe` 确认内容再开工

## 环境注意事项（本机实测）

- **Windows schannel 在沙箱不可用**（`SEC_E_NO_CREDENTIALS`）→ HTTPS 出站改用 node / OpenSSL；git 需 `-c http.sslBackend=openssl` 或 `GIT_CONFIG_GLOBAL` 指向自定义 config
- 直连 `github.com` / `registry.npmjs.org` 可行；`raw.githubusercontent.com` 被重置 → socks5 代理（如 `127.0.0.1:1088`）
- `gh` CLI（Go TLS）正常；git 认证用 token URL 或 `store --file=...`（系统 Git Credential Manager 在沙箱会崩）
- 上传的图片附件落在**工作区根目录**；vision 产物在 `.dsh-vision-router/artifacts/`
- 动态插件进程级临时：完整重启 DSH 后需重新 `cordis_run`

## 常见坑速查表（实战提炼）

| 现象 | 原因 | 修法 |
|---|---|---|
| 图片 data URI 损坏（立绘不显示） | `btoa` 按 UTF-8 语义编码二进制 | 纯 JS 字节级 base64（逐 3 字节位移） |
| 拖拽没反应 | 位置换算依赖单个父元素 rect | 多来源视口测量 + 像素位移预览兜底 + Pointer/Mouse 双事件链 |
| 状态图错乱（思考时显示出错图） | `agent/error` 事件覆盖 thinking | thinking 优先级最高（Host 快照 + Client mood 双重保险） |
| Client 无 window/localStorage | 受限环境只注入 ctx/React/host/styles/console | 持久化走 Host 内存或 fs |
| git 认证 helper 崩溃 | 系统 GCM 交互在沙箱不可用 | token URL 或 store helper，`-c credential.helper=` 禁用 GCM |

## 验证清单

- [ ] 需求基线 vs 实现逐条对照
- [ ] 视觉预览通过（vision_describe 复查 OK）
- [ ] `cordis_inspect_self` 无运行诊断错误
- [ ] 交互实测：拖拽 / 点击 / 开关 / 刷新后状态记忆
- [ ] 发布仓库可克隆（README 描述真实可复现）

## 参考案例

- 本 skill 的实战来源：`Sixlool/dsh-desktop-pet`（GitHub）
- 提问与预览的完整示例见本 skill 的 `examples/` 目录（与 SKILL.md 同目录）
