# Fork 变更记录

本文记录从官方 `2022` 分支提交 `15f6d855` 拉取代码后，本 Fork 所做的主要变更。

## 当前发布版本

- 最终可用版本：`v2022.0.16`
- 对应提交：`7c31efd2`
- Release 页面：<https://github.com/lhp9916/RedisDesktopManager/releases/tag/v2022.0.16>
- 发布文件：
  - `RESP-2022.0.16-windows-x64-setup.exe`
  - `RESP-2022.0.16-windows-x64-portable.zip`
  - `SHA256SUMS.txt`

## 1. 新增 Windows 自动构建与发布

新增 `.github/workflows/windows-release.yml`，用于在 GitHub Actions 中自动构建 Windows x64 版本。

工作流支持：

- 推送 `v*` 标签时自动触发
- 在 Actions 页面手动触发
- 自动计算发布版本和安装器版本号
- 安装 Qt 5.15.2、MSVC 和 Python 3.8
- 编译 zlib、LZ4、Zstandard、Snappy 和 Brotli
- 编译 RESP 主体和 qredisclient 自带的 hiredis
- 安装 Python 运行时依赖
- 编译 `.ts` 翻译文件为 `.qm`
- 部署 Python、Qt QML、Windows 平台插件和 OpenSSL
- 生成便携 ZIP 和 NSIS 安装包
- 生成 SHA256 校验文件
- 上传 Actions artifact 并自动创建 GitHub Release

## 2. 修复 Windows 启动与运行库问题

构建和发布过程中修复了以下 Windows 问题：

- 删除已过时的 `hiredis-win.patch` 步骤，当前 qredisclient 版本直接编译内置 hiredis。
- 为 Python 3.8 的 `python-lzf` 编译启用当前 MSVC SDK。
- 避免在部署目录重复复制整套 QML 模块，解决 QtQuick Controls 1/2 注册冲突导致的“进程存在但没有窗口”。
- 同时部署 OpenSSL 的未加后缀和 Qt 兼容的 `-x64.dll` 文件名，恢复 Qt TLS 支持。
- 将部署流程拆分为独立步骤，便于定位 Python、翻译、Qt 和运行库问题。

## 3. 修复中文翻译加载

修改 `src/app/app.cpp`：

- 支持通过 `RESP_UI_LOCALE` 环境变量显式指定 UI 语言。
- 优先从程序目录的 `translations` 文件夹加载 `.qm`。
- 外部翻译不存在时，回退到程序内置的 `:/translations/` 资源。

构建流程现在会在 qmake 执行前编译所有 `.ts`，并把生成的 `.qm` 部署到 `translations` 目录。

## 4. 精简 GitHub Actions

删除不再需要的官方工作流：

- `.github/workflows/run_tests.yml`
- `.github/workflows/sonar.yml`

仓库现在只保留 `.github/workflows/windows-release.yml`。

## 5. 发布验证

`v2022.0.16` 的 GitHub Actions 构建已验证：

- Windows x64 编译成功
- QML 主界面加载成功
- Windows 平台插件加载成功
- OpenSSL TLS 加载成功
- `zh_CN` 中文翻译加载成功
- 便携 ZIP、NSIS 安装包和 SHA256 文件生成成功
- GitHub Release 自动发布成功

## 6. 历史产物说明

迭代过程中创建过多个中间标签。它们用于触发构建和排查问题，其中部分 Release 保留在 GitHub 上：

- `v2022.0.6`：早期可启动版本，但中文和 TLS 支持不完整。
- `v2022.0.10`：修复主窗口问题，但尚未包含最终 OpenSSL 命名兼容处理。
- `v2022.0.12`：修复启动、QML 和 TLS，但尚未包含最终中文翻译修复。
- `v2022.0.16`：当前推荐版本，包含全部已确认修复。

建议用户始终下载最新版本 `v2022.0.16`。
