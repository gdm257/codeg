# 02: 桌面安装包构建上传

**What to build:** 在 01 的 workflow 中加入桌面构建 job：6 平台 matrix（macOS x64/arm64、Linux x64/arm64、Windows x64/arm64），复用上游的 sidecar staging 流程（`tauri:prepare-sidecars` + `tauri-action` 上传到 draft）。剥离全部 Apple 证书导入、公证、trust-anchor pin 与 updater 签名步骤；macOS dmg 无签名发布。publish job 依赖本 job 成功。

**Blocked by:** 01

**Status:** resolved

- [ ] 6 平台 matrix 全部构建成功并上传到 draft release
- [ ] sidecar（codeg-mcp / codeg-computer-helper / codeg-cli）随桌面 bundle 打包并有落地校验
- [ ] macOS 产物无签名、无公证，流水线不引用任何 Apple secrets
- [ ] 不产出 `.sig` 与 updater `latest.json`
- [ ] 任一平台失败时 release 不 publish
- [ ] Linux arm64 交叉编译链沿用上游做法

## Answer

已实现于 `publish-release.yml` 的 `build-tauri` job（YAML 校验通过）：上游 job 去掉 Apple 证书导入/身份解析/helper 签名/trust-anchor pin/computer-use 签名校验/keychain 清理，及全部 `TAURI_SIGNING_*` env。Windows bundles 改为仅 `nsis`（updater 签名缺失），全部 `includeUpdaterJson: false`。macOS 构建分出去后的 Linux x64 + macOS 共用一个 tauri-action step。publish job 已加入 `needs.build-tauri`。

