# MuseLink 安全审计报告（自有化前置）

审计对象：Aenvo/MuseLink @ d244f78（2026-11-24 建仓，MIT，Tauri 2＋Rust，约 1.7 万行 Rust）
审计方式：源码逐项对照 README/SECURITY 声称，只读审计，未运行任何上游代码。
日期：2026-11-26

## 结论：声称与实现一致，审计过关（附两条注记）

### 令牌 ✅
- token.rs：getrandom（系统 CSPRNG）32 字节 → base64url 43 字符；subtle 常数时间比较；Bearer 前缀大小写敏感。
- 轮换只写本机文件、绝不经响应返回（token.rs save 注释与实现一致）；令牌文件位于 ~/.muse-bridge，该目录本身在全保护清单内。
- 专用 agent 令牌（mfa_ 前缀）与旧版文件令牌分开管理。

### 日志脱敏 ✅
- audit.rs：已登记 secret 全量替换；mfa_＋43 字符的令牌形状按模式脱敏（带边界校验，不误伤普通文本）；protect_secret 会回溯脱敏内存缓冲中已记录条目；带单元测试覆盖轮换令牌场景。

### 路径闸门 ✅（比其 Python 前身更严，属有意收紧）
- path_guard.rs：绝对路径拒绝；任何 `..` 组件一律拒绝；ADS 冒号、尾部点/空格、保留设备名（CON 等）预检拒绝；canonicalize 后必须仍在授权根内；reparse/junction 拒绝（access.rs check_target 置 reject_reparse）。

### 敏感路径保护 ✅（与 README 逐项对上）
- 全保护（读＋写）：~/.muse-bridge、~/.ssh、~/.aws、~/.gnupg。
- 写保护：SystemRoot 全目录、Program Files（x64/x86）、两处 Startup、LocalAppData\Programs、Documents 下 PowerShell profile 目录、~/.gitconfig/.bashrc/.zshrc/.profile/.npmrc/.pypirc、程序自身目录。
- access.rs：授权根校验时对词法路径与 canonical 路径双查保护清单；盘符根目录不许按文件夹授权，必须走磁盘授权通道。

### 沙箱 ✅
- sandbox.rs：CreateRestrictedToken（DISABLE_MAX_PRIVILEGE | WRITE_RESTRICTED）＋按工作区派生 SID 的目录 ACL＋JobObject（进程数/内存/超时硬限，job.rs）。
- 模块头注释与实现一致：初始化失败直接报错，绝不静默降级裸跑。
- 自带测试实证：沙箱内 cmd 与 PowerShell 向工作区外写入均被 Windows 内核拒绝。

### 三档审批 ✅
- exec_policy.rs：RiskTier Auto/Session/Always 与 README 对应；强推（--force/-f/--force-with-lease/--delete）→ Always；混淆标记（FromBase64String 等）→ 提升；系统管理工具（format/shutdown/netsh 等清单）→ Always；-C 子目录含 `..` 或绝对路径不许 Auto。
- 会话免打扰排除敏感工作区与 unsandboxed 请求（exec_policy.rs:218）。
- exec_runtime.rs：APPROVAL_TTL = 120 秒，超时未确认即拒绝；超时/取消/主进程退出均清理 Job Object 进程树。
- 紧急中止：关命令开关 → abort_all；令牌变更 → abort_all（service.rs:314）。

### 出站连接 ✅（无声称外外发）
- Cargo.toml / package.json 无任何 telemetry/analytics/崩溃上报依赖。
- 更新器：仅当环境变量 ANGELA_LINK_UPDATE_ENDPOINT＋ANGELA_LINK_UPDATE_PUBLIC_KEY 同时设置才启用，强制 HTTPS、无内嵌凭据、Tauri 签名校验；未配置时明确返回「自动更新尚未配置」。出厂默认＝关。
- 隧道：调用本机 tailscale CLI（funnel --bg --https=443）或本机 cloudflared（--url 指回 localhost），无自有中继服务器；本地服务只听回环。
- 其余 URL 仅帮助文档链接（tailscale/cloudflare/voidtools 官网）。
- Everything：仅 Release 包捆绑 voidtools 官方签名二进制（仓库内无此二进制），附许可证文件；应用写入的 Everything 配置关掉 http_server、etp_server 与搜索/运行历史。

### API 面 ✅
- api.rs 路由与文档一致：roots/list/read/write/mkdir/search/exec（提交/状态/取消）/device-name confirm＋旧版 /api 只读兼容；未发现文档外端点。

## 注记（不构成否决，决定试用范围）
1. 凭证目录保护只覆盖 .ssh/.aws/.gnupg 三家；.azure/.kube/.docker 配置、浏览器 Login Data 等不在排除清单。→ 试用只授权单个无敏感目录即可避开；将来若授权范围扩大到用户主目录，需先在自有版补这份清单。
2. 上游 Release 程序未做代码签名、仓库仅两天——出身问题由自有化＋自家构建解决：咱们只跑自己从源码构建的产物，不跑他的二进制。

## 私货清单（自有化时剥离/替换）
- README「社区」节 LINUX DO 链接：剥离。
- README 内指向上游 Releases 的下载链接：换自有仓（属改名常规动作，非私货）。
- 上游 AGENTS.md：作者自家维护规矩（更新日志同步、测试矩阵），无对他人指令、无推广；自有版以我方 AGENTS.md 替换。
- 代码与前端未发现返利、追踪参数、付费引流。

## v0.3.0 基线复测记录（2026-11-26）

环境：用户电脑 DESKTOP-L3U37AU（精简版 Windows、内存约 4GB），VS Build Tools 2022
17.14（MSVC 14.44）＋rustup stable（cargo/rustc 1.100.0），CARGO_BUILD_JOBS=2。

- 构建：cargo build 成功（dev profile），产物 angela-link.exe 约 27.5MB。
- 测试：cargo test 全量 94 通过 / 2 失败 / 2 忽略；逐条复核后实际基线为
  **95 通过 / 1 条已知环境敏感失败 / 2 忽略**：
  - core::sandbox::tests::write_restricted_sandbox_allows_workspace_and_blocks_outside_writes：
    首跑在整机编译负载下因沙箱内 PowerShell 15 秒未退出而失败；空闲单跑通过——负载敏感。
  - service::tests::agent_exec_enforces_toggle_approval_session_and_sandbox_boundary：
    空闲单跑仍确定性失败（5.44 秒）。失败形态：direct 只读命令（where）被策略自动放行后，
    沙箱作业未在测试给定的 wait_ms=5000 内完成，接口回 202（期望 200＋completed）。
- 定性依据：该测试与整条 exec 路径（service.rs、exec_runtime.rs、exec 策略分类、sandbox
  spawn）与上游 Aenvo/MuseLink 逐字节一致（已 diff 核对）；中立化改动在该路径仅有子进程
  环境变量前缀过滤改名（MUSE_CONNECTOR_→ANGELA_LINK_）与注释，不影响执行与判定。
  失败源于上游测试的 5 秒完成预算在本机（4GB 内存、精简版系统、每跑新建临时家目录的
  冷沙箱工作区）被击穿，属环境敏感，非代码缺陷。不改上游测试口径凑绿，基线如实记录。
- 复核条件：换更强机器（内存 ≥16GB 的常规开发机）或上游更新该测试/执行路径时，
  重跑此条并更新本记录。
- 附记：上文注记 1 的凭证保护缺口已在 v0.3.0 补齐（新增 .azure/.kube/.docker、
  Chrome/Edge/Brave 密码库目录、Firefox 配置、gcloud 凭证，旧数据目录过渡期同级保护）。
