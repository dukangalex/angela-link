# CREDITS

## 上游出处

- **Aenvo/MuseLink** — https://github.com/Aenvo/MuseLink （MIT License）
  本仓库的起点：Tauri 2 + Rust 桌面端、文件授权与路径闸门、写受限沙箱、
  三档审批、Tailscale Funnel / Cloudflare 隧道接入、审计日志与令牌管理，
  均源自该项目（导入版本为其 2026-11-26 的 main 提交 d244f78）。
  其旧版文件接口基于 muse-file-bridge v0.3 协议（见上游仓库说明）。

## 第三方组件

- **voidtools Everything & ES** — Release 包捆绑官方 Authenticode 签名二进制
  （仓库内不含），许可证见 THIRD_PARTY_NOTICES.md 与随包的许可证文件。
- 其余依赖（Tauri、React、Rust crates、npm 包）各自适用其原始许可证，
  汇总见 THIRD_PARTY_NOTICES.md。

## 自有化说明

2026-11-26 起，本项目在 dukangalex 名下以 AngelaLink 之名独立维护：
自有命名与文档、与上游脱钩；上游许可证原文保留、致谢如实；
改造方向为「通用本地 AI 桥」——接入面与任何单一 AI 产品解绑。
