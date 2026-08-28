# 如何手动触发变异测试（run_mutation）

这个 demo 的 `mutation-testing` job 默认不跑，只在你**手动勾选**时才运行。

## 步骤（GitHub Web UI）

1. 进入你的仓库页面，点击顶部 **Actions** 标签。
2. 在左侧列表里点击 **CI** 这个 workflow（就是本仓库的 `ci.yml`）。
3. 在右侧会看到一个 **Run workflow** 按钮，点击它 → 展开表单。
4. 表单里有一项：

   > **Run mutation testing (slow & non-blocking, ~15-60 min)**
   >
   > ☐  （这是一个勾选框）

   把这一项**打勾**。
5. 点击 **Run workflow** 确认触发。

## 结果

- 这次运行会执行 `lint-and-security` + `test-fast` + `coverage-report` + `mutation-testing` 全部 job；
- 如果你**不打勾**就点 Run workflow，则只跑前三个门禁 job，跳过变异测试；
- 普通 `push` / 开 `PR` 永远只跑前三个，不会触发变异测试。

## 想默认就跑？

把 `ci.yml` 里 `workflow_dispatch.inputs.run_mutation` 的 `default:` 从 `false` 改成 `true`
并提交即可。但**一般不建议**——容易忘了取消，每次手动触发都白跑 15–60 分钟。
保持 `false` + 用时手动勾，最省资源。
