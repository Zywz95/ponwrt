# 本 Fork 的自动同步与构建

本仓库跟踪 `pbs05/ponwrt` 的 `master`。Fork 专用配置和说明分别位于 `.github/workflows/sync-upstream.yml` 与本文件。上游源码和原有 README 保持原样；`.github/workflows/release.yml` 仅包含下述发布权限修复。

## 自动检查

`Sync upstream and build` 在 GitHub Actions 云端每小时整点检查上游（cron：`0 * * * *`），不依赖本地电脑开机或本地定时任务。上游没有新提交时只进行 API 检查，不重复下载依赖或编译。

发现新提交后，工作流合并上游并调用原有的 `Release PonWrt`，编译上游定义的全部目标，目前为 AN7581 和 AN7583。发布标签采用 `upstream-<上游完整提交 SHA>`，标签固定本次构建的源码。相同上游提交已经有构建任务时不再重复触发；构建失败后可在对应运行页面选择 Re-run failed jobs。

只监视 `pbs05/ponwrt` 的主分支；外部 feeds 仓库单独变化不会触发新构建。也可以在 Actions 页面手动运行同步工作流。

## honk

`package/honk` 把 honk 打包进 AN7583 固件，并自动安装为 `/usr/bin/honk-core`、`/etc/init.d/honk`、`/etc/config/honk` 和 `/etc/honk/config.dae`。服务默认停用；修改配置后执行 `/etc/init.d/honk enable && /etc/init.d/honk start`，再从局域网访问 `http://<路由器IP>:9527/`。

honk 本体使用上游官方发布的 aarch64 静态 musl `stock-full` 构建（含 Clash API、native API 和 doona 界面），由上游 CI 从源码编译；本仓库只负责校验、打包和装进固件。这样不在 OpenWrt 构建中引入 Rust nightly、bpf-linker 和 BoringSSL 交叉编译链，避免固件构建随机失败。版本、文件名和 SHA-256 固定在 `package/honk/release.mk`。

运行时要求：Linux 6.12+（本目标为 6.18）、root、bpffs 和内核 BTF。`configs/an7583.config` 已启用 `CONFIG_KERNEL_DEBUG_INFO_BTF`，并关闭会阻止 BTF 的 `CONFIG_KERNEL_DEBUG_INFO_REDUCED`；`CGROUP_BPF` 原本已启用，TC/TCX 所需的 `NET_CLS_ACT` 由 `kmod-sched-core` 选择。发布工作流会校验内核 vmlinux 的 `.BTF` 段、`honk` 软件包和固件 manifest，任一缺失都会让构建失败，避免产出“装了但不能运行”的固件。

`Sync honk release` 每小时检查 `daeuniverse/honk` 的 Releases（包括 beta/rc 预发布，排除 debug 标签）。发现新版本后更新 `release.mk`，提交并用新标签触发 `Release PonWrt` 编译全部目标。跨仓库 Release 没有可用的直接 webhook（需要 honk 仓库的管理权限），所以采用定时轮询；GitHub 定时任务可能延迟。

## 固件与缓存

编译成功后，固件归档和校验值发布到本仓库 Releases，也可从 Actions Artifacts 下载；Artifacts 保留 7 天。

发布前先创建并校验标签，确保指向实际编译的提交，再用 `gh release create --verify-tag` 发布，不传 `--target`。这避免构建期间默认分支更新工作流后，GitHub 对旧提交触发额外的 Workflows 写权限检查而返回 403；不需要额外保存个人访问令牌。

每次构建使用新的 GitHub 托管运行环境。系统依赖按需重新安装，feeds 重新拉取；工具链和编译产物没有持久化缓存，因此仍会重新编译。

上游发布工作流使用 `actions/cache` 缓存源码下载目录 `dl/`。GitHub 缓存按分支或标签隔离，不同标签不能复用彼此创建的缓存。当前自动构建为每个上游版本使用独立标签，所以不能保证跨版本复用下载缓存；同一标签的重新运行可复用该标签已有的缓存。缓存仅在构建任务成功结束后保存，首次构建或缓存被清理后需要重新下载。

## 同步与调度限制

自动同步使用合并，不强制覆盖分支。目前新增 `sync-upstream.yml`、`honk-release.yml`、`package/honk/` 和本文件，并对 `release.yml` 的发布步骤作最小修复；若上游修改同一发布逻辑或新增同名文件，仍可能发生冲突，届时同步任务会报错。

GitHub 定时任务可能排队延迟，整点尤其容易出现延迟。公开仓库连续 60 天没有活动时，GitHub 会自动禁用定时工作流，届时需要在 Actions 页面重新启用。
