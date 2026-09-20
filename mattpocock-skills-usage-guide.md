# Matt Pocock Skills 使用说明书

> 适用环境：Claude Code  
> 本机插件版本：`mattpocock-skills 1.2.3`  
> 核验内容 commit：`5b15a47f2d7150f545fbcacbfe381787fc0230dc`  
> 本机插件目录：`C:\Users\z00053163\.claude\plugins\cache\claude-plugins-official\mattpocock-skills\1.2.3`

---

## 目录

1. [这套 Skills 是什么](#1-这套-skills-是什么)
2. [实际安装状态与数量口径](#2-实际安装状态与数量口径)
3. [Claude Code 中如何调用](#3-claude-code-中如何调用)
4. [第一次在项目中使用](#4-第一次在项目中使用)
5. [安全边界与推荐调用模板](#5-安全边界与推荐调用模板)
6. [25 个已导出技能逐项说明](#6-25-个已导出技能逐项说明)
7. [任务决策树](#7-任务决策树)
8. [端到端工作流配方](#8-端到端工作流配方)
9. [依赖与副作用速查](#9-依赖与副作用速查)
10. [11 个存在但未导出的技能](#10-11-个存在但未导出的技能)
11. [新手学习顺序](#11-新手学习顺序)
12. [故障排查](#12-故障排查)
13. [最终使用原则](#13-最终使用原则)

---

# 1. 这套 Skills 是什么

Matt Pocock Skills 不是一组单纯的提示词模板，而是一套面向软件工程全过程的工作方法。它主要处理五类问题。

## 1.1 需求和设计澄清

- `grill-with-docs`
- `grill-me`
- `grilling`
- `domain-modeling`
- `prototype`
- `research`

## 1.2 将讨论转为可执行工作

- `to-spec`
- `to-tickets`
- `wayfinder`
- `to-questionnaire`

## 1.3 实现与测试

- `implement`
- `tdd`
- `diagnosing-bugs`

## 1.4 质量与维护

- `code-review`
- `codebase-design`
- `improve-codebase-architecture`
- `resolving-merge-conflicts`
- `triage`

## 1.5 跨会话、教学和 Agent 文档

- `handoff`
- `teach`
- `wait-what`
- `wizard`
- `writing-for-agents`
- `ask-matt`
- `setup-matt-pocock-skills`

它的核心理念可以概括为：

> 先消除未知和歧义，再写规格；先把规格切成可独立验证的小任务，再让每个会话完成一个任务；实现阶段建立快速反馈环，最后分别检查代码质量和需求符合度。

它不是要求每个任务都走完整流程。正确方式是根据任务当前的不确定性，选择最轻、足够解决问题的技能链。

---

# 2. 实际安装状态与数量口径

本机安装目录中共有 **36 个 `SKILL.md`**，但并非全部由插件导出。

| 类别 | 数量 | 含义 |
|---|---:|---|
| 插件实际导出的技能 | 25 | 都可以由用户显式调用 |
| 仅用户显式调用 | 14 | 带 `disable-model-invocation: true` |
| 模型也可自动调用 | 11 | 可以显式调用，也可能被自然语言自动触发 |
| 磁盘存在但未导出 | 11 | 当前不能作为该插件的命令调用 |

## 2.1 14 个仅用户显式调用的技能

1. `ask-matt`
2. `grill-with-docs`
3. `triage`
4. `improve-codebase-architecture`
5. `setup-matt-pocock-skills`
6. `to-spec`
7. `to-tickets`
8. `wayfinder`
9. `implement`
10. `grill-me`
11. `handoff`
12. `teach`
13. `to-questionnaire`
14. `wait-what`

普通自然语言不会可靠地启动它们。需要严格执行对应流程时，应输入完整 slash 命令。

## 2.2 11 个允许模型自动调用的技能

1. `diagnosing-bugs`
2. `tdd`
3. `prototype`
4. `research`
5. `domain-modeling`
6. `codebase-design`
7. `code-review`
8. `resolving-merge-conflicts`
9. `wizard`
10. `grilling`
11. `writing-for-agents`

这 11 个既可以显式调用，也可能根据自然语言由 Claude 自动选择。

## 2.3 插件没有内置哪些组件

当前插件清单只声明了 Skills，没有插件级：

- commands 目录
- agents 目录
- hooks
- MCP server
- LSP server

各技能目录中的 `agents/openai.yaml` 是 Agent Skills 客户端的界面或调用策略元数据，不是 Claude Code 的预置子代理。运行期间出现的子代理，是 Skill 动态启动的 worker。

---

# 3. Claude Code 中如何调用

## 3.1 推荐使用完整命名空间

```text
/mattpocock-skills:<skill-name>
```

例如：

```text
/mattpocock-skills:tdd
/mattpocock-skills:diagnosing-bugs
/mattpocock-skills:code-review
```

有些环境可能接受 `/tdd` 等裸名称，但安装多个插件后容易发生重名或歧义，因此建议始终使用完整命名空间。

## 3.2 命令后直接附带任务

```text
/mattpocock-skills:tdd 为订单取消接口实现测试驱动开发
```

更推荐提供结构化上下文：

```text
/mattpocock-skills:tdd

目标：
为订单取消接口增加“已发货订单不可取消”的行为。

测试命令：
pnpm test order-cancellation.test.ts

测试入口：
POST /api/orders/:id/cancel

允许修改：
src/orders/
tests/orders/

禁止修改：
数据库 schema
支付接口

要求：
先确认测试 seam，再写第一个失败测试。
一次只做一个 red-green slice。
```

## 3.3 自然语言触发

下面的描述可能触发 `diagnosing-bugs`：

```text
这个问题只在线上偶发出现，请先建立可重复反馈环，不要直接猜原因。
```

但自动触发由模型判断，不是确定性的命令路由。需要严格执行时，显式调用：

```text
/mattpocock-skills:diagnosing-bugs
```

## 3.4 技能之间如何组合

主要组合关系如下：

| 上层技能 | 内部组合或后续能力 |
|---|---|
| `grill-me` | `grilling` |
| `grill-with-docs` | `grilling` + `domain-modeling` |
| `triage` | 必要时使用 `grilling` + `domain-modeling` |
| `improve-codebase-architecture` | `codebase-design`，之后可能使用 `grilling`、`domain-modeling` |
| `tdd` | seam 不清楚时使用 `codebase-design` |
| `wayfinder` | 根据 ticket 类型使用 `research`、`prototype`、`grilling`、`domain-modeling` |
| `implement` | 使用 `tdd`，完成后使用 `code-review` |
| `codebase-design` | Design It Twice 时启动多个并行设计子代理 |

重要限制：

- user-invoked skill 可以调用 model-invoked skill。
- user-invoked skill 不能直接替用户启动另一个 user-invoked skill。
- `ask-matt` 可以告诉你下一步应该输入什么，但不会替你启动 `to-spec`、`implement` 等手动技能。

## 3.5 权限模型

这些 Skill 不会自动获得额外权限。是否能够：

- 执行 Bash
- 修改文件
- 执行 Git 操作
- 创建或关闭 issue
- 打开浏览器
- 调用 MCP
- 写入 secret

取决于当前 Claude Code 权限配置。

子代理也不能绕过主会话权限。

---

# 4. 第一次在项目中使用

如果要使用完整工程流程，建议从项目根目录启动 Claude Code，然后运行一次：

```text
/mattpocock-skills:setup-matt-pocock-skills
```

推荐提示词：

```text
/mattpocock-skills:setup-matt-pocock-skills

这个项目使用 GitHub Issues。
请检查现有 CLAUDE.md、AGENTS.md 和 docs 目录。

要求：
1. 先展示准备创建或修改的文件。
2. 不要覆盖已有 Agent 指令。
3. 使用单一 domain context，除非代码库确实是大型 monorepo。
4. 任何文件写入前先让我确认。
5. 不要创建远端 issue。
```

初始化可能生成：

```text
docs/agents/issue-tracker.md
docs/agents/domain.md
docs/agents/triage-labels.md
```

并可能向已有的 `CLAUDE.md` 或 `AGENTS.md` 添加简短指针。

当前源码内建支持：

- GitHub Issues：通过 `gh`
- GitLab Issues：通过 `glab`
- 本地 Markdown tracker：通过 `.scratch/`
- 其他 tracker：保存用户提供的自定义流程

README 可能提到 Linear，但当前安装版没有 Linear 专用模板，应按 Other tracker 处理。

---

# 5. 安全边界与推荐调用模板

## 5.1 建议附加的安全约束

涉及高影响操作时，建议附加：

```text
安全约束：

- 任何 git commit、push、merge、rebase、branch 创建前先让我确认。
- 任何远端 issue、PR、comment、label、close 操作前先预览。
- 不要写入或显示 secret。
- 不要修改插件 cache。
- 不要安装新依赖，除非先说明原因并得到确认。
- 可以执行只读检查和本地测试。
```

## 5.2 通用调用模板

```text
/mattpocock-skills:<skill-name>

目标：
<最终想达到什么>

当前状态：
<已经完成什么，卡在哪里>

已有材料：
<issue、spec、日志、文档、URL、commit>

允许修改：
<目录、文件、远端对象>

禁止修改：
<明确边界>

可运行命令：
<test、typecheck、lint、build>

外部系统：
<GitHub、GitLab、数据库、浏览器、第三方服务>

需要我确认的节点：
<写文件、创建 issue、commit、merge 等>

停止条件：
<做到什么就停，不继续扩范围>
```

---

# 6. 25 个已导出技能逐项说明

## 6.1 `ask-matt`：技能路由器

**调用类型：仅用户显式调用。**

### 适合

- 第一次使用这套技能。
- 不知道该走 grilling、spec、tickets、TDD、review 还是 wayfinder。
- 不知道该继续当前上下文，还是 `/clear`、`/compact` 或 `handoff`。

### 示例

```text
/mattpocock-skills:ask-matt

我要重构一个已运行五年的计费系统。
需求还不完全明确，预计需要多个会话完成。
现在有 GitHub repo，但没有 spec。
告诉我应该按什么顺序使用这些 skills。
```

### 会做什么

1. 判断任务阶段。
2. 判断是否可在一个会话完成。
3. 推荐技能顺序。
4. 建议上下文切换策略。

### 不会做什么

它只路由，通常不会直接启动下一个手动 Skill，也不会直接实现。

---

## 6.2 `diagnosing-bugs`：困难 Bug 诊断

**调用类型：可自动调用。**

### 适合

- 偶发失败。
- 性能回归。
- 线上问题。
- 难以复现。
- 已尝试明显修复但无效。

### 不适合

- 明显拼写错误。
- 根因一眼可见的小问题。
- 没有环境或材料，却要求 Claude 凭代码猜测。

### 示例

```text
/mattpocock-skills:diagnosing-bugs

症状：
订单提交约 1% 会重复扣款。

期望：
一个 idempotency key 最多成功扣款一次。

环境：
Node 22、PostgreSQL 17。

最后正常版本：
commit abc123。

可运行命令：
pnpm test payments
pnpm test:e2e checkout

要求：
1. 先建立能捕获准确症状的反馈环。
2. 不要先改业务代码。
3. 给出 3～5 个可证伪假设。
4. 一次只改变一个变量。
5. 找到根因后先加回归测试。
6. 删除临时诊断日志。
```

### 流程

1. 脱敏日志和数据。
2. 建立 tight feedback loop。
3. 复现并最小化。
4. 提出多个可证伪假设。
5. 增加针对性 probe。
6. 在正确 seam 上写失败回归测试。
7. 修复根因。
8. 重跑原始场景。
9. 清理临时诊断代码。

### 停止条件

如果无法构造已运行、能捕获准确症状、速度足够快且结果确定的命令，应停止猜测并索取环境、脱敏日志、HAR、trace、样例数据或 instrumentation 权限。

### 推荐串联

```text
diagnosing-bugs → tdd → code-review
```

---

## 6.3 `grill-with-docs`：带项目文档的设计访谈

**调用类型：仅用户显式调用。**

### 适合

- 新功能还没有想清楚。
- 项目术语混乱。
- 希望后续 Agent 理解今天讨论的理由。
- 准备进入 `to-spec`。

### 示例

```text
/mattpocock-skills:grill-with-docs

我要设计“订单部分取消”。
请重点追问：
- 已发货和未发货商品怎么处理
- 退款失败怎么办
- 库存什么时候回补
- 并发取消如何避免重复退款
- “取消”“退款”“作废”的定义是否相同

不要开始实现。
每轮给出你的推荐答案。
术语落定后更新 CONTEXT.md。
只有真正难逆转的决定才写 ADR。
```

### 内部组合

```text
grilling + domain-modeling
```

### 可能产物

```text
CONTEXT.md
docs/adr/0001-*.md
```

它不会自动生成 spec 或实现代码。讨论完成后应手动调用 `to-spec`。

---

## 6.4 `triage`：Issue 与外部 PR 分诊

**调用类型：仅用户显式调用。**

### 适合

- 检查 bug issue 是否真实。
- 判断信息是否充足。
- 验证外部 PR。
- 将 issue 整理为 agent-ready。

### 示例

```text
/mattpocock-skills:triage

检查 GitHub issue #42。

要求：
1. 读取完整 issue、评论和相关代码。
2. 如果是 bug，尝试复现。
3. 推荐 category 和 state。
4. 任何 comment、label、close 操作前先让我确认。
5. 不修改业务代码。
```

### 可能副作用

- 添加评论。
- 修改 label。
- 关闭 issue/PR。
- 写 agent brief。
- 写 `.out-of-scope/*.md`。

### 前置条件

- 已运行 setup。
- GitHub 使用 `gh` 并完成认证。
- GitLab 使用 `glab` 并完成认证。

不要对 `to-tickets` 刚生成的 ticket 再做 triage。

---

## 6.5 `improve-codebase-architecture`：寻找架构改进候选

**调用类型：仅用户显式调用。**

### 适合

- 不知道哪里最值得重构。
- 某些区域频繁一起修改。
- 调用者需要理解过多内部细节。
- 测试必须绕过公共接口。

### 示例

```text
/mattpocock-skills:improve-codebase-architecture

只分析订单定价区域。
找出最多 3 个证据最强的 deepening 候选。
可以读取 git history，但先出报告，不改代码。
```

### 流程与产物

1. 加载 `codebase-design` 术语。
2. 查找频繁修改热点。
3. 启动探索子代理。
4. 对候选做 deletion test。
5. 在临时目录生成 HTML 报告。
6. 打开报告，让用户选择候选。

HTML 可能使用 Tailwind 与 Mermaid CDN，离线时样式或图表可能缺失。

---

## 6.6 `setup-matt-pocock-skills`：项目初始化

**调用类型：仅用户显式调用。**

每个 repo 通常只运行一次。它配置 tracker、领域文档和 Agent 指针。不要每个会话重复运行，也不要允许它覆盖已有 `CLAUDE.md` 或 `AGENTS.md`。

---

## 6.7 `tdd`：按测试 Seam 小步开发

**调用类型：可自动调用。**

### 适合

- 新增可观察行为。
- bug 回归测试。
- 明确要求 test-first。
- 有可执行测试环境。

### 示例

```text
/mattpocock-skills:tdd

行为：
过期 access token 必须返回 401，且不能调用订单服务。

公共入口：
POST /api/orders

测试命令：
pnpm test auth.test.ts

要求：
1. 先给出候选测试 seam。
2. 等我确认 seam 后再写测试。
3. 一次只写一个失败测试。
4. 实际证明测试先红。
5. 只写使当前测试变绿的最少代码。
6. 不 mock 项目内部模块。
```

### 正确循环

```text
测试 1 红 → 最小实现 → 测试 1 绿
→ 测试 2 红 → 最小实现 → 测试 2 绿
```

### 好的测试 Seam

- HTTP endpoint
- 公共函数
- CLI command
- package public API
- 消息消费者
- 用户界面行为

### 常见误用

- 测私有函数。
- mock 自己的内部模块。
- 测试重复实现生产算法。
- 一次写完全部测试再实现。
- 接口尚未确定就开始写测试。

Seam 不清楚时先用 `codebase-design`。

---

## 6.8 `to-spec`：把已完成讨论发布成规格

**调用类型：仅用户显式调用。**

### 适合

- grilling 已结束。
- 关键需求已决定。
- 需要稳定的后续实现依据。

### 示例

```text
/mattpocock-skills:to-spec

把当前关于“订单部分取消”的讨论整理成 spec。

要求：
- 不加入未决定的能力
- 包含 Problem、Solution、User Stories
- 包含 Implementation Decisions
- 包含 Testing Decisions
- 包含 Out of Scope
- 发布前让我确认测试 seams
- 创建远端 issue 前先预览正文
```

### 输出

GitHub/GitLab：创建 issue，可能添加 `ready-for-agent`。

Local tracker：

```text
.scratch/<feature>/spec.md
```

它不是从零开始的需求访谈工具。需求仍不清楚时先用 `grill-with-docs`。

---

## 6.9 `to-tickets`：拆成单会话垂直切片

**调用类型：仅用户显式调用。**

### 适合

- 一个功能跨多个会话。
- 需要多人或多 Agent 并行。
- 需要明确依赖图。
- 有大范围迁移。

### 示例

```text
/mattpocock-skills:to-tickets

基于 GitHub issue #120 拆分实现 tickets。

要求：
- 每个 ticket 可在一个新会话中完成
- 每个 ticket 有可独立验证的用户行为
- 使用 vertical slice
- 不按 database/API/UI 横向拆
- 明确 blocking edges
- 发布前展示依赖图
```

### 好的切片

```text
Ticket 1：用户可以提交取消请求并看到结果
Ticket 2：已发货商品被拒绝并给出原因
Ticket 3：退款失败时订单保持一致状态
```

### 不好的横向切片

```text
Ticket 1：建数据库表
Ticket 2：实现 API
Ticket 3：实现前端
```

### 输出

远端 tracker：多个 issue、依赖关系、labels。

Local tracker：

```text
.scratch/<feature>/issues/01-<slug>.md
```

---

## 6.10 `wayfinder`：大型未知工作的决策地图

**调用类型：仅用户显式调用。**

### 适合

- 工作明显跨多个会话。
- 关键路线仍未知。
- 有多个相互依赖的决策问题。
- 需要研究、原型和多人协作。

### 不适合

- 一个会话可完成。
- 已经有明确 spec。
- 只是想列 TODO。

### 示例

```text
/mattpocock-skills:wayfinder

Destination：
将旧计费系统迁移到事件驱动架构，并保持双写期间账单一致。

已知决定：
- PostgreSQL 保留
- 不更换支付供应商
- 允许三个月迁移期

当前未知：
- 事件 schema
- 重放语义
- 双写一致性
- 回滚边界

Tracker：GitHub Issues

本次只 chart map：
- 创建决策地图
- 只创建当前能精确定义的 decision tickets
- 不实现迁移
- 任何远端写入前先预览
```

### Chart map 模式

1. 定义 destination。
2. 找出已知决定。
3. 找出 frontier 与 fog。
4. 创建当前可精确定义的 decision tickets。
5. 建立 blocking edges。
6. 可并行启动 research tickets。
7. 停止，不直接实现。

### Work through map 模式

1. 选择第一个可执行 frontier ticket。
2. claim。
3. 一个会话只解决一个非 research ticket。
4. 把答案写回 tracker。
5. 关闭 ticket。
6. 更新 Decisions so far。
7. 根据新知识创建下一层 ticket。

当路线清晰后停止 wayfinder，转为：

```text
to-spec → to-tickets → implement
```

---

## 6.11 `implement`：实现一个规格化任务

**调用类型：仅用户显式调用。**

### 适合

- 已有明确 spec 或 agent-ready ticket。
- 范围可在当前会话完成。
- 有测试入口。

### 示例

```text
/mattpocock-skills:implement

实现 GitHub issue #142。

约束：
- 只处理该 issue 的 acceptance criteria
- 使用 /mattpocock-skills:tdd
- 测试命令：pnpm test order-cancellation.test.ts
- 完成后运行 typecheck、lint 和完整测试
- 使用 /mattpocock-skills:code-review
- 不要自动 commit；最后展示 diff 和测试结果
```

### 流程

1. 读取 ticket/spec。
2. 确认测试 seam。
3. 使用 TDD。
4. 频繁运行局部测试和 typecheck。
5. 最后运行完整测试。
6. 执行 code review。
7. 按原技能默认流程可能提交当前分支。

如果不希望提交，必须明确写：

```text
不要 commit；实现完成后停在 working tree。
```

---

## 6.12 `prototype`：用原型回答设计问题

**调用类型：可自动调用。**

### LOGIC 原型

适合状态机、业务规则和流程边界。

```text
/mattpocock-skills:prototype

唯一要回答的问题：
订单部分退款后，是否还能取消剩余未发货商品？

使用 LOGIC 模式。
生成一个可双击打开的单 HTML。
包含正常路径、重复退款、退款失败和非法状态转换。
不要接数据库或真实支付服务。
```

### UI 原型

```text
/mattpocock-skills:prototype

为现有设置页面做 3 个结构明显不同的布局。
使用 ?variant=A、B、C 切换。
不接真实写操作。
三个方案不能只是换颜色。
只回答“设置项应如何分组”。
```

### 重要限制

Prototype 允许无完整测试、最小错误处理、stub 数据和临时结构，不能直接作为生产实现上线。

正确流程：

```text
prototype → 用户验证 → 记录结论
→ 丢弃或保存原型分支 → 用 tdd 重写生产实现
```

---

## 6.13 `research`：后台一手资料研究

**调用类型：可自动调用。**

### 适合

- 官方 API 文档。
- 标准、规范、源码行为。
- 需要跨会话保留研究结果。
- 主会话还需要继续其他工作。

### 示例

```text
/mattpocock-skills:research

研究问题：
PostgreSQL 18 logical replication slot 的故障恢复语义。

来源要求：
- 只使用 PostgreSQL 官方文档和源码
- 不使用博客或搜索摘要

必须回答：
- 哪些状态会持久化
- crash 后是否可能重放
- failover slot 的限制
- 不同版本行为差异

输出：
docs/research/postgres-18-replication-slots.md

每项关键结论都给直接引用。
```

### 流程与副作用

1. 启动后台 agent。
2. 查一手资料。
3. 写 Markdown。
4. 返回报告路径。

会产生额外的子代理上下文成本，不适合查询一个简单事实。

---

## 6.14 `domain-modeling`：领域语言和 ADR

**调用类型：可自动调用。**

### 适合

- 多个术语被混用。
- 业务边界不清。
- 用户说法和代码行为矛盾。
- 需要创建或更新 `CONTEXT.md`。

### 示例

```text
/mattpocock-skills:domain-modeling

检查“客户”“账户”“用户”是否被混用。
阅读相关业务代码，构造至少 3 个边缘场景。
选择 canonical term，术语确定后更新 CONTEXT.md。
只有满足 ADR 条件时才创建 ADR。
```

### `CONTEXT.md` 应保存

- 领域实体定义。
- 实体关系。
- 业务不变量。
- 标准术语。
- 容易误解的区别。

不应保存容易过期的实现路径、类名清单和普通编程概念。

### ADR 条件

1. 决定难以逆转。
2. 将来缺乏背景的人会困惑。
3. 存在真实备选方案与取舍。

---

## 6.15 `codebase-design`：Deep Module 与 Seam 设计

**调用类型：可自动调用。**

### 适合

- 模块接口过宽。
- 调用者需要了解过多内部细节。
- 不知道测试 seam 应放在哪里。
- adapter 是否值得存在不明确。

### 示例

```text
/mattpocock-skills:codebase-design

评估订单定价模块。
分析所有调用者、公共接口、隐含不变量、错误模式和测试入口。
使用 deletion test 判断模块深度。
只给设计建议，不修改代码。
```

### 核心术语

- Module：提供能力的代码单位。
- Interface：调用者必须知道的全部内容，不只是 TypeScript `interface`。
- Implementation：模块内部细节。
- Depth：功能能力相对于接口复杂度。
- Seam：可独立替换、测试或观察的边界。
- Adapter：外部系统到内部模型的转换边界。
- Leverage：少量接口提供的能力。
- Locality：理解改动时需要跨越多少位置。

### Design It Twice

需要比较多个合理方案时，可能启动至少三个并行子代理，提出结构上真正不同的方案，再按 depth、locality 和 seam placement 评价。

---

## 6.16 `code-review`：Standards 与 Spec 双轴评审

**调用类型：可自动调用。**

### 两个评审轴

1. Standards：是否符合仓库规范、架构和测试原则。
2. Spec：是否正确实现需求，是否遗漏或扩张范围。

通常由两个并行子代理分别完成。

### 示例

```text
/mattpocock-skills:code-review

固定点：
origin/main

规格来源：
GitHub issue #142

规范来源：
CLAUDE.md
CONTRIBUTING.md

范围：
- origin/main...HEAD
- staged
- unstaged
- untracked files

要求：
- 只报告问题
- 不修改代码
- Standards 和 Spec 分开输出
- 每项给文件和行号
```

### 重要注意

```bash
git diff origin/main...HEAD
```

不包含 staged、unstaged 和 untracked 文件。评审 WIP 时必须显式要求额外检查这些内容。

该 Skill 默认只评审，不自动修复。

---

## 6.17 `resolving-merge-conflicts`：按原始意图解决冲突

**调用类型：可自动调用。**

只应在当前确实处于 merge/rebase conflict 时使用。

### 示例

```text
/mattpocock-skills:resolving-merge-conflicts

当前正在把 feature/payments rebase 到 main。
先列出冲突文件，阅读双方 commits、PR 和 issue。
对每个 hunk 说明双方原始意图。
不机械选择 ours/theirs，不发明新业务行为。
修改后运行 typecheck、tests、format。
执行 rebase --continue 前让我确认。
```

### 可能副作用

- 修改冲突文件。
- `git add`。
- merge commit。
- `rebase --continue`。

源码倾向“始终解决，不 abort”，但若基线错误、原始意图不明或存在数据丢失风险，应该先停下来确认，而不是机械遵循。

---

## 6.18 `wizard`：人工操作 Bash 向导

**调用类型：可自动调用。**

### 适合

- 必须人工登录第三方 Dashboard。
- 收集 API key。
- 写 `.env`。
- 设置 GitHub Actions secret。
- 一次性迁移或 cutover。

### 示例

```text
/mattpocock-skills:wizard

为 Stripe 测试环境生成交互式配置向导。
收集 STRIPE_PUBLISHABLE_KEY、STRIPE_SECRET_KEY 和 STRIPE_WEBHOOK_SECRET。
写入本地 .env 和 GitHub Actions secrets。
先验证 .env 已进入 .gitignore。
secret 不打印到终端。
不可逆步骤必须确认。
只生成并静态检查，不实际运行。
```

### 验证

```bash
bash -n script.sh
shellcheck script.sh
```

### Windows 注意

它生成 Bash，不是 PowerShell。Windows 下使用 Git Bash、WSL 或 Claude Code 可用的 Bash 环境。

---

## 6.19 `grill-me`：无状态强力访谈

**调用类型：仅用户显式调用。**

适合没有 repo、不需要保存 `CONTEXT.md` 或 ADR 的计划、商业、产品和技术决策。

```text
/mattpocock-skills:grill-me

我要判断是否把内部开发工具商业化。
重点挑战目标客户、付费意愿、支持成本、安全责任和停止条件。
不要替我拍板。
每轮只问当前可以回答的问题，并给出推荐。
```

它默认是纯对话，不写文件。

---

## 6.20 `grilling`：访谈原语

**调用类型：可自动调用。**

它把计划表示为有依赖关系的设计树，每轮只询问当前已解锁的 frontier，不会一次性抛出大量互相依赖的问题。

```text
/mattpocock-skills:grilling

压力测试我的 webhook 重试设计。
每轮只问当前 frontier。
每题给出推荐答案。
环境事实自行调查，不要问我。
frontier 为空时总结共同理解。
```

一般优先从 `grill-me` 或 `grill-with-docs` 进入，而不是直接使用底层 `grilling`。

---

## 6.21 `handoff`：跨会话交接文件

**调用类型：仅用户显式调用。**

### 适合

- 切换 Agent 或 harness。
- 切换 worktree 或 repo。
- 交给同事。
- 当前会话过长但后续仍需继续。

### 示例

```text
/mattpocock-skills:handoff

下一会话将在另一个 worktree 实现 issue #142。
只保留尚未完成的目标、已确认约束、必要路径、测试状态和建议 skills。
排除 secret、完整 diff 和已有文档全文。
```

### 输出

通常写到系统临时目录 `%TEMP%`，不写当前 workspace。

### 与其他上下文工具的选择

| 需求 | 使用方式 |
|---|---|
| 当前会话继续，只压缩上下文 | `/compact` |
| 无需保留当前讨论 | `/clear` |
| 新会话、新目录、新 Agent 接手 | `handoff` |
| 只需独立调查 | 后台子代理 |

Handoff 是有损压缩，不要把它当每个阶段必做的仪式。

---

## 6.22 `teach`：长期教学工作区

**调用类型：仅用户显式调用。**

### 适合

- 系统学习一个主题。
- 预计多个会话。
- 希望包含练习、复习和学习记录。

### 示例

```text
/mattpocock-skills:teach TypeScript 类型体操

现实目标：
6 周后能够独立维护团队的复杂泛型工具库。

当前水平：
熟悉基础泛型，不熟悉 conditional types 和 inference。

时间：
每次 20 分钟，每周 4 次。

要求：
- 每节课包含主动回忆
- 根据学习记录调整难度
- 使用官方 TypeScript 文档
- 不要只生成漂亮 HTML
```

### 可能生成

```text
MISSION.md
RESOURCES.md
NOTES.md
lessons/*.html
reference/*.html
learning-records/*.md
assets/*
```

最好在独立教学目录运行，不要在产品 repo 根目录生成整套课程文件。

---

## 6.23 `to-questionnaire`：给第三方的异步问卷

**调用类型：仅用户显式调用。**

当真正知道答案的人是产品负责人、客户、安全团队、法务、供应商或运维时使用。

```text
/mattpocock-skills:to-questionnaire

收件人：公司安全负责人。
需要获得：数据保留期限、允许部署区域、审计日志要求、删除请求 SLA、数据出境限制。
用途：完成客户数据平台架构规格。
填写时间不超过 20 分钟。
最重要问题放前面，每个问题只问一个概念。
```

输出：

```text
to-questionnaire-<slug>.md
```

选择原则：

```text
可查事实 → research
问当前用户 → grilling
问第三方 → to-questionnaire
```

---

## 6.24 `wait-what`：重新解释上一条

**调用类型：仅用户显式调用。**

```text
/mattpocock-skills:wait-what
```

也可以指定范围：

```text
/mattpocock-skills:wait-what
重新解释“测试 seam”，并结合当前订单模块举例。
```

它不会重新规划项目、研究或实现，只负责补背景、减少术语和讲清上一条。

---

## 6.25 `writing-for-agents`：编写 Agent 文档

**调用类型：可自动调用。**

### 适合

- 创建或修改 `SKILL.md`。
- 修改 `CLAUDE.md`、`AGENTS.md`。
- 优化 Skill 触发条件。
- 拆分过长 Agent 指令。

### 示例：审查 `CLAUDE.md`

```text
/mattpocock-skills:writing-for-agents

审查项目 CLAUDE.md。
删除可从 package.json、目录树和代码直接推导的信息。
保留非标准约定、风险和决策理由。
先给修改计划，不要直接编辑。
```

### 示例：创建 Skill

```text
/mattpocock-skills:writing-for-agents

设计一个“发布预检查”skill。
先判断它应是 model-invoked 还是 user-invoked。
明确触发条件、不触发条件、步骤、引用文件和可验证完成标准。
```

### 原则

- 只保留真正改变 Agent 行为的规则。
- 不复制代码库可直接查询的信息。
- 明确 trigger、sequence 和 completion criteria。
- 只在真正的分支或顺序边界拆分文件。

这些文档会影响以后所有 Agent 会话，属于高影响持久化修改。

---

# 7. 任务决策树

```text
不知道该用什么
└─ ask-matt

需求没想清
├─ 有 repo，需要保存术语或 ADR
│  └─ grill-with-docs
└─ 没有 repo，只做对话
   └─ grill-me

缺少外部事实
└─ research

缺少第三方业务答案
└─ to-questionnaire

只有运行起来才能判断
├─ 状态、规则、流程
│  └─ prototype LOGIC
└─ UI 布局或交互
   └─ prototype UI

讨论已经清楚
├─ 一个会话可完成
│  └─ implement
├─ 多会话，路线清楚
│  └─ to-spec → to-tickets → implement
└─ 多会话，路线仍未知
   └─ wayfinder → to-spec → to-tickets → implement

代码出错
├─ 简单且根因明确
│  └─ 直接修，必要时 tdd
└─ 偶发、难复现、性能回归
   └─ diagnosing-bugs

想改善架构
├─ 已知道目标模块
│  └─ codebase-design
└─ 不知道哪里最值得改
   └─ improve-codebase-architecture

Review 分支或 PR
└─ code-review

已经发生 merge/rebase conflict
└─ resolving-merge-conflicts

必须由人操作 Dashboard、secret 或 cutover
└─ wizard

需要切换会话、目录或 Agent
└─ handoff

上一条没听懂
└─ wait-what

编写 SKILL.md、CLAUDE.md、AGENTS.md
└─ writing-for-agents

长期系统学习
└─ teach
```

---

# 8. 端到端工作流配方

## 8.1 小功能：一个会话可完成

```text
setup（每个 repo 第一次）
→ grill-with-docs
→ implement
→ tdd
→ code-review
```

如果需求已经非常明确，可直接：

```text
tdd → code-review
```

## 8.2 中型功能

```text
grill-with-docs
→ to-spec
→ implement
→ code-review
```

不一定需要拆 tickets。

## 8.3 大型功能：路线清楚

```text
grill-with-docs
→ to-spec
→ to-tickets
→ /clear
→ 每个 ticket 使用独立 implement 会话
→ code-review
```

设计、spec 和 tickets 尽量保持在同一上下文；tickets 发布后，每个实现任务使用新会话。

## 8.4 巨型项目：路线未知

```text
wayfinder chart
→ 每个会话解决一个 decision ticket
→ 路线清晰
→ to-spec
→ to-tickets
→ implement
```

不要直接从 wayfinder 跳到 implement。

## 8.5 困难 Bug

```text
diagnosing-bugs
→ tight feedback loop
→ 最小复现
→ 3～5 个可证伪假设
→ instrumentation
→ 失败回归测试
→ 根因修复
→ 清理
→ code-review
```

## 8.6 PR Review

```text
/mattpocock-skills:code-review

固定点：origin/main
规格：issue #142
范围：committed + staged + unstaged + untracked
只报告，不修改。
```

## 8.7 需求不清

```text
有 repo → grill-with-docs
无 repo → grill-me
缺官方事实 → research
缺第三方回答 → to-questionnaire
必须体验后决定 → prototype
```

## 8.8 架构重构

目标模块已知：

```text
codebase-design → grill-with-docs → to-spec → tdd
```

目标区域未知：

```text
improve-codebase-architecture
→ 选择候选
→ grill-with-docs
→ to-spec
→ to-tickets
→ implement
```

## 8.9 合并冲突

```text
resolving-merge-conflicts
→ 查双方原始意图
→ 逐 hunk 处理
→ typecheck
→ tests
→ format
→ 用户确认
→ 完成 merge/rebase
```

## 8.10 基础设施人工操作

```text
wizard
→ bash -n
→ shellcheck
→ 人工运行
→ 将结果写回 ticket 或文档
```

必须确保：

- `.env` 已被 `.gitignore` 排除。
- secret 不打印、不进入日志。
- 不可逆步骤有人工确认。

## 8.11 研究任务

```text
research
→ 后台生成引用报告
→ 主线程继续工作
→ 报告完成
→ grill-with-docs
→ to-spec
```

## 8.12 原型验证

```text
grill-with-docs 明确一个问题
→ handoff 到原型目录
→ prototype
→ 用户实际操作
→ 记录 verdict
→ handoff 回主线程
→ 使用 tdd 做生产实现
```

---

# 9. 依赖与副作用速查

## 9.1 强依赖 Git

- `setup-matt-pocock-skills`
- `improve-codebase-architecture`
- `implement`
- `prototype`
- `code-review`
- `resolving-merge-conflicts`
- `triage`

## 9.2 强依赖 Tracker 配置

- `triage`
- `to-spec`
- `to-tickets`
- `wayfinder`
- `code-review`

`implement` 从 tracker ticket 开始时也会间接依赖 tracker，但也可使用本地 spec。

## 9.3 测试命令是核心前置

- `tdd`
- `diagnosing-bugs`
- `implement`
- `resolving-merge-conflicts`
- `triage` 的验证阶段

`prototype` 则明确不是生产级测试流程。

## 9.4 会启动子代理

明确会启动：

- `research`
- `code-review`
- `improve-codebase-architecture`

条件性启动：

- `codebase-design`
- `wayfinder`
- `grilling`

## 9.5 可能修改远端系统

- `triage`：comment、label、close issue/PR。
- `to-spec`：创建 issue、添加 label。
- `to-tickets`：创建多个 issue 和依赖关系。
- `wayfinder`：创建、claim、comment、close 和修改 map。
- `wizard` 生成的脚本：可能写 GitHub secrets/variables。

## 9.6 可能修改 Git 状态

- `implement`：默认流程可能 commit。
- `prototype`：可能创建和提交原型分支。
- `resolving-merge-conflicts`：可能 stage、commit 或继续 rebase。

## 9.7 可能写长期 Agent 指令

- `setup-matt-pocock-skills`
- `domain-modeling`
- `writing-for-agents`

---

# 10. 11 个存在但未导出的技能

以下目录有 `SKILL.md`，但没有出现在插件清单的 `skills` 数组中，因此当前不能通过 `/mattpocock-skills:<name>` 调用。

## 10.1 `in-progress/` 中的 7 个

### `claude-handoff`

压缩当前上下文并直接启动后台 Claude 会话。不同于稳定版 `handoff`：后者只写临时 Markdown。

### `implement-spec`

并行实现整个 ticket graph，可能创建 branch、draft PR、多个 worktree，并启动 implementer、merger 和 review 子代理。副作用远大于稳定版 `implement`。

### `loop-me`

通过 grilling 设计个人长期生活或工作 workflow。

### `setup-ts-deep-modules`

为 TypeScript 项目安装 `dependency-cruiser` 并建立 deep module 边界，会新增依赖和架构约束。

### `writing-beats`

从固定原材料出发，按 beat 逐段组织文章。

### `writing-fragments`

将写作碎片持续追加到 Markdown，不提前建立文章结构。

### `writing-shape`

基于只读原材料逐段塑造文章结构和表达形式。

## 10.2 `misc/` 中的 4 个

### `git-guardrails-claude-code`

安装 `PreToolUse` hook，阻止 `git push`、`reset --hard`、`clean -f`、`branch -D` 等高风险命令。

### `migrate-to-shoehorn`

将 TypeScript 测试中的部分类型断言迁移到 `@total-typescript/shoehorn`。

### `scaffold-exercises`

为 AI Hero 特定课程仓库生成练习目录，具有很强仓库专用性。

### `setup-pre-commit`

安装 Husky、lint-staged 和 Prettier，并修改相关配置。

## 10.3 如果确实要使用

不要直接修改插件 cache。上游给出的独立安装形式是：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

安装前应确认：

1. 该技能是否仍存在于当前上游。
2. 是否需要一起安装 references、scripts 或模板。
3. 是否会新增依赖或修改 settings。
4. 是否创建 worktree、commit、push 或 PR。
5. Windows 是否有 Bash 环境。
6. 是否接受 beta 技能随时变化或消失。

---

# 11. 新手学习顺序

## 11.1 第一阶段：先掌握 5 个

1. `ask-matt`：不知道选什么时负责路由。
2. `grill-with-docs`：在编码前澄清需求并保存领域知识。
3. `tdd`：建立一个测试、一个实现的小步循环。
4. `diagnosing-bugs`：困难问题先建立反馈环，不猜修。
5. `code-review`：分别检查代码规范和需求符合度。

每个工程项目再补一次：

```text
/mattpocock-skills:setup-matt-pocock-skills
```

## 11.2 第二阶段：中大型任务

学习：

- `to-spec`
- `to-tickets`
- `implement`
- `handoff`

## 11.3 第三阶段：复杂未知工作

最后再学习：

- `wayfinder`
- `prototype`
- `codebase-design`
- `improve-codebase-architecture`

## 11.4 不建议日常无边界自动运行

以下技能成本、副作用或流程重量较大，建议显式调用并给出边界：

- `wizard`
- `resolving-merge-conflicts`
- `prototype`
- `research`
- `code-review`
- `domain-modeling`
- `writing-for-agents`

不要随意移除这些技能的 manual-only 限制：

- `triage`
- `to-spec`
- `to-tickets`
- `wayfinder`
- `implement`
- `setup-matt-pocock-skills`

---

# 12. 故障排查

## 12.1 Slash 命令没有出现

在 Claude Code 中检查：

```text
/skills
/plugin
/help
```

必要时：

```text
/reload-plugins
```

CLI 检查：

```bash
claude plugin details mattpocock-skills
```

## 12.2 自然语言没有触发 Skill

先确认它是否属于 14 个 manual-only 技能。如果是，必须显式输入：

```text
/mattpocock-skills:<name>
```

这不是故障。

## 12.3 只看到 11 个技能

确认你看到的是模型可自动调用列表，还是 slash 菜单中的全部技能。本机插件实际导出 25 个，其中 11 个允许模型自动调用。

## 12.4 `to-spec`、`triage`、`wayfinder` 报缺配置

运行：

```text
/mattpocock-skills:setup-matt-pocock-skills
```

检查：

```text
docs/agents/issue-tracker.md
docs/agents/domain.md
docs/agents/triage-labels.md
```

## 12.5 GitHub/GitLab 操作失败

GitHub：

```bash
gh auth status
git remote -v
```

GitLab：

```bash
glab auth status
git remote -v
```

需要交互登录时，可以在 Claude Code 会话输入：

```text
! gh auth login
```

## 12.6 `code-review` 报空 diff

检查：

```bash
git rev-parse origin/main
git diff origin/main...HEAD
```

如果评审未提交修改，应明确要求检查：

```text
git diff
git diff --cached
untracked files
```

## 12.7 Wizard 脚本无法运行

它输出 Bash，不是 PowerShell。Windows 下使用 Git Bash、WSL 或 Claude Code 的 Bash 环境，并先执行：

```bash
bash -n <script>
```

## 12.8 子代理没有启动

检查：

- 当前 permission mode。
- Agent 工具是否可用。
- 后台任务是否被禁用。
- 子代理所需 Bash、Write、Web 或 MCP 权限是否被拒绝。

子代理不能绕过主会话权限。

## 12.9 架构报告没有图表

`improve-codebase-architecture` 生成的 HTML 可能引用 Tailwind 和 Mermaid CDN。离线或 CSP 拦截时，页面内容可能存在，但样式或图表缺失。

## 12.10 Prototype 看起来可用，能否直接上线

不能。应提取已验证的设计结论，然后使用 `tdd` 重新实现生产版本。

## 12.11 安装目录里有 Skill，为什么命令不可用

目录存在不等于被插件导出。最终以以下文件中的 `skills` 数组为准：

```text
C:\Users\z00053163\.claude\plugins\cache\claude-plugins-official\mattpocock-skills\1.2.3\.claude-plugin\plugin.json
```

不要直接修改插件 cache。

---

# 13. 最终使用原则

1. 每个 repo 只 setup 一次。
2. 始终使用完整 namespace。
3. 简单任务不要强行走完整流程。
4. 未知事实交给 `research`。
5. 未知决策交给 `grilling`。
6. 无法靠文字判断时使用 `prototype`。
7. 需求清楚后再使用 `to-spec`。
8. 只有跨多个会话时才使用 `to-tickets`。
9. 只有路线仍然未知的大型工作才使用 `wayfinder`。
10. 每个 implementation ticket 尽量使用独立会话。
11. 测试 seam 先确认，再进入 TDD。
12. 困难 bug 先建立能变红的反馈环。
13. Review 必须提供 fixed point 和 spec 来源。
14. 远端写入、commit、merge、rebase 和 secret 操作保留人工确认。
15. 不要直接修改插件 cache。
16. Windows 上遇到 `.sh` 时先确认 Bash 环境。

一句话总结：

> 不要每次把所有 Skill 跑一遍。根据当前的不确定性选择最轻的一条链：未知事实用 `research`，未知决策用 `grilling`，需要运行验证用 `prototype`，需求清楚后用 `to-spec`/`to-tickets`，编码用 `tdd`，困难故障用 `diagnosing-bugs`，最后用 `code-review` 验证代码质量和需求符合度。
