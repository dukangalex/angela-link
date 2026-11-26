# AngelaLink · 本地 AI 桥（中文别名：安渡）

让 AI 在你授权的范围内访问你 Windows 电脑上的文件，并按需在沙箱与本机审批保护下执行项目命令。中文别名「安渡」：安取安全之意，渡是摆渡——AI 经你自己的隧道安全渡入，门口有审批与审计守着。

AngelaLink 是一个通用的本地桥：电脑端负责授权、沙箱、审批与审计，AI 端通过一组带令牌的
HTTP 接口（`/agent/v1`）接入。它不绑定任何一家 AI——Muse、其他助手、你自己的 agent
脚本，只要能发 HTTP 请求并保管好令牌，都可以接入。接入说明见
[docs/AGENT-BRIEF.md](docs/AGENT-BRIEF.md)，通用化路线见 [docs/ROADMAP.md](docs/ROADMAP.md)。

## 功能概览

| 能力 | 可以做什么 |
| --- | --- |
| 文件授权 | 选择指定文件夹及其子目录，或从本机固定磁盘中选择授权范围；读/写/搜索分别授权。默认保护凭证与敏感系统路径。 |
| 本机文件工具 | 让 AI 按文件名查找文件、列出目录、读取内容，并按需写入与建目录。文件名搜索由内置索引或 voidtools Everything 提供。 |
| 受控命令执行 | 默认关闭；开启后 AI 可在授权项目子目录内运行开发工具与受控 PowerShell/cmd，运行于每工作区独立 SID 的 Windows 写受限沙箱与三档审批弹窗之下。 |
| 公网连接 | 默认使用 Tailscale Funnel，也支持 Cloudflare 命名隧道（自有域名）。隧道只调本机已装的 tailscale / cloudflared，无自有中继。 |
| 专用凭证 | 为接入方生成独立令牌，可轮换、撤销；轮换或关停会立即中止全部运行中命令。日志自动脱敏令牌。 |
| 桌面管理 | 按时间、结果、范围与关键词检索日志；管理本机文件索引、日志与令牌；开机静默自启、托盘常驻；简体中文 / English。 |

## 安全模型（摘要）

- **路径闸门**：任何 `..`、ADS 数据流、尾部点/空格、保留设备名一律拒绝；路径解析后必须仍在授权根内，junction/符号链接逃逸被拒。
- **敏感路径**：本程序数据目录（含令牌）、`~/.ssh`、`~/.aws`、`~/.gnupg` 全保护（不可读不可写）；系统目录、启动项、Shell Profile、`.gitconfig` 与程序自身目录默认写保护。
- **沙箱**：`CreateRestrictedToken(WRITE_RESTRICTED)` + 每工作区独立 SID 的目录 ACL + JobObject 内存/进程数/超时硬限制；沙箱初始化失败直接拒绝执行，绝不降级裸跑。
- **三档审批**：常见只读与标准构建/测试命令自动放行；首次在某项目内用 shell、装依赖、git push 走本机置顶审批（120 秒未确认自动拒绝，可给 4 小时项目会话授权）；强推、递归删除、系统管理工具、混淆脚本、触碰敏感目录等始终强制单次确认。
- **更新**：自动更新默认关闭，仅当部署时显式设置更新源环境变量才启用（HTTPS＋签名校验）。

完整安全审计（逐项源码对照）见 [docs/AUDIT.md](docs/AUDIT.md)。

## 从源码构建

准备 Windows、WebView2、Node.js 22.12+、pnpm、Rust MSVC 工具链与 Visual Studio C++ Build Tools。

```
pnpm -C angela-link install --frozen-lockfile
pnpm -C angela-link tauri dev            # 开发模式（隔离配置）
pnpm -C angela-link build
cargo test --manifest-path angela-link/src-tauri/Cargo.toml --locked
pnpm -C angela-link tauri build --bundles nsis
```

注：v0.3.0 起源码目录、crate 与数据目录已完成中立化（angela-link / ~/.angela-link），
旧数据目录 ~/.muse-bridge 首启自动迁移、旧令牌前缀 mfa_ 过渡期双认（见 docs/ROADMAP.md）。

## 许可证与致谢

本项目采用 MIT License 开源，许可证原文见 [LICENSE](LICENSE)，
第三方组件声明见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

本项目自有化自 [Aenvo/MuseLink](https://github.com/Aenvo/MuseLink)（MIT），
其旧版文件接口基于 muse-file-bridge v0.3 协议。致谢详见 [CREDITS.md](CREDITS.md)。
自有化后与上游脱钩独立维护，不跟随上游同步节奏。
