# 仓库指南

- 仓库地址：https://github.com/openclaw/openclaw
- GitHub issues/comments/PR comments：使用字面量多行字符串或 `-F - <<'EOF'`（或 $'...'）来输入真正的换行符；永远不要嵌入 "\\n"。

## 项目结构与模块组织

- 源代码：`src/`（CLI 连接在 `src/cli`，命令在 `src/commands`，web provider 在 `src/provider-web.ts`，基础设施在 `src/infra`，媒体管道在 `src/media`）。
- 测试：并置的 `*.test.ts`。
- 文档：`docs/`（图片、队列、Pi 配置）。构建输出存放在 `dist/`。
- 插件/扩展：位于 `extensions/*`（工作区包）。将插件专用依赖项保留在扩展的 `package.json` 中；除非核心使用，否则不要将它们添加到根目录的 `package.json`。
- 插件：安装时在插件目录运行 `npm install --omit=dev`；运行时依赖项必须位于 `dependencies` 中。避免在 `dependencies` 中使用 `workspace:*`（npm install 会失败）；将 `openclaw` 放在 `devDependencies` 或 `peerDependencies` 中（运行时通过 jiti 别名解析 `openclaw/plugin-sdk`）。
- 从 `https://openclaw.ai/*` 提供的安装程序：位于同级仓库 `../openclaw.ai` 中（`public/install.sh`、`public/install-cli.sh`、`public/install.ps1`）。
- 消息通道：在重构共享逻辑（路由、白名单、配对、命令门控、入职、文档）时，始终考虑**所有**内置 + 扩展通道。
  - 核心通道文档：`docs/channels/`
  - 核心通道代码：`src/telegram`、`src/discord`、`src/slack`、`src/signal`、`src/imessage`、`src/web`（WhatsApp web）、`src/channels`、`src/routing`
  - 扩展（通道插件）：`extensions/*`（例如 `extensions/msteams`、`extensions/matrix`、`extensions/zalo`、`extensions/zalouser`、`extensions/voice-call`、`extensions/dingtalk`）
- 添加通道/扩展/应用/文档时，更新 `.github/labeler.yml` 并创建匹配的 GitHub 标签（使用现有的通道/扩展标签颜色）。

## 文档链接（Mintlify）

- 文档托管在 Mintlify 上（docs.openclaw.ai）。
- `docs/**/*.md` 中的内部文档链接：根目录相对路径，不带 `.md`/`.mdx`（示例：`[Config](/configuration)`）。
- 处理文档时，阅读 mintlify 技能。
- 章节交叉引用：在根目录相对路径上使用锚点（示例：`[Hooks](/configuration#hooks)`）。
- 文档标题和锚点：避免在标题中使用破折号和撇号，因为它们会破坏 Mintlify 锚点链接。
- 当 Peter 询问链接时，回复完整的 `https://docs.openclaw.ai/...` URL（而非根目录相对路径）。
- 当你修改文档时，在回复末尾附上你引用的 `https://docs.openclaw.ai/...` URL。
- README（GitHub）：保留绝对文档 URL（`https://docs.openclaw.ai/...`），以便链接在 GitHub 上可用。
- 文档内容必须是通用的：不要使用个人设备名称/主机名/路径；使用占位符如 `user@gateway-host` 和 "网关主机"。

## 文档国际化（zh-CN / ja-JP）

- `docs/zh-CN/**` 和 `docs/ja-JP/**` 是生成的；除非用户明确要求，否则不要编辑。
- 流水线：更新英文文档 → 调整词汇表（`docs/.i18n/glossary.<lang>.json`）→ 运行 `go run scripts/docs-i18n/main.go` → 仅在指示时应用针对性修复。
- 翻译记忆库：`docs/.i18n/<lang>.tm.jsonl`（生成的）。
- 参见 `docs/.i18n/README.md`。
- 流水线可能很慢/效率低下；如果它拖慢了速度，在 Discord 上 ping jospalmbier，而不是绕过它。

## 构建、测试和开发命令

