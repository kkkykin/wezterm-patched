# 本地补丁目录（可选）

此目录用于存放**可选的本地 patch 文件**。

- 推荐方式：直接在工作流文件 `.github/workflows/build-wezterm.yml` 的 `env.PATCH_URLS` 中添加 GitHub PR 页面或 patch 链接，CI 会自动拉取并应用。
- 本地方式：如果你有未提交到 GitHub 的自定义修改，可直接将导出的 `*.patch` 或 `*.diff` 文件放置在此目录并提交，CI 构建时会自动检测并应用。

CI 先按 `PATCH_URLS` 的顺序应用远程补丁，再按文件名顺序应用本地 `*.patch`，最后应用 `*.diff`。

## Kitty 协议的 Shift 修复

[`0001-fix-kitty-shift-in-mux.patch`](0001-fix-kitty-shift-in-mux.patch) 是 [PR #7542](https://github.com/wezterm/wezterm/pull/7542) 的后续修复，**必须先应用该 PR**（或使用已经包含其修改的源码）。手动覆盖工作流的 `patches` 输入时，也需要保留此依赖。

修复两处信息丢失：

- GUI 将按键转交给 pane 时，从原始事件补回字符归一化移除的 Shift；快捷键匹配仍使用原有归一化结果，Alt/Ctrl 保留字符组合后的状态。
- mux 服务端在 Kitty 编码之后才执行旧编码的 Shift 删除逻辑，保留字母、标点的组合修饰键。

开启 Kitty 消歧义模式时，`Alt+Shift+a` 应输出 `ESC[97;4u`，而不是表示 `Alt+a` 的 `ESC[97;3u`。回归测试还覆盖 `Ctrl+Shift`、三个修饰键同时按下、方向键、备用键码及旧编码行为。

使用时，**GUI 客户端和 mux 服务端都应包含修复**，服务端配置 `enable_kitty_keyboard = true`，并由终端应用请求启用 Kitty 协议。此补丁不增加 mux 的 key-up 传输，也不扩展 tmux 的协议支持。

## 已验证的版本与测试

- WezTerm upstream：`cab25161054c50fd6c705db4ceefef0f1e5a9575`
- PR #7542：`2b76b4904ee80f323c940933c8a9a7ca6e3379b5`
- Linux / Rust 1.99.0：`termwiz` 48 项、`wezterm-input-types` 15 项单元测试通过。
- Windows MSVC 目标：两个模块及其测试代码的 `cargo check` 通过（交叉编译检查，未运行 Windows 二进制）。
- 修复前新增回归测试可复现 `ESC[97;3u` / `ESC[97;4u` 不一致。
- 干净源码上验证了 CI 的远程 PR → 本地补丁应用顺序，以及各补丁重复应用时的跳过逻辑。

在应用两个补丁后的源码根目录运行低资源测试，无需初始化图形相关 submodule，也无需编译完整 GUI：

```bash
CARGO_BUILD_JOBS=1 CARGO_INCREMENTAL=0 \
CARGO_PROFILE_DEV_DEBUG=0 CARGO_PROFILE_TEST_DEBUG=0 \
cargo test --locked -p wezterm-input-types -p termwiz --lib --no-default-features
```

可选的 Windows 目标检查（无需链接 GUI）：

```bash
rustup target add x86_64-pc-windows-msvc
CARGO_BUILD_JOBS=1 CARGO_INCREMENTAL=0 \
CARGO_PROFILE_DEV_DEBUG=0 CARGO_PROFILE_TEST_DEBUG=0 \
cargo check --locked -p wezterm-input-types -p termwiz --tests \
  --no-default-features --target x86_64-pc-windows-msvc
```

这些测试验证字符归一化、转发使用的修饰键和服务端编码逻辑；Windows GUI 实际按键、远程 mux 连接仍需实机验收。
