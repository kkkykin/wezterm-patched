# WezTerm Custom Windows Portable Build CI

参考 [`custom-caddy`](../custom-caddy) 架构实现的自动化 GitHub Actions 编译与 Release 发布工作流，用于自动编译带有自定义补丁的 Windows 便携版 [WezTerm](https://github.com/wezterm/wezterm)。

默认集成补丁：[PR #7542: fix kitty keyboard not work in mux mode](https://github.com/wezterm/wezterm/pull/7542)。

---

## 特性

- **仅输出便携版（Portable Zip）**：纯绿色包解压即用，剔除安装程序依赖（无需 Inno Setup），体积轻量且构建更快。
- **默认构建 upstream `main` 分支**：保证获取最新上游特性与修复。
- **自动每月运行**：通过 Cron（每月 1 号 `0 0 1 * *`）自动定时触发编译与发布。
- **多渠道补丁支持**：
  - **动态链接**：直接输入 GitHub PR 链接（如 `https://github.com/wezterm/wezterm/pull/7542/changes`）、Commit 链接或任意 Patch URL，自动抓取并规范化。
  - **本地文件**：支持提交本地补丁至 `patches/*.patch` 目录自动加载。
  - **防冲突机制**：内置 `git apply --reverse --check` 智能跳过已包含补丁，并支持 3-way merge 回退。
- **自动化 Release**：每次编译自动创建 GitHub Release，附带构建信息、SHA256 校验码及补丁变更清单。

---

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── build-wezterm.yml                  # 自动编译与 Release 工作流
├── patches/
│   └── pr-7542-fix-kitty-keyboard-mux.patch   # PR #7542 本地补丁
├── .gitignore
└── README.md
```

---

## 使用与触发

### 1. 自动触发
- **每月定时**：每月 1 号 00:00 (UTC) 自动抓取 `main` 最新代码并编译发布。
- **代码提交**：向 `main` 分支提交工作流修改或 `patches/` 变更时自动构建。

### 2. 手动触发 (workflow_dispatch)

前往 GitHub 仓库 -> **Actions** -> **Build WezTerm Windows (Patched)** -> **Run workflow**：

| 参数 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `wezterm_version` | 要编译的分支/Tag（支持 `main`、Tag 或 commit hash） | `main` |
| `patches` | 补丁列表（每行一个 URL，支持 PR 链接、commit 链接或 `.patch`） | `https://github.com/wezterm/wezterm/pull/7542/changes` |
| `tag_name` | 自定义 Release Tag（留空则自动生成 `main-YYYYMMDD-HHMMSS`） | *(留空自动生成)* |
| `release_name` | Release 标题 | `Custom WezTerm Windows Portable Build` |
| `prerelease` | 是否标记为 Pre-release | `false` |
| `run_tests` | 是否运行 cargo 测试 | `false` |

---

## 产物说明

Release 仅包含绿色便携包：
- `WezTerm-windows-<tag>.zip`：解压即用，包含 `wezterm.exe`、`wezterm-gui.exe`、`wezterm-mux-server.exe` 及必需的运行时动态库（ANGLE、ConPTY、Mesa OpenGL 回退）。
- `WezTerm-windows-<tag>.zip.sha256`：SHA256 校验和文件。
