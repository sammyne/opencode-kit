# TODO

## 任务列表

### ✅ 1. 创建 GitHub Actions 工作流文件

- 优先级: P0
- 依赖项: 无
- 涉及文件: `.github/workflows/release.yml`（新增）
- 验收标准:
  - 工作流在任意 tag 推送时触发（`on: push: tags`）
  - 从 `.opencode/agents/` 打包生成 `opencode-plan-then-build-{tag}.tgz`，解压后保留完整路径 `.opencode/agents/xxx`
  - 自动创建 GitHub Release 并上传压缩包作为 Asset
  - 仅使用 `GITHUB_TOKEN`，无需额外 Secret
  - 使用 `softprops/action-gh-release`（社区广泛使用的 Release Action）上传 Asset
- 风险/注意点:
  - 需确保仓库 Settings > Actions > General 中 `GITHUB_TOKEN` 的 Workflow permissions 设置为 "Read and write permissions"，否则创建 Release 会报 403
  - `tar` 打包时直接在仓库根目录执行 `tar -czf ... .opencode/agents`，保留完整路径前缀

## 实现建议

- 整个需求只需新增一个文件 `.github/workflows/release.yml`，无需修改任何现有文件
- 直接在仓库根目录执行 `tar -czf opencode-plan-then-build-{tag}.tgz .opencode/agents` 即可保留完整路径
- 使用 `softprops/action-gh-release@v2` 创建 Release 并上传文件，该 Action 内置幂等处理（Release 已存在时自动追加 Asset）
- tag 名称通过 `github.ref_name` 获取
