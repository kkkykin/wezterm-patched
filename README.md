# WezTerm Custom Windows Build CI

参考 [`custom-caddy`](../custom-caddy) 架构实现的自动化 GitHub Actions 编译与 Release 发布工作流，用于自动编译带有自定义补丁的 Windows 平台 [WezTerm](https://github.com/wezterm/wezterm)。

默认集成补丁：[PR #7542: fix kitty keyboard not work in mux mode](https://github.com/wezterm/wezterm/pull/7542)。

---

## 特性

- **仅针对 Windows 平台**：针对 `x86_64-pc-windows-msvc` 进行编译，采用官方静态链接方案（`-C target-feature=+crt-static`）。
- **多种补丁方式支持**：
  - **本地补丁**：直接放置在 `patches/*.patch` 或 `patches/*.diff` 目录。
  - **动态链接**：支持在 GitHub Actions 页面直接输入 GitHub PR 链接（如 `https://github.com/wezterm/wezterm/pull/7542/changes`、`https://github.com/wezterm/wezterm/pull/7542`）、Commit 链接或任意 Patch URL。
  - **安全防冲突机制**：自动检测补丁是否已合并或已应用，支持智能 3-way merge，避免重复打补丁中断流程。
- **构建产物完整**：
  - 便携版压缩包：`WezTerm-windows-<tag>.zip`（包含 `wezterm.exe`、`wezterm-gui.exe`、`wezterm-mux-server.exe` 及 ANGLE、ConPTY、Mesa 等运行时）。
  - 安装程序：`WezTerm-<tag>-setup.exe`（通过 Inno Setup 6 自动生成）。
  - 自动生成每个文件的 `SHA256` 校验和。
- **自动 Release 发布**：编译成功后自动创建 GitHub Release，上传安装包、绿色包及校验码，并格式化输出补丁列表和版本详情。
- **多触发模式**：
  - `workflow_dispatch` 手动触发，支持自定义版本、补丁和 Tag。
  - `schedule` 每周定时自动触发构建。
  - `push` 提交到 `main` 分支触发。

---

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── build-wezterm.yml   # 编译与发布工作流
├── patches/
│   └── pr-7542-fix-kitty-keyboard-mux.patch # PR 7542 补丁
├── .gitignore
└── README.md
```

---

## 使用方法

### 1. 手动触发构建 (workflow_dispatch)

进入 GitHub 仓库页面 -> **Actions** -> 选择 **Build WezTerm Windows (Patched)** -> 点击 **Run workflow**：

| 参数 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `wezterm_version` | 要编译的 upstream 分支、Tag 或 commit（填 `latest` 自动获取官方最新 release） | `main` |
| `patches` | 需要打上的补丁列表（每行一个 URL，支持 PR 链接、commit 链接或 `.patch`） | `https://github.com/wezterm/wezterm/pull/7542/changes` |
| `tag_name` | 自定义 Release Tag 名称（留空自动使用 `<wezterm_version>-YYYYMMDD-HHMMSS`） | *(留空自动生成)* |
| `release_name` | Release 标题 | `Custom WezTerm Windows Build` |
| `prerelease` | 是否标记为 Pre-release | `false` |
| `run_tests` | 打包前是否运行 cargo test（耗时较长，默认跳过） | `false` |

### 2. 添加其它补丁

有两种便捷方式：

#### 方式 A：提交本地补丁到 `patches/` 目录
将导出的 `.patch` 或 `.diff` 文件保存至 `patches/` 目录下（如 `patches/0002-my-feature.patch`），推送至 GitHub 即可自动识别并应用。

#### 方式 B：在触发构建时输入链接
直接在 `patches` 输入框中填入 PR 页面地址（如 `https://github.com/wezterm/wezterm/pull/7542/changes`），工作流会自动解析并下载标准补丁。
