# WezTerm Custom Windows Portable Build CI

参考 [`custom-caddy`](../custom-caddy) 架构实现的自动化 GitHub Actions 编译与 Release 发布工作流，用于自动编译带有自定义补丁的 Windows 便携版 [WezTerm](https://github.com/wezterm/wezterm)。

---

## 特性

- **从 PR 链接自动拉取补丁（推荐）**：只需在工作流中配置 PR 页面链接（如 `https://github.com/wezterm/wezterm/pull/7542/changes`），CI 构建时自动转换为 `.patch` 下载并应用，无需手动下载维护文件。
- **支持可选的本地 Patch 文件**：如有本地私有补丁或未推送到 GitHub 的修改，可直接将 `*.patch` / `*.diff` 放入 `patches/` 目录，CI 会自动一并应用。
- **仅输出便携版（Portable Zip）**：纯绿色解压即用，包含完整的二进制与运行库（`wezterm.exe`、`wezterm-gui.exe`、`wezterm-mux-server.exe` 及 ANGLE、ConPTY、Mesa 等），无 Inno Setup 安装包多余开销。
- **纯净小巧（默认剔除 PDB）**：默认不打包体积庞大的 `.pdb` 调试符号文件，安装包极其干净精简（体积缩减 60%+）。
- **默认构建 upstream `main` 分支**：确保获取上游最新特性与修复。
- **自动每月运行**：通过 Cron（每月 1 号 `0 0 1 * *`）自动定时触发编译与发布。
- **防冲突安全机制**：内置 `git apply --reverse --check` 智能跳过已被上游合并或已打上的补丁，支持 3-way merge 自动处理代码偏移。
- **自动 Release 发布**：编译成功后自动创建 GitHub Release，附带构建信息、SHA256 校验码及补丁列表。

---

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── build-wezterm.yml   # 编译与 Release 核心工作流
├── patches/
│   └── README.md               # 可选本地补丁存放目录
├── .gitignore
└── README.md
```

---

## 如何添加补丁？

你可以自由使用以下两种方式（可单独使用，也可混用）：

### 方式 1：修改 CI 里的链接列表（推荐）
直接编辑 [.github/workflows/build-wezterm.yml](.github/workflows/build-wezterm.yml) 开头的 `env.PATCH_URLS`：

```yaml
env:
  PATCH_URLS: |
    https://github.com/wezterm/wezterm/pull/7542/changes
    # 在此继续添加其他 PR 或 Commit 链接（每行一个）：
    # https://github.com/wezterm/wezterm/pull/1234
    # https://github.com/wezterm/wezterm/commit/abcdef123456
    # https://example.com/custom.patch
```

### 方式 2：放入 `patches/` 目录（可选本地补丁）
将你自己的 `*.patch` 或 `*.diff` 文件直接放入 `patches/` 目录（例如 `patches/my-fix.patch`）并提交到仓库，CI 构建时会自动检测并应用。

CI 先按配置顺序应用远程补丁，再应用本地补丁。本仓库的 [Kitty Shift 修复](patches/README.md) 依赖 PR #7542，用于保留 mux 会话中的 `Alt+Shift`、`Ctrl+Shift` 等组合键。

---

## 触发方式

1. **自动每月触发**：每月 1 号 00:00 (UTC) 自动拉取 upstream `main` 最新代码并编译发布。
2. **代码提交触发**：修改工作流文件或 `patches/` 目录推送到 `main` 分支时自动触发。
3. **网页手动触发 (workflow_dispatch)**：在 GitHub 仓库 **Actions** -> **Build WezTerm Windows (Patched)** -> **Run workflow**：
   - `wezterm_version`：默认 `main`，也可输入具体分支、Tag 或 Commit SHA。
   - `patches`：留空则使用 `PATCH_URLS` 中配置的列表；也可临时输入其他链接进行单次测试。
   - `include_pdb`：是否打包 `.pdb` 调试符号文件（默认 `false`，默认保持纯净精简）。
