#
# OpenWrt package: honk (https://github.com/daeuniverse/honk)
#
# honk is a Rust eBPF transparent proxy. Building it inside the OpenWrt
# buildroot needs a pinned Rust nightly, bpf-linker, BoringSSL and the doona
# frontend, which makes firmware builds slow and fragile. This package instead
# installs the official static musl release binary published by upstream CI.
# The version, tag, file name and checksum live in release.mk and are updated
# by .github/workflows/honk-release.yml when a new honk release appears.
#
# Runtime requirements handled here:
#   - Linux 6.12+ (PONWrt AN7583 uses 6.18)
#   - kernel BTF, enabled through CONFIG_KERNEL_DEBUG_INFO_BTF=y
#   - BPF, cgroup BPF, TC/TCX, nftables queue and veth kernel support
#
# This file is free software, licensed under the GNU General Public License v3.

PKG_DIR:=$(patsubst %/,%,$(dir $(abspath $(lastword $(MAKEFILE_LIST)))))

include $(TOPDIR)/rules.mk
include $(PKG_DIR)/release.mk

PKG_NAME:=honk
PKG_VERSION:=$(HONK_VERSION)
PKG_RELEASE:=1

PKG_SOURCE:=$(HONK_SOURCE)
PKG_SOURCE_URL:=https://github.com/daeuniverse/honk/releases/download/$(HONK_GIT_TAG)
PKG_HASH:=$(HONK_HASH)

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-3.0-only

include $(INCLUDE_DIR)/package.mk

define Package/honk
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=eBPF transparent proxy engine (honk-core, static release)
  URL:=https://github.com/daeuniverse/honk
  LICENSE:=GPL-3.0-only
  # The binary is static musl. The kernel modules provide the TC/TCX datapath,
  # the dae0 veth pair, NFQUEUE staging and conntrack; the geo packages supply
  # the databases linked into /usr/share/honk.
  DEPENDS:=+kmod-veth +kmod-sched-core +kmod-sched-bpf \
           +kmod-nft-core +kmod-nft-queue +kmod-nfnetlink-queue \
           +kmod-nf-conntrack +v2ray-geoip +v2ray-geosite
endef

define Package/honk/description
 honk is an experimental eBPF transparent proxy engine for Linux, with the
 Clash API, the native API and the embedded doona dashboard. The transparent
 datapath needs Linux 6.12+, kernel BTF and root. The service is installed but
 disabled; configure /etc/honk/config.dae before enabling it.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(TAR) -xzf $(DL_DIR)/$(PKG_SOURCE) -C $(PKG_BUILD_DIR) --strip-components=1
	find $(PKG_BUILD_DIR) -type f -name honk-core -print -quit | grep -q .
endef

define Package/honk/conffiles
/etc/config/honk
/etc/honk/config.dae
endef

define Package/honk/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) "$$(find $(PKG_BUILD_DIR) -type f -name honk-core -print -quit)" \
		$(1)/usr/bin/honk-core
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_DIR)/files/honk.init $(1)/etc/init.d/honk
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) $(PKG_DIR)/files/honk.config $(1)/etc/config/honk
	$(INSTALL_DIR) $(1)/etc/honk
	$(INSTALL_CONF) $(PKG_DIR)/files/config.dae $(1)/etc/honk/config.dae
	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	printf '%s\n' '/etc/honk/data/' > $(1)/lib/upgrade/keep.d/honk
	$(INSTALL_DIR) $(1)/usr/share/honk
	$(LN) ../v2ray/geoip.dat $(1)/usr/share/honk/geoip.dat
	$(LN) ../v2ray/geosite.dat $(1)/usr/share/honk/geosite.dat
endef

$(eval $(call BuildPackage,honk))
