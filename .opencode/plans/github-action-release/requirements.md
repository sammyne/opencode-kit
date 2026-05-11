# 需求

## 目标与背景

本仓库的产出物是 `.opencode/agents/` 下的两个 agent 配置文件（`planner.md`、`builder.md`），用户需要将它们拷贝到自己的项目中使用。为简化分发流程，需要创建一条 GitHub Actions 流水线，在打 tag 时自动将 agent 文件打包为压缩包并发布到 GitHub Release 页面，方便用户一键下载。

## 功能需求列表

### 核心功能

- 流水线在推送 tag 时自动触发（tag 格式不做限制，任意 tag 均触发）
- 将 `.opencode/agents/` 目录下的所有文件打包为 `.tgz` 压缩包
- 压缩包命名格式：`opencode-plan-then-build-{tag}.tgz`，其中 `{tag}` 为触发的 tag 名称（例如 tag 为 `v0.1.0` 则文件名为 `opencode-plan-then-build-v0.1.0.tgz`）
- 压缩包内保留完整路径 `.opencode/agents/`，即解压后得到 `.opencode/agents/planner.md`、`.opencode/agents/builder.md`
- 自动创建对应 tag 的 GitHub Release，并将压缩包作为 Release Asset 上传

## 非功能需求

- **可维护性**：使用 GitHub 官方或社区广泛使用的 Actions，避免依赖冷门第三方 Action
- **安全**：仅使用 `GITHUB_TOKEN` 默认权限，无需额外 Secret 配置
- **幂等性**：如果 Release 已存在，流水线应能正常运行（更新或跳过）

## 边界与不做事项

- 不对 tag 格式做校验或限制（如不强制 semver 格式）
- 不生成 changelog 或 release notes
- 不做多平台构建（仅打包文本文件，无需矩阵构建）
- 不涉及 npm/pypi 等包管理器发布

## 假设与约束

- **技术假设**：使用 GitHub Actions 作为 CI/CD 平台，运行在 `ubuntu-latest` runner 上
- **文件假设**：`.opencode/agents/` 目录始终存在且包含需要分发的 agent 文件
- **权限假设**：仓库默认的 `GITHUB_TOKEN` 具有创建 Release 和上传 Asset 的权限

## 待确认事项

无
