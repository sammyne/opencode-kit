# gstack 调研报告

> 调研对象: [garrytan/gstack](https://github.com/garrytan/gstack)
> 调研目的: 找出可用于优化本项目 planner/builder agent 的特性，评估新增 reviewer/retro agent 的可行性

## 一、gstack 项目概况

gstack 是 Y Combinator CEO Garry Tan 的开源 AI 工程工作流工具集，核心理念是将 AI coding agent 变成一个虚拟工程团队。它定义了 23+ 个专业化 skill（Claude Code 的 SKILL.md），涵盖完整的软件开发生命周期：

**Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect**

每个 skill 扮演一个专家角色（CEO、工程经理、设计师、QA 主管、发布工程师、调试专家等），前一个 skill 的输出自动传递给下一个。

### 核心 skill 列表

| 阶段 | Skill | 角色 | 职责 |
|------|-------|------|------|
| Think | `/office-hours` | YC Office Hours | 6 个 forcing questions 重塑产品思维 |
| Plan | `/plan-ceo-review` | CEO/Founder | 挑战范围，找到 10-star 产品 |
| Plan | `/plan-eng-review` | Eng Manager | 锁定架构、数据流、边界、测试 |
| Plan | `/plan-design-review` | Senior Designer | 10 维度评分 + AI Slop 检测 |
| Plan | `/autoplan` | Review Pipeline | 自动串联 CEO -> Design -> Eng 评审 |
| Build | `/review` | Staff Engineer | Pre-landing PR 审查，自动修复 |
| Build | `/investigate` | Debugger | 系统化根因调试，Iron Law |
| Test | `/qa` | QA Lead | 真实浏览器测试 + 自动修复 |
| Ship | `/ship` | Release Engineer | 测试 + 覆盖率审计 + PR |
| Ship | `/land-and-deploy` | Release Engineer | 合并 + 部署 + 生产验证 |
| Reflect | `/retro` | Eng Manager | 周复盘，per-person 分析 |
| Memory | `/learn` | Memory | 跨会话学习项目模式 |
| Safety | `/careful` | Safety | 破坏性命令告警 |
| Safety | `/freeze` | Safety | 限制编辑范围到指定目录 |

### 核心设计哲学（ETHOS.md）

1. **Boil the Lake**: AI 让完整实现的边际成本趋近于零，做完整的事而非走捷径
2. **Search Before Building**: 三层知识框架——Layer 1 经验（tried-and-true）、Layer 2 趋势（new-and-popular）、Layer 3 第一性原理（first-principles）
3. **User Sovereignty**: AI 推荐，用户决定。两个模型达成一致是强信号，不是决定

### 跨 skill 共享机制

gstack 通过模板系统（`.tmpl` + resolvers）将通用行为注入所有 skill。以下是注入到每个 skill 的共享模块（按 tier 分层）：

| Tier | 包含模块 | 适用 skill 举例 |
|------|---------|----------------|
| T1 | 核心 bash setup、升级检查、onboarding、voice、completion status | browse, benchmark |
| T2 | T1 + AskUser 格式、context recovery、writing style、completeness principle、confusion protocol、checkpoint、context health、question tuning | investigate, cso, retro |
| T3 | T2 + repo-mode、search-before-building | autoplan, office-hours, plan-*-review |
| T4 | T3 + test failure triage | ship, review, qa |

关键共享模块说明：

- **Confusion Protocol**: 遇到高风险歧义（架构、数据模型、破坏性范围、缺失上下文）时，必须 STOP，用一句话命名困惑，给出 2-3 个选项及权衡，然后问用户。不用于常规编码或显而易见的变更。
- **Context Recovery**: 会话开始时自动恢复最近的项目上下文（最近的 artifact、上次会话信息、最近的 skill 模式），帮助会话连续性。
- **Continuous Checkpoint**: 可选的自动提交模式，完成逻辑单元后用 `WIP:` 前缀提交，附带结构化上下文元数据（决策、剩余工作、失败尝试）。`/ship` 后续会将 WIP 提交压缩为干净提交。
- **Context Health**: 长时间运行的 skill 会话中，定期输出 `[PROGRESS]` 摘要（已完成、下一步、意外发现）。如果在同一个诊断/文件/修复变体上循环，必须 STOP 并重新评估。
- **Completeness Principle**: 每个选项附带 `Completeness: X/10` 评分（10=所有边界情况，7=happy path，3=捷径），让用户看到完整度权衡。
- **Test Failure Ownership Triage**: 测试失败时，先分类为"本分支引入"还是"预先存在"，再根据仓库模式（solo/collaborative）决定处理方式。
- **Confidence Calibration**: 评审发现的 1-10 评分，7+ 正常显示，5-6 附带警告，<5 抑制。没有引用代码片段的发现强制降为 4-5 分。
- **Review Army**: `/review` 和 `/ship` 中的并行专家审查——根据 diff 范围自动选择专家（Testing、Maintainability 始终启用；Security、Performance、Data Migration、API Contract、Design 按条件启用），并行 dispatch subagent，合并去重发现。

### 其他值得注意的机制

- **Adversarial Spec Review Loop**: 生成设计文档后，用对抗性 subagent 审查（最多 3 轮迭代），确保规格文档经得起挑战。
- **Scope Drift Detection**: 在 review 中检测 diff 是否偏离了原始计划/TODOS。
- **TODOS.md 作为活跃 backlog**: gstack 维护一个统一的 TODOS.md，各 skill 交叉引用并自动更新。
- **Plan Persistence**: CEO 计划持久化到 `~/.gstack/projects/{slug}/ceo-plans/`，跨会话可引用。
- **Review Readiness Dashboard**: ship 前展示哪些 review 已完成及其状态的仪表板。
- **Worktree Parallelization Strategy**: 工程评审输出如何跨 git worktree 并行化实现的策略。
- **Slop Scan**: AI 代码质量检查器，捕捉 AI 生成代码比人类代码更差的模式（空 catch、冗余 return await 等）。

## 二、与本项目 planner/builder 的对比

### Planner vs gstack 规划类 skill

| 维度 | 本项目 planner | gstack |
|------|---------------|--------|
| 需求挑战 | 阶段 0 有范围评估，阶段 1 有待确认事项 | `/office-hours` 用 6 个 forcing questions 挑战需求前提 |
| 替代方案 | 有"方案比较"章节，但措辞为可简短 | 强制要求 2-3 个替代方案（mandatory），含 effort 估算 |
| 评审层次 | 单一 agent 完成全部评审 | CEO/Design/Eng/DX 四个独立评审阶段 |
| 对抗审查 | 无 | Outside Voice（Codex 或 Claude subagent 独立挑战计划） |
| 信心校准 | 无 | 每个维度 1-10 评分，< 7 分必须说明不确定原因 |
| 自审深度 | 4 项（占位符、一致性、范围、歧义） | 9 Prime Directives + 15 Cognitive Patterns |
| NOT IN SCOPE | "边界与不做事项"章节 | 独立的"NOT in scope"+"What already exists"两个章节 |
| 决策分类 | 无 | Mechanical/Taste/User Challenge 三级分类，避免过度打扰 |
| 歧义处理 | 自审清单中的歧义检查（事后收集） | Confusion Protocol: 高风险歧义实时触发，STOP + 命名困惑 + 给出选项 |

### Builder vs gstack 编码/发布类 skill

| 维度 | 本项目 builder | gstack |
|------|---------------|--------|
| 调试流程 | 4 步调试 + 3 次失败规则 | `/investigate` Iron Law: 4 阶段 + scope lock + 模式库 |
| 完成状态 | 3 种（✅/⚠️/❌）+ 升级协议 | 4 种（DONE/DONE_WITH_CONCERNS/BLOCKED/NEEDS_CONTEXT） |
| 安全护栏 | 无 | `/careful` 自动拦截破坏性命令 |
| 编辑范围锁 | 无 | `/freeze` 调试时限制编辑到指定目录 |
| 计划完成审计 | 标记 todo 状态后输出报告 | `/ship` Step 8 用 subagent 交叉审计每个 plan item |
| 质量检查 | lint/typecheck | `/review` 检查 SQL 安全、信任边界、条件副作用 |
| 发布流程 | 简单的分支完成选项 | `/ship` 20 步全自动流程（测试 -> 覆盖率 -> 审查 -> 版本号 -> CHANGELOG -> PR） |
| 跨会话记忆 | 无 | `/learn` 持久化项目模式，`/context-save` 保存工作上下文 |
| 测试失败分类 | 无区分 | Test Failure Triage: 区分"本分支引入"vs"预先存在"，分别处理 |
| 进度可见性 | 每任务输出完成状态 | Context Health: 长会话定期输出 [PROGRESS] 摘要 + 循环检测 |
| Scope Drift | 无 | Scope Drift Detection: 检测 diff 是否偏离原始计划 |

## 三、整体工作流设计

### 目标架构：四个 agent 的串联

```
用户需求 -> [planner] -> requirements.md + todo.md
                ↓ 用户确认
         -> [builder] -> 代码实现
                ↓ 所有任务完成
         -> [reviewer] -> 审查报告（只报告，不修复）
                ↓ 用户决定是否合并
         -> [retro] -> 工程复盘报告（可选，独立运行）
```

### 各 agent 职责边界

| Agent | 职责 | 可写范围 | 设计原则 |
|-------|------|---------|---------|
| planner | 需求分析 + TODO 生成 | `.opencode/plans/*.md` | 只读项目代码，挑战需求前提 |
| builder | 按 TODO 编码实现 | 项目文件 + `plans/**/todo.md` | 自主连续执行，仅在阻塞时停止 |
| reviewer | 审查 builder 的 diff | `.opencode/plans/*.md`（审查报告） | 只读项目代码，只报告不修复 |
| retro | 分析 git 历史生成复盘 | `.opencode/plans/*.md`（复盘报告） | 只读项目代码，数据驱动 |

### 关键设计约束

- **builder 不停顿**：builder 的核心设计是自主连续执行，不在任务之间等待用户确认。所有建议不得引入新的用户交互停止点。现有的 ❌（被阻塞）和升级协议已覆盖需要人为介入的场景。
- **reviewer 不修复**：reviewer 只输出审查报告，修复动作留给用户或 builder。避免职责混淆。
- **retro 独立运行**：retro 不依赖前面三个 agent 的输出，随时可以独立运行。

## 四、Planner 优化建议

### P0: 强化需求挑战

planner 阶段 0 已有"范围评估"，阶段 1 已有"待确认事项"，但挑战力度不够。借鉴 gstack `/office-hours` 的 forcing questions，在阶段 1 探索代码库之后、生成需求文档之前，增加一个需求挑战步骤：

- 现状为什么不够好？不做这个需求会怎样？
- 最小可行切入点是什么？能否进一步缩小范围？
- 有没有已有代码/工具/库可以直接解决这个问题？

如果挑战结果导致需求方向调整，应在需求文档的"目标与背景"中体现调整过程和理由。

### P0: 强制替代方案

planner 已有"方案比较"章节，但措辞为"简单需求的方案比较可以简短（几句话即可）"。借鉴 gstack `/plan-ceo-review` 的 0C-bis，将替代方案改为强制要求：

- 至少提出 2 个实现方案
- 每个方案包含思路、优缺点和粗略 effort 估算（S/M/L）
- 一个必须是"最小可行版"（最快发布），一个必须是"理想架构"（长期最优）
- 用户选择方案后再生成功能需求和 TODO
- 不允许跳过方案比较直接生成 TODO

### P1: 自审清单强化

在 planner 生成 TODO 列表的自审清单中增加：

- **依赖 DAG 检查**: 验证任务之间的依赖关系是否形成有向无环图，无循环依赖
- **文件冲突检查**: 检查不同任务是否会修改同一个文件的同一区域，如有则标注风险
- **已有资源盘点**: 独立列出项目中已有的、可复用的模块/函数/组件

### P1: Confusion Protocol

借鉴 gstack 的 Confusion Protocol，在 planner 中增加实时歧义处理规则（与现有"待确认事项"的区别：待确认事项是文档生成后批量收集的，Confusion Protocol 是探索代码库和生成文档过程中实时触发的）：

当遇到高风险歧义（架构选择、数据模型设计、破坏性范围变更、关键上下文缺失）时：
1. STOP，不要猜测
2. 用一句话命名困惑点
3. 给出 2-3 个选项及各自的权衡
4. 问用户

不用于显而易见的决策。

### P2: Confidence Calibration

在 TODO 列表中，对每个任务的复杂度和风险做 1-5 的信心评估：

- 5: 确定，有参考实现
- 3: 大致清楚，有些细节需要探索
- 1: 高度不确定，需要原型验证

低信心任务应在步骤中标注不确定的假设，提示 builder 执行时优先验证。

## 五、Builder 优化建议

### P0: 安全护栏

builder 当前完全没有破坏性命令防护。借鉴 gstack `/careful`，在约束中增加：

执行以下命令前必须输出警告：
- `rm -rf`（排除 node_modules、dist、.next 等构建产物目录）
- `git push --force`
- `git reset --hard`
- 数据库删表/清数据操作

格式：`WARNING: 即将执行破坏性命令: <命令>。继续执行。`

注意：这是 prompt 级别的指令，不阻断 builder 的连续执行流程。builder 输出警告后继续执行，不等待用户确认。

### P1: 计划完成度审计

在 builder 输出最终报告前，增加一个交叉验证步骤。借鉴 gstack `/ship` Step 8：

1. 重新读取 todo.md
2. 对照每个任务的验收标准，检查实际代码变更是否满足
3. 如有遗漏，在最终报告中明确标注

### P1: 测试失败分类

借鉴 gstack 的 Test Failure Ownership Triage，builder 在测试失败时先分类再处理：

1. 获取本分支变更的文件列表（`git diff origin/<base>...HEAD --name-only`）
2. 对每个失败的测试分类：
   - **本分支引入**: 失败的测试文件或其测试的代码在本分支被修改过，或能追溯到本分支的变更
   - **预先存在**: 测试文件和被测代码均未在本分支修改，且失败与本分支变更无关
   - 不确定时默认归为"本分支引入"
3. 本分支引入的失败：必须修复后才能继续
4. 预先存在的失败：在最终报告中标注，由用户事后决定处理方式

注意：预先存在的失败不阻断 builder 的连续执行。

## 六、新增 Agent: Reviewer

### 定位

只读分析 agent，不修改项目代码。对 builder 产出的 diff 做 pre-landing review，形成"规划 -> 编码 -> 审查"三阶段闭环。

### 核心能力

借鉴 gstack `/review` skill：

1. 读取 git diff（当前分支 vs 主分支），按文件逐个审查
2. 检查常见 bug 模式（竞争条件、nil 引用、边界错误、资源泄漏）
3. 检查代码风格是否与项目一致
4. 检查测试覆盖是否充分（新增代码是否有对应测试）
5. 对照 plan 的验收标准做完成度审计
6. 输出结构化审查报告，问题分三级：
   - **FIX**: 明确的问题，建议修复（未使用的 import、调试代码残留、格式问题）
   - **ASK**: 需要用户判断的问题（设计选择、潜在的竞争条件）
   - **INFO**: 信息性提示（性能建议、可选优化）

### 权限配置

```yaml
permission:
  question: "allow"
  plan_exit: "allow"
  edit:
    "*": "deny"
    ".opencode/plans/*.md": "allow"
```

与 planner 相同的权限：只读项目文件，可写入 plans 目录（用于审查报告）。审查报告输出到 `.opencode/plans/review-{日期}.md`。

### 关键约束

- **只报告，不修复**：reviewer 不修改任何项目代码，所有问题以报告形式输出
- 修复动作留给用户自行决定，或切换到 builder 处理

## 七、新增 Agent: Retro

### 定位

只读分析 agent，分析 git 历史和工作模式，生成数据驱动的工程复盘报告。独立于 planner/builder/reviewer，随时可运行。

### 核心能力

借鉴 gstack `/retro` skill：

1. 通过 git log 采集提交数据（作者、时间、文件变更、LOC）
2. 计算核心指标（提交数、LOC、测试比例、活跃天数）
3. 检测工作会话模式（45 分钟间隔分割，分类为 Deep/Medium/Micro）
4. 分析提交时间分布（高峰时段、深夜编码）
5. 热点文件分析（churn hotspot）
6. 团队成员分析（per-person 表扬和成长建议，锚定在具体提交上）
7. 连续发布天数（streak）
8. 历史对比（与上次 retro 的 delta）

### 权限配置

```yaml
permission:
  question: "allow"
  plan_exit: "allow"
  edit:
    "*": "deny"
    ".opencode/plans/*.md": "allow"
```

复盘报告输出到 `.opencode/plans/retro-{日期}.md`。

### 输入格式

```
/retro              — 默认最近 7 天
/retro 24h          — 最近 24 小时
/retro 14d          — 最近 14 天
/retro compare      — 当前窗口 vs 上一个同长度窗口
```

### 与 gstack `/retro` 的简化

| gstack 有 | 本 agent | 原因 |
|-----------|---------|------|
| 全局模式（跨仓库） | 不支持 | 需要额外的发现脚本 |
| AI 工具会话检测 | 不支持 | 依赖 gstack 特有的遥测基础设施 |
| 可分享个人卡片 | 不支持 | 视觉格式依赖终端渲染 |
| Greptile/eureka/skill 使用统计 | 不支持 | gstack 特有的生态 |

## 八、低优先级参考（不建议当前引入）

| gstack 特性 | 不引入的原因 |
|------------|------------|
| 跨模型二次意见（Codex + Claude） | 依赖 OpenAI Codex CLI，增加复杂度和成本 |
| 持久化学习（/learn） | 需要文件存储和跨会话状态管理，超出 agent 配置范畴 |
| Context Save/Restore | 需要额外基础设施，opencode 本身的会话管理已覆盖部分场景 |
| 浏览器 QA（/browse, /qa） | 特定于 Web 项目，不通用 |
| Design Review / Design Shotgun | 特定于 UI/设计场景 |
| 遥测和分析 | 增加复杂度，个人使用场景收益有限 |
| 20 步全自动 ship 流程 | 过于复杂，当前 builder 的分支完成流程够用 |
| Continuous Checkpoint（WIP 自动提交） | 需要与 git 工作流深度集成，当前 builder 的手动提交流程够用 |
| Review Army（并行专家审查） | 需要 subagent 调度能力，opencode 的 task agent 可部分替代 |
| Adversarial Spec Review Loop | 增加规划时间，当前的自审清单已覆盖主要场景 |
| Slop Scan（AI 代码质量检查） | 可通过 lint 规则部分替代，引入独立工具增加复杂度 |

## 九、实施优先级

| 优先级 | 改动 | 影响范围 | 说明 |
|--------|------|---------|------|
| P0 | planner: 强化需求挑战 | planner.md | 阶段 1 探索代码库后增加 forcing questions |
| P0 | planner: 强制替代方案 | planner.md | 方案比较从可选改为强制，含 effort 估算 |
| P0 | builder: 安全护栏 | builder.md | 破坏性命令输出警告，不阻断执行 |
| P1 | planner: 自审清单强化 | planner.md | 增加依赖 DAG 检查、文件冲突检查、已有资源盘点 |
| P1 | planner: Confusion Protocol | planner.md | 实时歧义处理，区别于批量的待确认事项 |
| P1 | builder: 计划完成度审计 | builder.md | 最终报告前交叉验证验收标准 |
| P1 | builder: 测试失败分类 | builder.md | 区分本分支引入 vs 预先存在，预先存在不阻断执行 |
| P2 | 新增 Reviewer agent | reviewer.md | 只读审查，FIX/ASK/INFO 三级分类，只报告不修复 |
| P2 | 新增 Retro agent | retro.md | git 数据分析 + 工程复盘，独立运行 |
| P2 | planner: Confidence Calibration | planner.md | TODO 任务 1-5 信心评估 |