- 运行时基线：Node **22+**（保持 Node + Bun 路径正常工作）。
- 安装依赖：`pnpm install`
- 如果缺少依赖（例如 `node_modules` 缺失、`vitest not found` 或 `command not found`），运行仓库的包管理器安装命令（优先使用 lockfile/README 定义的 PM），然后重新运行一次确切的请求命令。将此应用于测试/构建/检查/类型检查/开发命令；如果重试仍然失败，报告命令和第一个可操作的错误。
- 预提交钩子：`prek install`（运行与 CI 相同的检查）
- 也支持：`bun install`（在修改依赖/补丁时保持 `pnpm-lock.yaml` + Bun 补丁同步）。
- 优先使用 Bun 执行 TypeScript（脚本、开发、测试）：`bun <file.ts>` / `bunx <tool>`。
- 在开发中运行 CLI：`pnpm openclaw ...`（bun）或 `pnpm dev`。
- Node 仍然支持运行构建输出（`dist/*`）和生产环境安装。
- Mac 打包（开发）：`scripts/package-mac-app.sh` 默认为当前架构。发布清单：`docs/platforms/mac/release.md`。
- 类型检查/构建：`pnpm build`
- TypeScript 检查：`pnpm tsgo`
- 检查/格式化：`pnpm check`
- 格式化检查：`pnpm format`（oxfmt --check）
- 格式化修复：`pnpm format:fix`（oxfmt --write）
- 测试：`pnpm test`（运行并行测试运行器）；`pnpm test:fast` 用于快速单元测试

## 测试指南

- 框架：Vitest 配合 V8 覆盖率阈值（核心 src 为 70% 行/分支/函数/语句）。
- 命名：使用 `*.test.ts` 匹配源文件名；e2e 测试使用 `*.e2e.test.ts`。
- 测试配置：`vitest.unit.config.ts`（核心单元）、`vitest.e2e.config.ts`（e2e）、`vitest.extensions.config.ts`（扩展）、`vitest.gateway.config.ts`（网关）、`vitest.live.config.ts`（实时测试）。
- 在推送修改逻辑之前运行 `pnpm test:fast`（或 `pnpm test:coverage`）。
- 不要将测试工作线程设置为超过 16；已经试过了。
- 实时测试（真实密钥）：`OPENCLAW_LIVE_TEST=1 pnpm test:live`（仅 OpenClaw）或 `CLAWDBOT_LIVE_TEST=1 pnpm test:live`。Docker：`pnpm test:docker:live-models`、`pnpm test:docker:live-gateway`。入职 Docker E2E：`pnpm test:docker:onboard`。
- 完整套件 + 覆盖范围：`docs/testing.md`。
- 变更日志：仅面向用户的更改；没有内部/元注释（版本对齐、appcast 提醒、发布流程）。
- 纯测试添加/修复通常**不需要**变更日志条目，除非它们改变面向用户的行为或用户要求。
- 移动端：在使用模拟器之前，检查连接的真实设备（iOS + Android），并在可用时优先使用它们。
- 语音通话闭环测试：`pnpm test:voicecall:closedloop`

## 代码风格与命名约定

- 语言：TypeScript（ESM）。优先使用严格类型；避免 `any`。
- 通过 Oxlint 和 Oxfmt 进行格式化/检查；在提交前运行 `pnpm check`。
- 永远不要添加 `@ts-nocheck` 也不要禁用 `no-explicit-any`；修复根本原因，仅在需要时更新 Oxlint/Oxfmt 配置。
- 永远不要通过原型突变共享类行为（`applyPrototypeMixins`、`Object.defineProperty` 在 `.prototype` 上，或导出 `Class.prototype` 进行合并）。使用显式继承/组合（`A extends B extends C`）或辅助组合，以便 TypeScript 可以进行类型检查。
- 如果需要此模式，在发布前停止并获得明确批准；默认行为是拆分/重构为显式类层次结构并保持成员强类型。
- 在测试中，优先使用每个实例的存根而不是原型突变（`SomeClass.prototype.method = ...`），除非测试明确记录了为什么需要原型级修补。
- 为棘手或不明显的逻辑添加简短的代码注释。
- 保持文件简洁；提取辅助函数而不是 "V2" 副本。使用现有模式通过 `createDefaultDeps` 进行 CLI 选项和依赖注入。
- 目标是将文件保持在 ~500 LOC 以下；这只是指导原则（不是硬性限制）。在提高清晰度或可测试性时进行拆分/重构。
- 命名：使用 **OpenClaw** 作为产品/应用/文档标题；使用 `openclaw` 作为 CLI 命令、包/二进制文件、路径和配置键。

