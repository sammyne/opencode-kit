# opencode-kit

OpenCode 工作流的 agent 和 skill 集合。三个 agent + 一个 skill，覆盖从需求规划到工程复盘的完整开发周期。

## 架构

```
.opencode/agents/
  planner.md   — 需求分析 + TODO 生成
  builder.md   — 按 TODO 编码实现
  reviewer.md  — 代码审查（只报告不修复）

.opencode/skills/
  retro/SKILL.md — 工程复盘（分析 git 历史）
```

## 工作流

```
用户需求 -> [planner] -> requirements.md + todo.md
                ↓ 用户确认
         -> [builder] -> 代码实现
                ↓ 所有任务完成
         -> [reviewer] -> 审查报告（可选）
                ↓
         -> [retro skill] -> 工程复盘（可选，随时可运行）
```

## 各角色说明

| 角色 | 类型 | 职责 | 可写范围 |
|------|------|------|---------|
| planner | agent | 挑战需求前提，生成需求文档和 TODO 列表 | `.opencode/plans/*.md` |
| builder | agent | 按 TODO 自主连续编码，TDD，系统化调试 | 项目文件 + `plans/**/todo.md` |
| reviewer | agent | 审查 diff，输出 FIX/ASK/INFO 三级报告 | `.opencode/plans/*.md` |
| retro | skill | 分析 git 历史，生成工程复盘报告 | 项目根目录 `retro-*.md` |

## 快速开始

1. 将 `.opencode/` 目录拷贝到你的项目根目录
2. 在 opencode TUI 界面切换到 planner agent
3. 输入需求描述，开始规划流程

### 示例

```
用 langchain 实现一个极简聊天机器人
```

planner 会依次完成：
- 需求挑战（质疑必要性、寻找最小切入点）
- 强制替代方案（至少 2 个方案，含工作量估算）
- 生成需求文档和 TODO 列表

确认后切换到 builder，自动按依赖顺序编码实现。

### 运行复盘

在任意 agent 模式下加载 retro skill：

```
跑个复盘
```

或指定时间窗口：

```
跑个 14 天的复盘
```

## 特性

### Planner
- 需求挑战：探索代码库后自主评估 3 个 forcing questions
- 强制替代方案：至少 2 个方案（最小可行版 + 理想架构），含 S/M/L 工作量估算
- Confusion Protocol：高风险歧义实时停止询问，区别于事后批量收集
- 自审清单：占位符扫描、一致性、范围、歧义、依赖 DAG、文件冲突、已有资源盘点
- Confidence Calibration：每个 TODO 任务 1-5 信心评估

### Builder
- 自主连续执行：按依赖拓扑顺序执行，仅在阻塞时停止
- TDD 流程：RED -> GREEN -> REFACTOR
- 安全护栏：破坏性命令（rm -rf、force push 等）输出警告
- 系统化调试：4 步调试 + 3 次失败规则
- 测试失败分类：区分本分支引入 vs 预先存在
- 计划完成度审计：最终报告前交叉验证验收标准

### Reviewer
- 只读审查，不修改代码
- FIX/ASK/INFO 三级问题分类
- 验收标准完成度审计

### Retro (Skill)
- 基于 git 数据的工程复盘
- 工作会话检测（45 分钟间隔分割）
- 提交时间分布、热点文件、聚焦度分析
- 团队成员分析（表扬锚定具体提交）
- 连续发布天数（streak）
- 历史对比（与上次复盘的 delta）

## 参考文献

- [OpenCode Agents 文档](https://opencode.ai/docs/agents/)
- [OpenCode Skills 文档](https://opencode.ai/docs/skills/)
- [gstack 调研报告](docs/gstack.md)
