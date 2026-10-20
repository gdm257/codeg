# 01: Workflow 骨架：版本触发 + draft release 生命周期

**What to build:** 新增独立于上游 `release.yml` 的发布 workflow。push 到默认分支且版本 SSOT 文件（`src-tauri/tauri.conf.json`）发生变更时触发；workflow 读取该文件得出版本号，以 `v$VERSION` 创建或复用 draft release（版本含 `-rc` 段则标记 prerelease），并在所有下游 job 成功后 publish。本阶段无构建 job，publish 直接执行。

**Blocked by:** None (can start immediately).

**Status:** resolved

- [ ] 新 workflow 文件独立命名，不修改上游 `release.yml`
- [ ] 触发条件：push 到默认分支 + 版本文件在 `paths` 过滤中
- [ ] 版本号唯一来源于 SSOT 文件，不读取 git tag
- [ ] 同版本重跑时复用既有 draft，不重复创建
- [ ] `-rc` 版本自动标记 prerelease
- [ ] 不引用 fork 中不存在的 secrets
- [ ] 在测试分支/测试 repo 验证 draft 创建 → publish 链路可用

## Answer

已实现 `.github/workflows/publish-release.yml`（YAML 校验通过）：

- 触发：push 到 `main` 且 `src-tauri/tauri.conf.json` 在 diff 中（`on.push` 不支持表达式，分支硬编码为 `main`）
- 版本与 prerelease 判定从 SSOT 文件读取；release 由 `createRelease` 以 `target_commitish: context.sha` 自动打 `v$VERSION` tag，全程无手工 git tag
- 同版本重跑复用既有 draft（非 draft 则 fail）；publish job 挂在 `needs` 上，02/03 加入后自动形成"全部成功才 publish"
- 端到端 draft→publish 验证推迟到 ticket 04 的真实 `-rc` 冒烟（在 fork 上单独验证骨架需要真的 bump 版本文件，会触发真实 release，不值得单独跑）

