# 04: 端到端冒烟与收尾

**What to build:** 用一次真实的 `-rc` 版本 bump 提交触发完整流水线，端到端验证：自动触发、draft 创建、全部平台产物上传、prerelease 标记、最终 publish。验证后清理测试 release 并把版本文件改回。同时在 fork Settings 中 disable 上游 `release.yml`（手动操作，本 ticket 记录步骤即可）。

**Blocked by:** 02, 03

**Status:** ready-for-agent

- [ ] 一次 `-rc` 版本提交自动触发全流水线并跑通
- [ ] release 资产齐全：桌面安装包 + server 压缩包 + `.sha256`，无 `.sig`、无 latest.json
- [ ] prerelease 标记正确
- [ ] 测试 release 已删除、版本文件已回退
- [ ] 上游 `release.yml` 在 fork 中 disabled，步骤记录在案
