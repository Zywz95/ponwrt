# Runtime repository key for the Vantage userspace packages.
# The repository entry is baked into the root overlay at
# files/etc/apk/repositories.d/customfeeds.list.

include $(TOPDIR)/rules.mk

PKG_NAME:=vantage-repo
PKG_VERSION:=1
PKG_RELEASE:=3
PKG_LICENSE:=GPL-3.0-or-later

include $(INCLUDE_DIR)/package.mk

define Package/vantage-repo
  SECTION:=base
  CATEGORY:=Base system
  TITLE:=Vantage userspace repository key
  PKGARCH:=all
endef

define Package/vantage-repo/description
 Installs the Vantage userspace repository signing key. The repository entry
 is part of the firmware overlay and no init-time script is installed.
endef

define Build/Compile
endef

define Package/vantage-repo/install
	$(INSTALL_DIR) $(1)/etc/apk/keys
	$(INSTALL_DATA) ./files/vantage-signing.pem $(1)/etc/apk/keys/vantage-signing.pem
endef

$(eval $(call BuildPackage,vantage-repo))
