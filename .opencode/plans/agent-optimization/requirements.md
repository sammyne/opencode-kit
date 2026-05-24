# 需求

## 目标与背景

基于 gstack 项目调研（docs/gstack.md），优化本项目的 planner 和 builder agent，并新增 reviewer 和 retro 两个 agent，形成"规划 → 编码 → 审查 → 复盘"的四阶段工作流。

核心原则：
- 所有改动不得破坏 builder 的自主连续执行设计（不引入新的用户交互停止点）
- reviewer 和 retro 均为只读 agent，不修改项目代码
- 改动应为增量式，在现有 agent 基础上增强，而非重写

## 功能需求列表

### P0: Planner 强化需求挑战

在阶段 1 探索代码库之后、生成需求文档之前，增加需求挑战步骤，向用户提出 2-3 个关键反问：
- 现状为什么不够好？不做这个需求会怎样？
- 最小可行切入点是什么？能否进一步缩小范围？
- 有没有已有代码/工具/库可以直接解决这个问题？

如果挑战导致需求方向调整，在需求文档"目标与背景"中体现。

### P0: Planner 强制替代方案

将"方案比较"从可选改为强制：
- 至少 2 个方案，含思路、优缺点和 effort 估算（S/M/L）
- 一个"最小可行版"，一个"理想架构"
- 用户选择方案后再生成功能需求和 TODO
- 不允许跳过

### P0: Builder 安全护栏

在 builder 约束中增加破坏性命令警告（不阻断执行）：
- `rm -rf`（排除 node_modules、dist、.next 等）
- `git push --force`
- `git reset --hard`
- 数据库删表/清数据操作

格式：`WARNING: 即将执行破坏性命令: <命令>。继续执行。`

### P1: Planner 自审清单强化

在 TODO 列表自审清单中增加：
- 依赖 DAG 检查：验证无循环依赖
- 文件冲突检查：不同任务是否修改同一文件同一区域
- 已有资源盘点：列出可复用的模块/函数/组件

### P1: Planner Confusion Protocol

增加实时歧义处理规则（区别于批量的"待确认事项"）：
遇到高风险歧义时 STOP → 命名困惑 → 给出 2-3 个选项及权衡 → 问用户。
不用于显而易见的决策。

### P1: Builder 计划完成度审计

在最终报告前交叉验证：重新读取 todo.md，对照验收标准检查实际代码变更，遗漏项在报告中标注。

### P1: Builder 测试失败分类

测试失败时先分类再处理：
- 本分支引入：必须修复
- 预先存在：在最终报告中标注，不阻断执行
- 不确定时默认归为本分支引入

### P2: 新增 Reviewer Agent

只读审查 agent，审查 builder 的 diff，输出结构化报告：
- FIX: 明确问题，建议修复
- ASK: 需用户判断
- INFO: 信息性提示
权限同 planner，报告输出到 `.opencode/plans/review-{日期}.md`。

### P2: 新增 Retro Agent

只读分析 agent，分析 git 历史生成工程复盘报告：
- git 指标采集、会话检测、时间分布、热点分析、团队分析、streak、历史对比
权限同 planner，报告输出到 `.opencode/plans/retro-{日期}.md`。

### P2: Planner Confidence Calibration

TODO 任务增加 1-5 信心评估，低信心任务标注不确定假设。

## 非功能需求

- **兼容性**：改动后的 agent 文件必须保持 YAML frontmatter 结构完整，格式损坏会导致 agent 失效
- **可维护性**：每个 agent 文件自包含，不依赖外部脚本或模板系统
- **测试要求**：无自动化测试（本项目只有 agent 配置文件），通过人工验证 agent 行为

## 边界与不做事项

- 不引入跨模型二次意见（Codex）
- 不引入持久化学习、Context Save/Restore
- 不引入浏览器 QA、Design Review
- 不引入遥测和分析
- 不引入 20 步全自动 ship 流程
- 不修改 opencode.json 或其他配置文件
- 不修改 AGENTS.md

## 假设与约束

- **技术假设**：opencode agent 的 YAML frontmatter 中 `edit` 权限统一控制 write/edit/apply_patch 三个工具
- **环境约束**：agent 文件位于 `.opencode/agents/` 目录

## 待确认事项

无
