# 项目长期记忆 · Frametell

## 架构约定（改代码前必读）

- **依赖方向自上而下，无环**：`apps` → `packages/core` / `packages/ui` → `packages/agent` → `packages/engine` / `models` / `voice`。
  `engine` 与 `models` 是叶子包，可独立复用测试。
- **`packages/core` 是唯一的行为定义处**。`Studio` 类的每个方法就是一项客户端能力；
  `apps/cli`（HTTP+SSE）与 `apps/desktop`（IPC）只做协议翻译。新增功能改 core，不要在两个宿主各写一遍。
- **`packages/ui` 只能类型导入 `@frametell/core/types`**，不可引入运行时依赖——
  这样浏览器包不含 Node 代码，传输方式才能运行时切换。
- **`docs/ARCHITECTURE.md` 的"仓库布局"一节已过时**（仍写 `apps/studio` 单应用）。
  现状以 `docs/ARCHITECTURE-MAP.md` 与实际目录为准。
- **阶段常量**以 `packages/agent/src/events.ts` 为准：
  `research | storyboard | script | narrate | pages | review | render`。

## 核心设计理念

- **质量靠工程护栏，不靠模型自觉**：分镜 lint、页面 lint、截图自检、成片门禁都由 Director 强制执行，
  再把结果交给模型修。弱模型也能稳定出片。
- **每个阶段是独立 agent 会话**：上下文短、缓存命中率高，失败只重跑该阶段。
- **每个项目 = 一个 git 仓库**：agent 每步一次 commit，`git revert` 即撤销。
- **本地优先**：只监听 127.0.0.1，密钥存 `~/.frametell/config.json`（0600），日志与事件中一律脱敏。

## 调试约定

- 改 TS 后必须重新 build（仓库无 watch 脚本）。用 `node --watch apps/cli/dist/cli.js serve --no-open` 自动重启。
- UI 热更新：终端 1 跑 serve，终端 2 `cd packages/ui && npx vite`，访问 `http://localhost:5173/?token=<token>`。
- headless `frametell run` **只读环境变量，不读 config.json**——与面板行为不同，容易踩坑。
- 项目级调试抓手：`git log`（agent 流水）、`.studio/events.jsonl`（全部事件）、
  `engine/tools/{lint,snap,check,pages}.mjs`（可单跑，不必起整条流水线）。
- 环境自检：`packages/core/src/env.ts` 的 `checkEnv()` 查 Chrome / ffmpeg / git / node 四项。

## 用户偏好

- 中文交流。
- 交付偏好：结论先行 + 可直接执行的命令，事实需经核实（已多次要求"实测"而非推测）。
- 涉及密钥时须脱敏，禁止完整打印。
