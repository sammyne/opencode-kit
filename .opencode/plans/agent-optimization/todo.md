# TODO

## 文件结构

| 文件 | 操作 | 职责 |
|------|------|------|
| `.opencode/agents/planner.md` | 修改 | 增加需求挑战、强制替代方案、自审清单强化、Confusion Protocol、Confidence Calibration |
| `.opencode/agents/builder.md` | 修改 | 增加安全护栏、计划完成度审计、测试失败分类 |
| `.opencode/agents/reviewer.md` | 新增 | 只读代码审查 agent |
| `.opencode/agents/retro.md` | 新增 | 工程复盘 agent |

## 任务列表

### ✅ 1. Planner: 强化需求挑战

- 优先级: P0
- 依赖项: 无
- 涉及文件: `.opencode/agents/planner.md`
- 验收标准: planner.md 阶段 1 中，在"探索代码库"之后、"生成需求文档"之前，有一个独立的"需求挑战"步骤，包含 3 个 forcing questions
- 风险/注意点: 需求挑战步骤不能变成阻塞流程的多轮对话，应该是 planner 自主评估后在需求文档中体现
- 步骤:
  - [ ] 在阶段 1 的第 2 步"探索代码库"和第 3 步"生成需求文档"之间，插入新的第 3 步"需求挑战"
  - [ ] 新步骤内容：基于代码库探索结果，自主评估 3 个问题（现状为什么不够好、最小切入点、已有方案），将评估结论体现在需求文档的"目标与背景"中
  - [ ] 原第 3-5 步序号顺调为 4-6

### ✅ 2. Planner: 强制替代方案

- 优先级: P0
- 依赖项: 无
- 涉及文件: `.opencode/agents/planner.md`
- 验收标准: 需求文档模板中"方案比较"章节的描述改为强制要求，包含 effort 估算和最小可行版/理想架构的要求；删除"简单需求的方案比较可以简短"的措辞
- 风险/注意点: 不改变阶段流程，只修改需求文档模板的内容描述
- 步骤:
  - [ ] 修改需求文档模板中"方案比较"章节的描述，改为强制要求至少 2 个方案
  - [ ] 每个方案增加 effort 估算（S/M/L）字段
  - [ ] 增加约束：一个方案必须是"最小可行版"，一个必须是"理想架构"
  - [ ] 删除"简单需求的方案比较可以简短（几句话即可）"这句话
  - [ ] 在阶段 2 步骤 2 中明确：用户必须选择方案后才能进入阶段 3

### ✅ 3. Builder: 安全护栏

- 优先级: P0
- 依赖项: 无
- 涉及文件: `.opencode/agents/builder.md`
- 验收标准: builder.md 约束部分新增一条关于破坏性命令警告的约束，包含具体的命令列表和警告格式，明确不阻断执行
- 风险/注意点: 必须明确排除安全的删除操作（node_modules、dist 等），避免过度警告
- 步骤:
  - [ ] 在 builder.md 的"约束"部分末尾新增第 9 条约束
  - [ ] 内容：列出需要警告的命令（rm -rf、git push --force、git reset --hard、数据库删表），排除安全目录，指定警告格式，明确不阻断执行

### ✅ 4. Planner: 自审清单强化

- 优先级: P1
- 依赖项: 无
- 涉及文件: `.opencode/agents/planner.md`
- 验收标准: 阶段 3 的自审清单从 3 项增加到 6 项，新增依赖 DAG 检查、文件冲突检查、已有资源盘点
- 风险/注意点: 无
- 步骤:
  - [ ] 在阶段 3 第 4 步"自审清单"中，在现有 3 项之后追加 3 项新检查
  - [ ] 第 4 项：依赖 DAG 检查——验证任务依赖关系无循环
  - [ ] 第 5 项：文件冲突检查——检查不同任务是否修改同一文件同一区域，标注风险
  - [ ] 第 6 项：已有资源盘点——列出可复用的现有模块/函数/组件

### ✅ 5. Planner: Confusion Protocol

- 优先级: P1
- 依赖项: 无
- 涉及文件: `.opencode/agents/planner.md`
- 验收标准: planner.md 约束部分新增一条 Confusion Protocol 规则，明确触发条件、处理步骤和与"待确认事项"的区别
- 风险/注意点: 要明确区分实时触发（Confusion Protocol）和事后收集（待确认事项），避免混淆
- 步骤:
  - [ ] 在 planner.md 的"约束"部分末尾新增第 5 条约束
  - [ ] 内容：当遇到高风险歧义（架构选择、数据模型设计、破坏性范围变更、关键上下文缺失）时，STOP → 命名困惑 → 给出 2-3 个选项及权衡 → 问用户。不用于显而易见的决策。

### ✅ 6. Builder: 计划完成度审计