## 发布通道（命名）

- stable：仅标记发布（例如 `v2026.2.18`），npm dist-tag `latest`。
- beta：预发布标签 `vYYYY.M.D-beta.N`，npm dist-tag `beta`（可能不包含 macOS 应用）。
- dev：`main` 上的移动头部（无标签；git checkout main）。

## 提交与拉取请求指南

**完整的维护者 PR 工作流（可选）：** 如果你想要仓库的端到端维护者工作流（分类顺序、质量标准、变基规则、提交/变更日志约定、共同贡献者政策以及 `review-pr` > `prepare-pr` > `merge-pr` 流水线），参见 `.agents/skills/PR_WORKFLOW.md`。维护者可能使用其他工作流；当维护者指定工作流时，遵循该工作流。如果没有指定工作流，默认为 PR_WORKFLOW。

- 使用 `scripts/committer "<msg>" <file...>` 创建提交；避免手动 `git add`/`git commit`，以便暂存保持范围限定。
- 遵循简洁、面向行动的提交消息（例如，`CLI: add verbose flag to send`）。
- 分组相关更改；避免捆绑不相关的重构。
- PR 提交模板（规范）：`.github/pull_request_template.md`
- Issue 提交模板（规范）：`.github/ISSUE_TEMPLATE/`

## 速记命令

- `sync`：如果工作树是脏的，提交所有更改（选择一个合理的 Conventional Commit 消息），然后 `git pull --rebase`；如果变基冲突且无法解决，停止；否则 `git push`。

## Git 注意事项

- 如果 `git branch -d/-D <branch>` 被策略阻止，直接删除本地引用：`git update-ref -d refs/heads/<branch>`。
- 批量 PR 关闭/重新打开安全：如果关闭操作会影响超过 5 个 PR，首先要求明确的用户确认，包括确切的 PR 数量和目标范围/查询。

## 安全与配置提示

- Web provider 将凭证存储在 `~/.openclaw/credentials/`；如果注销，重新运行 `openclaw login`。
- Pi 会话默认位于 `~/.openclaw/sessions/` 下；基本目录不可配置。
- 环境变量：参见 `~/.profile`。
- 永远不要提交或发布真实的电话号码、视频或实时配置值。在文档、测试和示例中使用明显虚假的占位符。
- 发布流程：在进行任何发布工作之前，始终阅读 `docs/reference/RELEASING.md` 和 `docs/platforms/mac/release.md`；一旦这些文档回答了问题，就不要再问常规问题。

## exe.dev VM 操作（通用）

- 访问：稳定路径是 `ssh exe.dev` 然后 `ssh vm-name`（假设 SSH 密钥已设置）。
- SSH 不稳定：使用 exe.dev Web 终端或 Shelley（Web 代理）；为长时间操作保持 tmux 会话。
- 更新：`sudo npm i -g openclaw@latest`（全局安装需要在 `/usr/lib/node_modules` 上使用 root）。
- 配置：使用 `openclaw config set ...`；确保设置 `gateway.mode=local`。
- Discord：仅存储原始令牌（不带 `DISCORD_BOT_TOKEN=` 前缀）。
- 重启：停止旧网关并运行：
  `pkill -9 -f openclaw-gateway || true; nohup openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &`
- 验证：`openclaw channels status --probe`、`ss -ltnp | rg 18789`、`tail -n 120 /tmp/openclaw-gateway.log`。

## GHSA（仓库公告）修补/发布

- 获取：`gh api /repos/openclaw/openclaw/security-advisories/<GHSA>`
- 最新 npm：`npm view openclaw version --userconfig "$(mktemp)"`
- 必须关闭私有 fork PR：
  `fork=$(gh api /repos/openclaw/openclaw/security-advisories/<GHSA> | jq -r .private_fork.full_name)`
  `gh pr list -R "$fork" --state open`（必须为空）
