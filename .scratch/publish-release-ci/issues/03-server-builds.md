# 03: Server 产物构建上传

**What to build:** 在 01 的 workflow 中加入 server 构建 job：目标矩阵（Linux x64/arm64、macOS x64/arm64、Windows x64），构建 codeg-server、codeg-mcp、codeg-computer-helper、codeg-cli 及静态 web 前端，打包为 tarball/zip 并附 `.sha256`，沿用上游 smoke test 步骤后上传 draft。不包含 Docker 相关任何步骤与 artifact 上传。publish job 依赖本 job 成功。与 02 相互独立，可并行。

**Blocked by:** 01

**Status:** resolved

- [ ] 各目标平台 tarball/zip 上传到 draft release，内含四个二进制 + web 静态文件
- [ ] 每个压缩包附 `.sha256`
- [ ] 沿用上游 smoke test（`--help` / `--version` 校验，交叉目标跳过执行）
- [ ] 无签名步骤、无 Docker artifact、无 `TAURI_SIGNING_PRIVATE_KEY`
- [ ] 任一目标失败时 release 不 publish

## Answer

已实现于 `publish-release.yml` 的 `build-server` job（YAML 校验通过）：上游 job 去掉 minisign 签名步骤（`tauri signer sign` 及 `.sig` 校验）与 Docker artifact 上传，checksum 步骤保留 `.sha256`，`gh release upload` 目标改为从 draft job 输出的 `tag`（`v$VERSION`，draft release 可按 tag 名上传）。publish job 已加入 `needs.build-server`。