- 优先级: P1
- 依赖项: 无
- 涉及文件: `.opencode/agents/builder.md`
- 验收标准: builder.md 在"分支完成工作流"之前、所有任务完成后，有一个"计划完成度审计"步骤，重新读取 todo.md 对照验收标准检查
- 风险/注意点: 审计结果应体现在最终报告中，不引入新的用户交互停止点
- 步骤:
  - [ ] 在 builder.md 的"分支完成工作流"之前，插入新的步骤"6. 计划完成度审计"
  - [ ] 内容：重新读取 todo.md，逐条对照验收标准检查实际代码变更是否满足，遗漏项在最终报告中标注
  - [ ] 原"分支完成工作流"和"最终报告"的编排顺序不变，审计结果合入最终报告

### ✅ 7. Builder: 测试失败分类

- 优先级: P1
- 依赖项: 无
- 涉及文件: `.opencode/agents/builder.md`
- 验收标准: builder.md 的"系统化调试"或"执行质量检查"部分包含测试失败分类逻辑，区分本分支引入和预先存在，预先存在的不阻断执行
- 风险/注意点: 分类是启发式的，不确定时默认归为本分支引入（保守策略）
- 步骤:
  - [ ] 在 builder.md 的"执行质量检查"部分增加测试失败分类逻辑
  - [ ] 分类方法：对比 `git diff origin/<base>...HEAD --name-only` 与失败测试涉及的文件
  - [ ] 本分支引入的失败：必须修复后才能继续
  - [ ] 预先存在的失败：在最终报告中标注，不阻断执行
  - [ ] 不确定时默认归为本分支引入

### ✅ 8. 新增 Reviewer Agent

- 优先级: P2
- 依赖项: 无
- 涉及文件: `.opencode/agents/reviewer.md`（新增）
- 验收标准: reviewer.md 包含完整的 YAML frontmatter（权限同 planner）和审查工作流（读取 diff → 逐文件审查 → 输出 FIX/ASK/INFO 三级报告），明确"只报告不修复"
- 风险/注意点: 需要 bash 权限运行 git diff 命令
- 步骤:
  - [ ] 创建 `.opencode/agents/reviewer.md`
  - [ ] YAML frontmatter：description、mode: primary、temperature: 0.3、permission 同 planner
  - [ ] 工作流：读取规划文件（requirements.md + todo.md）→ 获取 git diff → 按文件逐个审查 → 输出结构化报告
  - [ ] 审查维度：bug 模式、代码风格、测试覆盖、验收标准完成度
  - [ ] 报告格式：FIX/ASK/INFO 三级分类，输出到 `.opencode/plans/review-{日期}.md`
  - [ ] 明确约束：只报告不修复

### ✅ 9. 新增 Retro Agent

- 优先级: P2
- 依赖项: 无
- 涉及文件: `.opencode/agents/retro.md`（新增）
- 验收标准: retro.md 包含完整的 YAML frontmatter（权限同 planner）和复盘工作流（git 数据采集 → 指标计算 → 会话检测 → 团队分析 → 报告输出），支持时间窗口参数和历史对比
- 风险/注意点: 需要 bash 权限运行 git 命令
- 步骤:
  - [ ] 创建 `.opencode/agents/retro.md`
  - [ ] YAML frontmatter：description、mode: primary、temperature: 0.3、permission 同 planner
  - [ ] 工作流：预检 → 数据采集（8 条 git 命令）→ 指标计算 → 提交时间分布 → 会话检测 → 提交类型 → 热点分析 → 聚焦度 → 团队分析 → streak → 历史对比
  - [ ] 输入格式：支持 7d/24h/14d/30d/compare 参数
  - [ ] 报告输出到 `.opencode/plans/retro-{日期}.md`，末尾嵌入 JSON 快照供历史对比
  - [ ] 语气要求：锚定在具体提交上，表扬具体化，建议框架为投资而非批评

### ✅ 10. Planner: Confidence Calibration

- 优先级: P2
- 依赖项: 无
- 涉及文件: `.opencode/agents/planner.md`
- 验收标准: TODO 列表的任务模板中增加"信心评估"字段（1-5），低信心任务的步骤中标注不确定假设
- 风险/注意点: 不改变 TODO 列表的整体结构，只在每个任务的元数据中增加一个字段
- 步骤:
  - [ ] 在 TODO 列表任务模板中，在"风险/注意点"之后增加"信心评估: [1-5]"字段
  - [ ] 增加评估标准说明：5=确定有参考实现，3=大致清楚有细节需探索，1=高度不确定需原型验证
  - [ ] 在步骤粒度说明中增加：低信心（1-2）任务的步骤应标注不确定的假设

## 实现建议

- 任务 1-3（P0）优先实施，任务 4-7（P1）其次，任务 8-10（P2）最后
- 修改 planner.md 和 builder.md 时，保持 YAML frontmatter 不变，只修改 `---` 之后的内容
- 新建 reviewer.md 和 retro.md 时，YAML frontmatter 的 permission 配置参照 planner.md
- 所有 agent 文件使用中文编写，与现有 planner.md 和 builder.md 保持一致
