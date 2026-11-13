# 本 Fork 的自动同步与构建

本仓库跟踪 `pbs05/ponwrt` 的 `master`。Fork 维护自动同步和独立构建配置；上游源码和原有 README 保持原样，发布工作流增加 feeds 准备、构建检查及标签发布修复。

## 自动检查

`Sync upstream and build` 在 GitHub Actions 云端每小时整点检查上游（cron：`0 * * * *`），不依赖本地电脑开机或本地定时任务。上游没有新提交时只进行 API 检查，不重复下载依赖或编译。

发现新提交后，工作流合并上游并调用原有的 `Release PonWrt`，编译上游定义的全部目标，目前为 AN7581 和 AN7583。发布标签采用 `upstream-<上游完整提交 SHA>`，标签固定本次构建的源码。相同上游提交已经有构建任务时不再重复触发；构建失败后可在对应运行页面选择 Re-run failed jobs。

只监视 `pbs05/ponwrt` 的主分支；外部 feeds 仓库单独变化不会触发新构建。也可以在 Actions 页面手动运行同步工作流。

## honk

AN7583 固件通过官方 daeuniverse/openwrt-packages feed 构建并预装 `honk`、`luci-lib-daeuniverse`、`luci-app-honk` 和 `luci-i18n-daeuniverse-zh-cn`。中文语言包由共享库的 `po/zh_Hans` 生成，在 LuCI 的“服务 → honk”中管理服务状态、配置文件和服务日志。本仓库不再保留 `package/honk/` 或 LuCI 源码副本。

每次构建启动时，`Resolve latest honk and Vantage releases` 查询最新已发布版本（包括 beta/rc、排除草稿和 debug 滚动标签），解析 honk、Vantage 的实际源码提交及官方 daeuniverse feed 的最新 main 提交。两个目标共享同一份结果，构建过程中不再漂移。实际版本和提交会写入固件归档中的 `source-versions.json`。构建先安装基础 feeds，再追加官方 honk feed，并只选择 honk、两个 LuCI 包和 `vmlinux-btf-builtin`；honk 源码检出到 feed 的 `honk/src`。init、UCI 配置和默认配置由官方 feed 提供。

**不会安装 `dae`、`luci-app-dae` 或 `luci-app-daed`**：dae 与 honk 会争用 `dae0`/`daens` 和相同接口，二者不能同时运行。配置和 manifest 检查都会拒绝这些包。服务默认停用；修改配置后执行 `/etc/init.d/honk enable && /etc/init.d/honk start`，也可从 LuCI 页面操作。

honk 本体和 eBPF 对象由本次 Actions 从源码构建，使用该源码要求的 Rust stable/nightly 及校验 SHA-256 的 bpf-linker；doona 界面由官方构建脚本下载并校验。功能为官方 feed 的 full 构建（包括 Clash API、native API 和嵌入式界面），链接本次 PONWrt 的 musl 与 C++ 运行库。相比复用官方二进制，首次构建时间、磁盘和网络需求更高。

honk 的构建选项放在独立的 `configs/honk.config`，仅在 AN7583 目标构建时合并；上游 `configs/an7583.config` 保持原样，减少同步冲突。

运行时要求：Linux 6.12+（当前目标为 6.18）、root、bpffs 和内核 BTF。`configs/honk.config` 启用内核 BTF，并选择 `vmlinux-btf-builtin` 满足官方依赖，不安装需要额外 BTF 文件的 `vmlinux-btf`。官方依赖的 `v2ray-geoip`、`v2ray-geosite`、`flock` 和 `coreutils-timeout` 由 PONWrt 基础仓库或 packages feed 提供。配置及产物检查会校验 BTF、honk、中文 LuCI、地理数据和 dae 禁用状态。

主题使用 `luci-theme-vantage` 和中文包 `luci-i18n-vantage-theme-zh-cn`，通过构建启动时解析的最新 Vantage Release 对应 feed 提交安装，不固定为仓库编写时的版本。主题自带的 uci-defaults 在首次安装时将默认主题设为 Vantage。`configs/theme.config` 关闭 argon 及其配置包；两个编译目标都会应用该片段。

`Sync honk release` 每小时检查 `daeuniverse/honk` 的 Releases（包括 beta/rc 预发布，排除 debug 标签）。`configs/honk-release.json` 只记录上次观察到的发布版本，用于判断是否触发构建，不限制本次构建版本。发现新版本后更新记录并触发 `Release PonWrt`；实际构建会重新查询 honk、Vantage 和官方 feed 的最新版本。Vantage 单独发版不额外触发构建，但下一次构建会自动使用它的最新版。GitHub 定时任务可能延迟。

## 固件与缓存

编译成功后，固件归档和校验值发布到本仓库 Releases，也可从 Actions Artifacts 下载；Artifacts 保留 7 天。

发布前先创建并校验标签，确保指向实际编译的提交，再用 `gh release create --verify-tag` 发布，不传 `--target`。这避免构建期间默认分支更新工作流后，GitHub 对旧提交触发额外的 Workflows 写权限检查而返回 403；不需要额外保存个人访问令牌。

每次构建使用新的 GitHub 托管运行环境。系统依赖按需重新安装，feeds 重新拉取；工具链和编译产物没有持久化缓存，因此仍会重新编译。

上游发布工作流使用 `actions/cache` 缓存源码下载目录 `dl/`。GitHub 缓存按分支或标签隔离，不同标签不能复用彼此创建的缓存。当前自动构建为每个上游版本使用独立标签，所以不能保证跨版本复用下载缓存；同一标签的重新运行可复用该标签已有的缓存。缓存仅在构建任务成功结束后保存，首次构建或缓存被清理后需要重新下载。

## 同步与调度限制

自动同步使用合并，不强制覆盖分支。本 Fork 只维护工作流、发版观察记录、独立配置片段和说明文件；honk、LuCI 包及 Vantage 主题均由构建时解析的官方 feeds 引入。`sync-upstream` 与 `honk-release` 共用 `master-write` 并发组，避免同时写 master。若上游修改 `release.yml` 的同一段或新增同名文件，仍可能冲突，届时同步任务会报错。

GitHub 定时任务可能排队延迟，整点尤其容易出现延迟。公开仓库连续 60 天没有活动时，GitHub 会自动禁用定时工作流，届时需要在 Actions 页面重新启用。