- 描述换行陷阱：通过 heredoc 将 Markdown 写入 `/tmp/ghsa.desc.md`（不要使用 `"\\n"` 字符串）
- 通过 jq 构建修补 JSON：`jq -n --rawfile desc /tmp/ghsa.desc.md '{summary,severity,description:$desc,vulnerabilities:[...]}' > /tmp/ghsa.patch.json`
- 修补 + 发布：`gh api -X PATCH /repos/openclaw/openclaw/security-advisories/<GHSA> --input /tmp/ghsa.patch.json`（发布 = 包含 `"state":"published"`；没有 `/publish` 端点）
- 如果发布失败（HTTP 422）：缺少 `severity`/`description`/`vulnerabilities[]`，或私有 fork 有开放的 PR
- 验证：重新获取；确保 `state=published`、`published_at` 已设置；`jq -r .description | rg '\\\\n'` 返回空

## 故障排除

- 品牌重塑/迁移问题或旧配置/服务警告：运行 `openclaw doctor`（参见 `docs/gateway/doctor.md`）。

## 代理特定说明

- 词汇表："makeup" = "mac app"。
- 永远不要编辑 `node_modules`（全局/Homebrew/npm/git 安装也是）。更新会覆盖。技能说明放在 `tools.md` 或 `AGENTS.md` 中。
- 在仓库中的任何位置添加新的 `AGENTS.md` 时，还要添加一个指向它的 `CLAUDE.md` 符号链接（示例：`ln -s AGENTS.md CLAUDE.md`）。
- Signal："update fly" => `fly ssh console -a flawd-bot -C "bash -lc 'cd /data/clawd/openclaw && git pull --rebase origin main'"` 然后 `fly machines restart e825232f34d058 -a flawd-bot`。
- 处理 GitHub Issue 或 PR 时，在任务末尾打印完整的 URL。
- 回答问题时，仅回复高置信度的答案：在代码中验证；不要猜测。
- 永远不要更新 Carbon 依赖项。
- 任何带有 `pnpm.patchedDependencies` 的依赖项必须使用精确版本（没有 `^`/`~`）。
- 修补依赖项（pnpm 补丁、覆盖或供应商更改）需要明确批准；默认情况下不要这样做。
- CLI 进度：使用 `src/cli/progress.ts`（`osc-progress` + `@clack/prompts` 旋转器）；不要手动创建旋转器/进度条。
- 状态输出：保持表格 + ANSI 安全换行（`src/terminal/table.ts`）；`status --all` = 只读/可粘贴，`status --deep` = 探测。
- 网关目前仅作为菜单栏应用运行；没有单独的 LaunchAgent/辅助标签安装。通过 OpenClaw Mac 应用或 `scripts/restart-mac.sh` 重启；要使用 `launchctl print gui/$UID | grep openclaw` 验证/终止，而不是假设固定标签。**在 macOS 上调试时，通过应用启动/停止网关，而不是临时 tmux 会话；在交接前终止任何临时隧道。**
- macOS 日志：使用 `./scripts/clawlog.sh` 查询 OpenClaw 子系统的统一日志；它支持 follow/tail/类别过滤器，并期望 `/usr/bin/log` 具有免密码 sudo。
- 如果本地有共享护栏，请查看它们；否则遵循本仓库的指南。
- SwiftUI 状态管理（iOS/macOS）：优先使用 `Observation` 框架（`@Observable`、`@Bindable`）而不是 `ObservableObject`/`@StateObject`；除非需要兼容性，否则不要引入新的 `ObservableObject`，并在修改相关代码时迁移现有用法。
- 连接提供商：添加新连接时，更新每个 UI 界面和文档（macOS 应用、Web UI、移动端（如果适用）、入职/概述文档），并添加匹配的状态 + 配置表单，以便提供商列表和设置保持同步。
- 版本位置：`package.json`（CLI）、`apps/android/app/build.gradle.kts`（versionName/versionCode）、`apps/ios/Sources/Info.plist` + `apps/ios/Tests/Info.plist`（CFBundleShortVersionString/CFBundleVersion）、`apps/macos/Sources/OpenClaw/Resources/Info.plist`（CFBundleShortVersionString/CFBundleVersion）、`docs/install/updating.md`（固定的 npm 版本）、`docs/platforms/mac/release.md`（APP_VERSION/APP_BUILD 示例）、Peekaboo Xcode 项目/Info.plists（MARKETING_VERSION/CURRENT_PROJECT_VERSION）。
- "Bump version everywhere" 表示上述所有版本位置**除了** `appcast.xml`（仅在发布新的 macOS Sparkle 版本时才修改 appcast）。
- **重启应用：** "restart iOS/Android apps" 意味着重新构建（重新编译/安装）并重新启动，而不仅仅是终止/启动。
- **设备检查：** 在测试之前，验证连接的真实设备（iOS/Android），然后再使用模拟器/仿真器。
- iOS Team ID 查找：`security find-identity -p codesigning -v` => 使用 Apple Development (…) TEAMID。备用：`defaults read com.apple.dt.Xcode IDEProvisioningTeamIdentifiers`。
- A2UI 包哈希：`src/canvas-host/a2ui/.bundle.hash` 是自动生成的；忽略意外的更改，仅在需要时通过 `pnpm canvas:a2ui:bundle`（或 `scripts/bundle-a2ui.sh`）重新生成。将哈希作为单独的提交提交。
- 发布签名/公证密钥在仓库外部管理；遵循内部发布文档。
- Notary 认证环境变量（`APP_STORE_CONNECT_ISSUER_ID`、`APP_STORE_CONNECT_KEY_ID`、`APP_STORE_CONNECT_API_KEY_P8`）应在你的环境中（根据内部发布文档）。
- **多代理安全：** 除非明确要求，否则**不要**创建/应用/删除 `git stash` 条目（这包括 `git pull --rebase --autostash`）。假设其他代理可能正在工作；保持不相关的 WIP 不受影响，避免跨领域的状态更改。
- **多代理安全：** 当用户说 "push" 时，你可以 `git pull --rebase` 来集成最新更改（永远不要丢弃其他代理的工作）。当用户说 "commit" 时，仅限定在你的更改范围内。当用户说 "commit all" 时，以分组块提交所有内容。
- **多代理安全：** 除非明确要求，否则**不要**创建/删除/修改 `git worktree` 检出（或编辑 `.worktrees/*`）。
- **多代理安全：** 除非明确要求，否则**不要**切换分支 / 检出不同的分支。
- **多代理安全：** 只要每个代理有自己的会话，运行多个代理是可以的。
- **多代理安全：** 当你看到无法识别的文件时，继续前进；专注于你的更改，仅提交这些更改。
- 检查/格式化变动：
  - 如果暂存+未暂存的差异仅是格式化，自动解决，无需询问。
  - 如果已经请求了 commit/push，自动暂存并在同一提交中包含仅格式化的后续操作（或如果需要，一个微小的后续提交），无需额外确认。
  - 仅在更改是语义性（逻辑/数据/行为）时才询问。
