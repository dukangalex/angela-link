# AngelaLink 维护规矩（本仓自有，替代上游协作指南）

- 本仓是 dukangalex 名下的自有化项目，与上游 Aenvo/MuseLink 脱钩独立维护；
  不跟上游同步节奏，上游更新只作情报参考，收编须过自家审计与基线复测。
- 安全边界是本仓的命：文件授权、路径解析（core/path_guard.rs、core/access.rs）、
  令牌处理（core/token.rs）、沙箱（core/sandbox.rs）与审批策略（core/exec_policy.rs）
  的任何改动，先在隔离测试目录＋测试凭证下验证，再跑全量 cargo test。
- 文件操作必须经 core/handle.rs 的已校验句柄路径（校验后对象变化返回 409），
  不许退回只按路径字符串操作。
- 界面文案走 angela-link/src/lib/i18n.tsx 与 messages.ts，中英文同步改。
- 环境选择不焊死进本体：设备名、隧道域名、更新源、令牌目录属部署配置。
- 改用户流程/安装/构建命令时同步更新 README.md 与 README.en.md；
  功能更新同步记 CHANGELOG.md 与 CHANGELOG.en.md。
- 提交前排除本机配置、凭证、日志、构建产物。
