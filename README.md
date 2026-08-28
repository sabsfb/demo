# 知识库文件预处理系统 · 测试工程展示（脱敏版）

> 一个**完全脱敏**的 GitHub Pages 静态站点，展示某电力行业知识库文件预处理系统的完整测试工程：
> 测试体系分层、CI/CD 架构、量化度量看板，以及**变异测试按需触发**这一核心 CI 优化技巧。
>
> 🔗 在线站点：`https://sabsfb.github.io/demo/`（启用 Pages 后生效）

本仓库**不含任何公司内部信息**——没有内网域名、没有真实业务文档名、没有安全缺陷利用细节、没有密钥。
它脱胎于一个真实的内部文档预处理系统的测试工程，但所有敏感内容都已泛化，可放心公开。

---

## 站点内容

| 页面 / 章节 | 内容 |
|---|---|
| `index.html` | 项目概述、测试体系、CI 架构图、度量看板、变异测试说明、安全声明 |
| `docs/test-strategy.md` | 脱敏测试策略：分层 / 已知问题 / 覆盖率 / 变异 / 度量 / 环境 |
| `docs/trigger-guide.md` | 如何手动勾选触发变异测试（图文步骤） |
| `.github/workflows/ci.yml` | 变异测试按需触发的 CI 模板（可直接复用） |

站点内导航锚点：`#overview` 项目概述 · `#system` 测试体系 · `#cicd` CI/CD 架构 · `#metrics` 度量看板 · `#mutation` 变异测试 · `#security` 安全声明

---

## CI 优化核心：变异测试按需触发

变异测试（mutation testing）要跑全量变异体，动辄 **15–60 分钟**，且本质是**质量参考指标，不是合并门禁**。
如果挂在 `push` / `pull_request` 上自动跑，每次普通提交都白烧大量 CI 时长。

正确做法：**默认不跑，按需手动勾选。** 核心两处改动（见 `ci.yml`）：

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
```yaml
  mutation-testing:
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'workflow_dispatch' && inputs.run_mutation == true }}
    timeout-minutes: 240
```

| 触发方式 | 变异测试跑吗 |
|---|---|
| 普通 `push` | ❌ 不跑 |
| 开 `pull_request` | ❌ 不跑 |
| 手动 `Run workflow` 不勾选 | ❌ 不跑 |
| 手动 `Run workflow` **勾选 run_mutation** | ✅ 跑 |

手动触发步骤见 [`docs/trigger-guide.md`](docs/trigger-guide.md)。

---

## 本地预览

纯静态站点，无需构建。直接双击 `index.html` 即可本地浏览器查看；或起一个本地服务：

```bash
python -m http.server 8080
# 浏览器打开 http://localhost:8080
```

---

## 启用 GitHub Pages（只需做一次）

1. 进入仓库 **Settings** → 左侧 **Pages**；
2. **Build and deployment** → Source 选 **Deploy from a branch**；
3. Branch 选 **main**，目录选 **/ (root)**，点 **Save**；
4. 等待约 1 分钟，访问 `https://sabsfb.github.io/demo/` 即可看到本站。

> 之后每次 `git push` 更新内容，Pages 会自动重新发布，无需额外操作。

---

## 安全说明

本仓库为**完全脱敏的展示**：所有内部服务地址、真实业务文档类型、安全缺陷利用细节均已移除或泛化。
原始项目（含完整源码、测试资产、综合报告）保存在**私有仓库**，不对外公开。本仓库仅复用其测试工程设计与方法论。
