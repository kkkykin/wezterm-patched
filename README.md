# WezTerm Custom Windows Portable Build CI

参考 [`custom-caddy`](../custom-caddy) 架构实现的自动化 GitHub Actions 编译与 Release 发布工作流，用于自动编译带有自定义补丁的 Windows 便携版 [WezTerm](https://github.com/wezterm/wezterm)。

**所有补丁均在 CI 运行期间从 PR 或链接自动抓取并应用，无需在仓库中维护任何本地 patch 文件，只需直接在 CI 工作流文件中维护链接列表。**

---

## 特性

- **全自动从 PR 获取补丁**：直接在工作流中填入 PR 链接（如 `https://github.com/wezterm/wezterm/pull/7542/changes`），CI 自动抓取并转为 `.patch` 补丁应用。
- **配置极简**：类似 `custom-caddy`，只需编辑 `.github/workflows/build-wezterm.yml` 中的 `PATCH_URLS` 环境变量即可增删补丁。
- **仅输出便携版（Portable Zip）**：纯绿色包解压即用（包含 `wezterm.exe`、`wezterm-gui.exe`、`wezterm-mux-server.exe` 及 ANGLE、ConPTY、Mesa 等全部运行库），无 Inno Setup 安装包多余开销。
- **默认构建 upstream `main` 分支**：确保获取上游最新特性与修复。
- **自动每月运行**：通过 Cron（每月 1 号 `0 0 1 * *`）自动定时触发编译与发布。
- **防冲突安全机制**：内置 `git apply --reverse --check` 智能跳过已被上游合并的补丁，支持 3-way merge 自动处理代码微小偏移。
- **自动 Release 发布**：编译成功后自动创建 GitHub Release，附带构建信息、SHA256 校验码及补丁列表。

---

## 仓库结构

整个仓库结构非常轻量，无需维护 `patches/` 目录：

```text
.
├── .github/
│   └── workflows/
│       └── build-wezterm.yml   # 编译与 Release 核心工作流
├── .gitignore
└── README.md
```

---

## 如何增删补丁

直接修改 [.github/workflows/build-wezterm.yml](.github/workflows/build-wezterm.yml) 开头的 `env.PATCH_URLS`：

```yaml
env:
  PATCH_URLS: |
    https://github.com/wezterm/wezterm/pull/7542/changes
    # 在这里继续添加其他 PR 或补丁链接，每行一个：
    # https://github.com/wezterm/wezterm/pull/1234
    # https://github.com/wezterm/wezterm/commit/abcdef123456
    # https://example.com/custom.patch
```

提交并推送到 GitHub，CI 即会自动拉取并应用所有补丁。

---

## 触发方式

1. **自动每月触发**：每月 1 号 00:00 (UTC) 自动抓取 upstream `main` 分支并编译发布。
2. **代码提交触发**：修改并推送工作流文件时自动触发。
3. **网页手动触发 (workflow_dispatch)**：在 GitHub 仓库 **Actions** -> **Build WezTerm Windows (Patched)** -> **Run workflow**：
   - `wezterm_version`：默认 `main`，也可指定分支、Tag 或 Commit SHA。
   - `patches`：留空则使用 `PATCH_URLS` 中配置的列表；也可临时输入其他链接进行单次测试。
