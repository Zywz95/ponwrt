# Runtime repository configuration for daeuniverse userspace packages.
# The repository entry is baked into the root overlay at
# files/etc/apk/repositories.d/customfeeds.list. This package only installs
# the signing key; it has no boot-time repository mutation script.

include $(TOPDIR)/rules.mk

PKG_NAME:=daeuniverse-repo
PKG_VERSION:=1
PKG_RELEASE:=3
PKG_LICENSE:=GPL-2.0-only

include $(INCLUDE_DIR)/package.mk

define Package/daeuniverse-repo
  SECTION:=base
  CATEGORY:=Base system
  TITLE:=daeuniverse userspace package repository key
  PKGARCH:=all
endef

define Package/daeuniverse-repo/description
 Installs the daeuniverse userspace repository signing key. The repository
 entry is part of the firmware overlay and no init-time script is installed.
endef

define Build/Compile
endef

define Package/daeuniverse-repo/install
	$(INSTALL_DIR) $(1)/etc/apk/keys
	$(INSTALL_DATA) ./files/daeuniverse.pem $(1)/etc/apk/keys/daeuniverse.pem
endef

$(eval $(call BuildPackage,daeuniverse-repo))
