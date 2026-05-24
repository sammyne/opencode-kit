# 需求

## 目标与背景

将 retro 从 agent 改为 skill。retro 是一次性的"跑命令 → 算指标 → 输出报告"任务，不需要多轮交互，不需要独立的权限边界，更适合作为 skill 被任意 agent 按需加载执行。

改造后：
- 删除 `.opencode/agents/retro.md`
- 新增 `.opencode/skills/retro/SKILL.md`
- 更新 `AGENTS.md` 反映架构变化

## 方案比较（强制）

### 方案 1: 直接迁移（最小可行版）

- 思路: 将 retro.md 的内容原样搬到 SKILL.md 格式，加上 YAML frontmatter（name + description），删除 agent 特有的 permission 配置
- 优点: 改动最小，行为完全一致
- 缺点: 无
- 工作量估算: S

### 方案 2: 迁移 + 重构（理想架构）

- 思路: 迁移的同时，按 opencode skill 的最佳实践重构内容结构（比如拆分为多个 skill、增加 triggers 等）
- 优点: 更符合 skill 生态的惯例
- 缺点: retro 是单一功能，拆分没有实际收益；增加复杂度
- 工作量估算: M

### 推荐

方案 1。retro 功能单一且自包含，直接迁移即可，不需要额外重构。

## 功能需求列表

### 核心功能

- 删除 `.opencode/agents/retro.md`
- 创建 `.opencode/skills/retro/SKILL.md`，包含 YAML frontmatter（name、description）和原 retro.md `---` 之后的全部内容
- 更新 `AGENTS.md`：架构图中去掉 retro.md，工作流描述中将 retro 改为 skill 调用，关键约束中去掉 retro 条目，产出物描述从"四个 agent"改为"三个 agent + skill"

## 非功能需求

- **兼容性**：SKILL.md 的 YAML frontmatter 必须包含 `name` 和 `description` 字段
- **可维护性**：skill 内容自包含，不依赖外部文件

## 边界与不做事项

- 不修改 retro 的工作流逻辑和报告格式
- 不修改 planner、builder、reviewer agent
- 不修改 docs/gstack.md

## 假设与约束

- **技术假设**：opencode skill 文件位于 `.opencode/skills/{name}/SKILL.md`
- **技术假设**：skill 继承调用它的 agent 的权限，不需要自己配置 permission

## 待确认事项

无
