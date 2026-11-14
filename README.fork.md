# 本 Fork 的自动同步与构建

本仓库跟踪 `pbs05/ponwrt` 的 `master`。Fork 维护自动同步、源码版本解析和构建配置；上游已有源码与目标配置保持原样。

## 构建目标

固件工作流同时构建 `an7581` 和 `an7583`。honk、LuCI 管理页面、中文语言包、Vantage 主题和运行时仓库配置通过独立配置片段及 feeds 进入两个目标；不安装 `dae`、`luci-app-dae` 或 `luci-app-daed`。

Nokia XG-040G-MF 使用 AN7583，刷机时必须选择 `an7583` 归档中的 `nokia_xg-040g-mf-ubi` 镜像。AN7581 归档中的 MD/TF 设备镜像不适用于 MF。

## honk

每次构建开始时，Actions 查询最新的 honk Release、Vantage Release 和 daeuniverse/openwrt-packages feed 提交。honk Release 包括 beta/rc 预发布，排除草稿和 debug 滚动版本；两个目标共享同一组解析结果，实际标签和源码提交写入固件归档的 `source-versions.json`。

honk 使用官方 feed 的源码规则构建：源码检出到 feed 的 `honk/src`，安装源码指定的 Rust stable/nightly、校验过的 bpf-linker，并构建 eBPF 对象、honk-core 和嵌入式 doona UI。运行时要求 Linux 6.12+、root、bpffs 和内核 BTF；当前 PONwrt 内核为 6.18。

默认安装：

- `honk`
- `luci-lib-daeuniverse`
- `luci-app-honk`
- `luci-i18n-daeuniverse-zh-cn`
- `vmlinux-btf-builtin`
- `v2ray-geoip`、`v2ray-geosite`
- `daeuniverse-repo`

`daeuniverse-repo` 只添加用户态 packages 仓库和签名公钥，不添加 daeuniverse kmods 仓库，避免替换自定义 PONwrt 内核。刷机后可更新：

```sh
apk update
apk upgrade honk@daeuniverse \
  luci-app-honk@daeuniverse \
  luci-i18n-daeuniverse-zh-cn@daeuniverse
```

honk 默认停用。配置 `/etc/honk/config.dae` 后执行 `/etc/init.d/honk enable && /etc/init.d/honk start`，LuCI 页面位于“服务 → honk”。

## Vantage 主题

Vantage 通过构建时 feed 安装，每次构建解析最新 Release，并同时安装中文包 `luci-i18n-vantage-theme-zh-cn`。主题自带的 uci-defaults 会在首次启动时设置 `/luci-static/vantage` 为默认主题。

`luci-theme-argon` 和 `luci-app-argon-config` 被明确禁用，并通过固件 manifest 校验不存在。

## 自动同步与发布

`Sync upstream and build` 每小时整点检查 `pbs05/ponwrt` 的 `master`。发现新提交后合并上游并触发完整构建；没有变化时不会重复编译。

`Sync honk release` 每小时检查 honk Releases。发现新版本后更新观察记录并触发构建；实际构建仍会重新解析当时的 honk、Vantage 和 feed 最新版本。

构建成功后发布正式 GitHub Release，不标记为 Pre-release。当前 Release 会包含 `an7581` 和 `an7583` 两个归档；Artifacts 保留 7 天。

两个写 master 的工作流共用 `master-write` 并发锁。同步冲突时不会强制覆盖分支，任务会失败并保留现状。

## 缓存与限制

每次构建使用新的 GitHub runner。`dl/` 下载目录使用 Actions cache，但工具链和编译产物不会持久化，因此源码构建仍可能耗时较长。定时任务可能因 GitHub 排队延迟；公开仓库连续 60 天无活动时，GitHub 可能自动停用定时工作流。
