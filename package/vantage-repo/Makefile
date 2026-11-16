# Runtime repository configuration for the Vantage userspace packages.
# The feed is architecture-independent; the key and repository are installed
# into the image so LuCI can display and apk can update the theme later.

include $(TOPDIR)/rules.mk

PKG_NAME:=vantage-repo
PKG_VERSION:=1
PKG_RELEASE:=2
PKG_LICENSE:=GPL-3.0-or-later

include $(INCLUDE_DIR)/package.mk

define Package/vantage-repo
  SECTION:=base
  CATEGORY:=Base system
  TITLE:=Vantage theme userspace repository configuration
  PKGARCH:=all
endef

define Package/vantage-repo/description
 Installs the Vantage signing key and registers its userspace apk repository
 in the LuCI-editable customfeeds.list. No kernel repository is added.
endef

define Build/Compile
endef

define Package/vantage-repo/install
	$(INSTALL_DIR) $(1)/etc/apk/keys
	$(INSTALL_DATA) ./files/vantage-signing.pem $(1)/etc/apk/keys/vantage-signing.pem
	$(INSTALL_DIR) $(1)/usr/share/vantage-repo
	$(INSTALL_DATA) ./files/vantage.list $(1)/usr/share/vantage-repo/repository.list
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files/95-vantage-repo $(1)/etc/uci-defaults/95-vantage-repo
endef

$(eval $(call BuildPackage,vantage-repo))
