# OpenIM 批量 Fork 与自动同步


统一管理从 [openimsdk](https://github.com/openimsdk) Fork 到本仓库所属账号的公开仓库。


## 自动化行为


- 每次运行重新发现上游的全部公开仓库，包含已归档仓库和上游本身的 Fork；空仓库、已禁用仓库会记录并跳过。
- 首次 Fork 包含上游当时已有的分支。后续定时同步只更新上游当前默认分支对应的同名分支。
- 每小时第 7、22、37、52 分钟（UTC）计划执行一次，也可在 Actions 中手动执行。GitHub 调度可能延迟，这不是实时推送。
- 更新使用 GitHub 服务端强制检查的快进方式（`force: false`）。同名但无关的仓库、已归档目标仓库、出现独立提交或分叉的分支会报告错误，保留用户现有内容。
- GitHub 创建 Fork 是异步操作。`CREATED_PENDING` 表示创建请求已接受，后续运行会验证并同步。
- Tags、Releases、发布附件、Issues 和 PR 不做持续镜像；上游删除的分支或仓库也不会从目标删除。


## 启用


1. 在本仓库的 **Settings → Secrets and variables → Actions** 添加 Repository secret：`OPENIM_SYNC_TOKEN`。
2. 使用当前仓库所有者的 classic PAT，权限仅为 `public_repo` 和 `workflow`。可从[权限预填页面](https://github.com/settings/tokens/new?scopes=public_repo,workflow)创建，核对未勾选完整 `repo` 权限，并在生成后的列表再次核对实际 scopes。此类 token 可操作该账号有权限的公开仓库，不只限于 OpenIM。
3. 打开 **Actions → Fork and sync all OpenIM repositories → Run workflow**。勾选 `Preview only` 可预览本轮计划而不修改仓库；取消勾选才实际 Fork 和同步。
4. 在每次运行的 **Summary** 查看逐仓库结果。首次创建后可再手动运行一轮，核对各分支是否 `UP_TO_DATE`。


工作流使用单独的 token，因为本仓库自动提供的 `GITHUB_TOKEN` 无法跨仓库完成此任务。Token 只放在 Actions secret 中，不能写入工作流或 README。


## 维护


- 本次令牌按账号所有者要求设置为 **No expiration（无到期时间）**。如果令牌被撤销或失效，需要更新 `OPENIM_SYNC_TOKEN`。
- 每个 UTC 日期最多向本控制仓库的 `sync-status.json` 提交一次实际运行摘要，包含检查时间、运行链接、发现的仓库数和结果计数。预览不写入，也不会在各个 Fork 中加入这个文件。这是每日抽样记录；最新结果以 Actions 为准。
- 每日状态提交让控制仓库保持实际更新，减少连续 60 天没有活动而被 GitHub 停用的情况。如果平台已经停用任务，仍需在 Actions 中重新启用。
- 这个控制任务不检出或执行上游代码，也不会主动启用各个 Fork 内的 Actions；如果某个目标仓库已启用 Actions，PAT 推送可能触发该仓库原有的工作流。
- 出错时先查看 Summary 和日志。不要为解决同步冲突强制覆盖自己的代码；可把自己的开发放在独立分支上。


## 官方参考


- [Fork API](https://docs.github.com/en/rest/repos/forks#create-a-fork)
- [Git references API](https://docs.github.com/en/rest/git/refs#update-a-reference)
- [Actions schedule](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)
- [GITHUB_TOKEN 的仓库范围](https://docs.github.com/en/actions/concepts/security/github_token)
- [管理个人访问令牌](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

