Status: ready-for-agent

# Spec: publish-release-ci

## Problem Statement

本仓库是上游 codeg 的 fork。上游自带的 Release workflow 依赖一组 fork 中不存在的 secrets（Apple 签名/公证、Tauri updater 签名 key、DockerHub 凭据），因此每次 push tag 在 fork 里必然失败。fork 维护者需要一条自己能跑通的发布流水线，产物只落在 GitHub Releases，不涉及任何其他发布渠道。

## Solution

新增一个独立于上游 `release.yml` 的 workflow 文件（如 `publish-release.yml`），由 push 到默认分支且 `src-tauri/tauri.conf.json` 发生变更时触发（`paths` 过滤），版本号直接从该文件读取：

- 构建桌面安装包：macOS dmg（无签名、无公证）、Linux deb/rpm、Windows nsis
- 构建 server 侧产物：各平台 codeg-server tarball/zip（含 codeg-mcp、codeg-computer-helper、codeg-cli 与静态 web），附 `.sha256`
- 全部上传到一个 draft release，全部 job 成功后 publish
- 版本号（含 release 名与 prerelease 判定）以 SSOT 文件内容为准，不依赖 git tag

上游 `release.yml` 保持原样不动（建议在 fork Settings 中手动 disable，避免同一 tag 双跑）。

## User Stories

1. As a fork maintainer, I want push 到默认分支且版本文件变更时自动触发发布流水线, so that 我不需要手动打 tag、构建和上传任何产物
2. As a fork maintainer, I want 版本号唯一来源于 `tauri.conf.json`, so that 不存在 tag 与文件版本不一致的可能
3. As a fork maintainer, I want 所有产物先汇总到 draft release、全部 job 成功后才 publish, so that 不会出现半成品 release
4. As a fork maintainer, I want 任一平台构建失败时 release 保持 draft（或整体失败）, so that 用户不会下载到缺失平台的版本
5. As a fork maintainer, I want workflow 不引用任何 fork 中不存在的 secrets, so that 每次发版都能跑通
6. As a fork maintainer, I want macOS dmg 以无签名方式发布, so that mac 用户仍可右键打开绕过 Gatekeeper 使用
7. As a fork user, I want release 附带每个 server 压缩包的 sha256, so that 我能校验下载完整性
8. As a fork user, I want release 附带 Linux deb/rpm 与 Windows nsis 安装包, so that 我能直接安装桌面版
9. As a fork maintainer, I want 同一版本重跑时 draft release 可复用并覆盖上传, so that 修复构建失败后重跑无需清理
10. As a fork maintainer, I want 含 `-rc` 的版本号自动标记为 prerelease, so that 正式 latest 通道不被候选版污染
11. As a fork maintainer, I want workflow 与上游 `release.yml` 并存且零修改后者, so that 定期 merge upstream 不产生冲突
12. As a fork maintainer, I want sidecar（codeg-mcp / codeg-computer-helper / codeg-cli）随桌面 bundle 一起打包, so that 桌面版的多智能体委托与 computer use 功能可用

## Implementation Decisions

- 新 workflow 文件独立命名，不修改上游 `release.yml`（merge upstream 友好）
- 版本 SSOT 为 `src-tauri/tauri.conf.json`；触发条件为 push 到默认分支且该文件在 diff 中（`on.push.paths` 过滤），版本号由 workflow 读取该文件得出。bump 版本时同步 `package.json` 与 `src-tauri/Cargo.toml`（另行处理，不在本 spec 范围内）
- 不打 git tag；release 的 tag 名由 workflow 以 `v$VERSION` 创建（GitHub release 自带 tag），同一版本重跑时复用既有 draft
- prerelease 判定：版本号含 `-rc`（或同类预发布段）即 prerelease
- 桌面构建沿用上游的 matrix 结构与 sidecar staging 流程（`tauri:prepare-sidecars` + `tauri-action`），但剥离全部 Apple 证书/公证步骤与 trust-anchor pin；macOS 产物无签名发布
- 不设置 `TAURI_SIGNING_PRIVATE_KEY`，因此不产出 `.sig` 与 updater `latest.json`（fork 的自更新不可用，属预期）
- 不包含 Docker/ghcr/DockerHub 相关 job 与 artifact 上传
- server 产物沿用上游 build-server 的目标矩阵、打包与 smoke test 步骤，签名步骤替换为仅生成 `.sha256`
- 与上游 `release.yml`（tag 触发）触发条件天然不同，无撞车风险；上游 workflow 仍建议 disable，避免误推 tag 时双跑
- draft release 创建/复用/publish 逻辑沿用上游的 github-script 模式（create-or-reuse draft → 全部成功 → publish）

## Testing Decisions

- 纯 CI workflow，无运行时代码，不写单元测试
- 静态校验：YAML 语法解析（必要时 `actionlint`）
- 端到端冒烟：push 一个仅修改版本文件（含 `-rc` 预发布段）的提交触发完整流水线，确认 draft 创建、各平台产物上传、prerelease 标记与最终 publish 均正常；验证后删除测试 release，并把版本文件改回
- 仅验证外部行为（workflow 是否产出正确 release 与资产），不测 workflow 内部步骤编排细节

## Out of Scope

- 代码签名、公证、updater 签名与自更新通道
- Docker / ghcr / Docker Hub 发布
- npm 或其他包管理器发布
- 修改或删除上游 `release.yml`（仅建议在 fork Settings 中 disable）
- 版本 bump 的自动化脚本（SSOT 同步另行处理）

## Further Notes

- 上游 `release.yml` 在 fork 中会因缺 secrets 失败，属预期噪音；disable 它是维护者的一次性手动操作
- fork 定期 merge upstream 时，三个版本号文件（package.json / Cargo.toml / tauri.conf.json）必然冲突，bump 时统一为 merge 后的目标版本即可
