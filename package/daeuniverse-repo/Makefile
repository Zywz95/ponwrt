#
# Runtime repository configuration for daeuniverse userspace packages.
#
# Only the userspace repository is installed. The daeuniverse kmods repository
# is intentionally excluded because this firmware uses a custom PONwrt kernel.

include $(TOPDIR)/rules.mk

PKG_NAME:=daeuniverse-repo
PKG_VERSION:=1
PKG_RELEASE:=2
PKG_LICENSE:=GPL-2.0-only

include $(INCLUDE_DIR)/package.mk

define Package/daeuniverse-repo
  SECTION:=base
  CATEGORY:=Base system
  TITLE:=daeuniverse userspace package repository configuration
  PKGARCH:=all
endef

define Package/daeuniverse-repo/description
 Installs the daeuniverse signing key and registers its userspace apk
 repository in the LuCI-editable customfeeds.list without replacing existing
 entries. No kmods repository is added for the custom PONwrt kernel.
endef


define Build/Compile
endef
define Package/daeuniverse-repo/install
	$(INSTALL_DIR) $(1)/etc/apk/keys
	$(INSTALL_DATA) ./files/daeuniverse.pem $(1)/etc/apk/keys/daeuniverse.pem
	$(INSTALL_DIR) $(1)/usr/share/daeuniverse-repo
	$(INSTALL_DATA) ./files/daeuniverse.list $(1)/usr/share/daeuniverse-repo/repository.list
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files/95-daeuniverse-repo $(1)/etc/uci-defaults/95-daeuniverse-repo
endef

$(eval $(call BuildPackage,daeuniverse-repo))