- Lobster 接缝：使用 `src/terminal/palette.ts` 中的共享 CLI 调色板（没有硬编码颜色）；根据需要应用调色板到入职/配置提示和其他 TTY UI 输出。
- **多代理安全：** 专注于你的编辑报告；除非真正被阻止，否则避免护栏免责声明；当多个代理修改同一文件时，如果安全则继续；仅在相关时以简短的"存在其他文件"说明结束。
- Bug 调查：在得出结论之前，阅读相关 npm 依赖项的源代码和所有相关的本地代码；旨在找到高置信度的根本原因。
- 代码风格：为棘手的逻辑添加简短注释；尽可能将文件保持在 ~500 LOC 以下（根据需要拆分/重构）。
- 工具模式护栏（google-antigravity）：避免在工具输入模式中使用 `Type.Union`；没有 `anyOf`/`oneOf`/`allOf`。对字符串列表使用 `stringEnum`/`optionalStringEnum`（Type.Unsafe enum），使用 `Type.Optional(...)` 而不是 `... | null`。保持顶级工具模式为 `type: "object"` 和 `properties`。
- 工具模式护栏：避免在工具模式中使用原始 `format` 属性名称；某些验证器将 `format` 视为保留关键字并拒绝该模式。
- 当要求打开 "session" 文件时，打开 `~/.openclaw/agents/<agentId>/sessions/*.jsonl` 下的 Pi 会话日志（使用系统提示中 Runtime 行的 `agent=<id>` 值；除非给出特定 ID，否则使用最新的），而不是默认的 `sessions.json`。如果需要在另一台机器上获取日志，通过 Tailscale SSH 并读取相同路径。
- 不要通过 SSH 重建 macOS 应用；重建必须直接在 Mac 上运行。
- 永远不要向外部消息界面（WhatsApp、Telegram）发送流式/部分回复；只有最终回复应该被传送到那里。流式/工具事件仍然可以进入内部 UI/控制通道。
- 语音唤醒转发提示：
  - 命令模板应保持 `openclaw-mac agent --message "${text}" --thinking low`；`VoiceWakeForwarder` 已经对 `${text}` 进行了 shell 转义。不要添加额外的引号。
  - launchd PATH 是最小的；确保应用的启动代理 PATH 包含标准系统路径加上你的 pnpm bin（通常是 `$HOME/Library/pnpm`），以便在通过 `openclaw-mac` 调用时 `pnpm`/`openclaw` 二进制文件可以解析。
