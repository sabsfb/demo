# CI 实战 Demo：把「慢且非阻塞」的测试做成可手动勾选

> 一个**脱敏后的 GitHub Actions 模板**，展示一个在真实项目里踩出来的 CI 优化技巧：
> **让变异测试（mutation testing）这种又慢又非阻塞的 job，默认不跑，需要时手动勾一下才跑。**

这个仓库**不含任何公司内部信息**——没有内网域名、没有业务文档名、没有缺陷编号。
它脱胎于一个真实的内部文档预处理系统的 CI，但所有敏感内容都已替换为通用占位，可放心公开。

---

## 为什么需要这个 pattern

变异测试要跑全量变异体，动辄 **15–60 分钟**，而且本质是**质量参考指标，不是合并门禁**。
如果把它挂在 `push` / `pull_request` 上自动跑：

- 每一次普通提交都白烧 15–60 分钟 CI 时长；
- 它失败也不会真卡住合并（非阻塞），却天天在浪费算力。

正确做法：**默认不跑，按需手动触发。**

---

## 它怎么工作

核心只有两处改动（都在 `.github/workflows/ci.yml`）：

### 1. 给手动触发加一个开关

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
  workflow_dispatch:
    inputs:
      run_mutation:
        description: "Run mutation testing (slow & non-blocking, ~15-60 min)"
        type: boolean
        default: false
```

### 2. 给那个 job 加一个 `if` 条件

```yaml
  mutation-testing:
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'workflow_dispatch' && inputs.run_mutation == true }}
    timeout-minutes: 240
```

效果：

| 触发方式 | 变异测试跑吗 |
|---|---|
| 普通 `push` | ❌ 不跑 |
| 开 `pull_request` | ❌ 不跑 |
| 手动 `Run workflow` 且不勾选 | ❌ 不跑 |
| 手动 `Run workflow` **勾选 run_mutation** | ✅ 跑 |

因为 `mutation-testing` 没有 `needs:` 依赖，跳过它**不会影响** lint / 测试等门禁 job。

---

## 怎么手动触发（见 `docs/trigger-guide.md`）

1. 打开仓库 **Actions** 页
2. 左侧选 **CI** workflow
3. 点 **Run workflow**
4. 勾选 **Run mutation testing**
5. 再点 **Run workflow**

---

## 这个 demo 还顺手展示了什么

- **分层门禁（fail-fast）**：`lint-and-security` → `test-fast` → `coverage-report`，低级问题先拦。
- **`concurrency` 取消旧运行**：同一分支新提交进来时，自动取消上一次没跑完的 CI。
- **扩展点注释**：`ci.yml` 末尾给出了"需要 LibreOffice/ImageMagick 的集成测试 job"的范例写法（默认注释掉，保持 demo 轻量免费）。

---

## 适用场景

- 变异测试、大规模性能压测、重型 e2e 等**慢且非阻塞**的 job；
- 想保留"可审计的质量趋势记录"，但不想每次提交都付时间成本。

## 不适用

- 合并门禁类测试（单测、lint、安全扫描）——这些应该**每次都跑**，别用这个 pattern 跳过。

---

## 安全说明

本仓库为**完全脱敏的模板**：所有内部服务地址、业务文档类型、缺陷 ID 均已移除或泛化。
原始项目为某内部文档预处理系统的私有仓库，本 demo 仅复用其 CI 设计思路。
