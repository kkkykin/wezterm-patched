# 本地补丁目录（可选）

此目录用于存放**可选的本地 patch 文件**。

- 推荐方式：直接在工作流文件 `.github/workflows/build-wezterm.yml` 的 `env.PATCH_URLS` 中添加 GitHub PR 页面或 patch 链接，CI 会自动拉取并应用。
- 本地方式：如果你有未提交到 GitHub 的自定义修改，可直接将导出的 `*.patch` 或 `*.diff` 文件放置在此目录并提交，CI 构建时会自动检测并应用。
