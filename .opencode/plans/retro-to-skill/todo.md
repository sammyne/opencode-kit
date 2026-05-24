# TODO

## 文件结构

| 文件 | 操作 | 职责 |
|------|------|------|
| `.opencode/agents/retro.md` | 删除 | 原 retro agent，迁移为 skill 后删除 |
| `.opencode/skills/retro/SKILL.md` | 新增 | retro skill，工程复盘 |
| `AGENTS.md` | 修改 | 更新架构描述，反映 retro 从 agent 变为 skill |

## 任务列表

### ✅ 1. 创建 retro skill

- 优先级: P0
- 依赖项: 无
- 涉及文件: `.opencode/skills/retro/SKILL.md`
- 验收标准: SKILL.md 存在，包含 YAML frontmatter（name: retro, description）和原 retro.md `---` 之后的全部工作流内容
- 风险/注意点: YAML frontmatter 格式与 agent 不同，skill 只需要 name 和 description，不需要 mode/temperature/permission
- 信心评估: 5
- 步骤:
  - [ ] 创建 `.opencode/skills/retro/SKILL.md`
  - [ ] YAML frontmatter 写入 name 和 description
  - [ ] 将原 retro.md `---` 之后的全部内容（从"你是工程复盘专家"开始）复制为 skill 正文

### ✅ 2. 删除 retro agent

- 优先级: P0
- 依赖项: 1
- 涉及文件: `.opencode/agents/retro.md`
- 验收标准: `.opencode/agents/retro.md` 文件不存在
- 风险/注意点: 确保任务 1 完成后再删除，避免内容丢失
- 信心评估: 5
- 步骤:
  - [ ] 删除 `.opencode/agents/retro.md`

### ✅ 3. 更新 AGENTS.md

- 优先级: P0
- 依赖项: 1, 2
- 涉及文件: `AGENTS.md`
- 验收标准: AGENTS.md 中产出物描述改为"三个 agent 配置和一个 skill"；架构图中 retro.md 从 agents 移到 skills；工作流描述中 retro 改为 skill 调用；关键约束中去掉 retro 条目；agent 数量从"四个"改为"三个"
- 风险/注意点: 无
- 信心评估: 5
- 步骤:
  - [ ] 仓库说明：产出物从"四个 agent 配置"改为"三个 agent 配置和一个 skill"
  - [ ] 架构图：去掉 agents 下的 retro.md，增加 skills/retro/SKILL.md
  - [ ] 工作流：将"切换到 retro agent"改为"加载 retro skill"
  - [ ] 关键约束：去掉 retro 条目，"四个 agent"改为"三个 agent"

## 实现建议

- 任务 1 先创建 skill 文件，确认内容完整后再执行任务 2 删除 agent 文件
- SKILL.md 的正文内容与原 retro.md 完全一致，只有 frontmatter 不同