- 对于包含 `!` 的手动 `openclaw message send` 消息，使用下面提到的 heredoc 模式来避免 Bash 工具的转义。
- 发布护栏：未经操作员明确同意，不要更改版本号；在运行任何 npm 发布/发布步骤之前始终请求许可。

## NPM + 1Password（发布/验证）

- 使用 1password 技能；所有 `op` 命令必须在新的 tmux 会话中运行。
- 登录：`eval "$(op signin --account my.1password.com)"`（应用已解锁 + 集成开启）。
- OTP：`op read 'op://Private/Npmjs/one-time password?attribute=otp'`。
- 发布：`npm publish --access public --otp="<otp>"`（从包目录运行）。
- 在没有本地 npmrc 副作用的情况下验证：`npm view <pkg> version --userconfig "$(mktemp)"`。
- 发布后终止 tmux 会话。

## 插件发布快速路径（不发布核心 `openclaw`）

- 仅发布已在 npm 上的插件。源列表在 `docs/reference/RELEASING.md` 中的 "Current npm plugin list" 下。
- 在 tmux 中运行所有 CLI `op` 调用和 `npm publish` 以避免挂起/中断：
  - `tmux new -d -s release-plugins-$(date +%Y%m%d-%H%M%S)`
  - `eval "$(op signin --account my.1password.com)"`
- 1Password 辅助工具：
  - `npm login` 使用的密码：
    `op item get Npmjs --format=json | jq -r '.fields[] | select(.id=="password").value'`
  - OTP：
    `op read 'op://Private/Npmjs/one-time password?attribute=otp'`
- 快速发布循环（`/tmp` 中的本地辅助脚本可以；保持仓库干净）：
  - 将本地插件 `version` 与 `npm view <name> version` 进行比较
  - 仅在版本不同时运行 `npm publish --access public --otp="<otp>"`
  - 如果包在 npm 上缺失或版本已匹配，则跳过。
- 保持 `openclaw` 不变：除非明确要求，否则永远不要从仓库根目录运行发布。
- 每次发布后的检查：
  - 每个插件：`npm view @openclaw/<name> version --userconfig "$(mktemp)"` 应该是 `2026.2.17`
  - 核心保护：`npm view openclaw version --userconfig "$(mktemp)"` 应保持在前一个版本，除非明确要求。

## 变更日志发布说明

- 当使用 beta GitHub 预发布版本发布 mac 版本时：
  - 从发布提交中标记 `vYYYY.M.D-beta.N`（示例：`v2026.2.15-beta.1`）。
  - 创建标题为 `openclaw YYYY.M.D-beta.N` 的预发布版本。
  - 使用 `CHANGELOG.md` 版本部分（`Changes` + `Fixes`，不重复标题）的发布说明。
  - 至少附加 `OpenClaw-YYYY.M.D.zip` 和 `OpenClaw-YYYY.M.D.dSYM.zip`；如果可用，包括 `.dmg`。

- 保持 `CHANGELOG.md` 中的顶级版本条目按影响排序：
  - `### Changes` 优先。
  - `### Fixes` 去重并按面向用户的修复优先排序。
- 在标记/发布之前，运行：
  - `node --import tsx scripts/release-check.ts`
  - `pnpm release:check`
  - `pnpm test:install:smoke` 或 `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` 用于非 root 烟雾路径。
