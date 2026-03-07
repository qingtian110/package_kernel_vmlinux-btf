#
# Copyright (C) 2015-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v3.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=vmlinux-btf
PKG_VERSION:=6.12.71
PKG_RELEASE:=1

PKG_BUILD_PARALLEL:=1
PKG_MAINTAINER:=sbwml <admin@cooluc.com>

PKG_SOURCE:=vmlinux-btf-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=https://github.com/sbwml/package_kernel_vmlinux-btf/releases/download/v25.12.0
PKG_HASH:=24737db97b6e39f0329caa576b5e9dc028f91b6c907f7d7baa10fd49852b6947

include $(INCLUDE_DIR)/package.mk

define Package/vmlinux-btf/Default
  SECTION:=kernel
  CATEGORY:=Kernel
  TITLE:=Kernel vmlinux.btf for CO-RE compatibility
  URL:=https://github.com/sbwml/openwrt-vmlinux-btf
endef

define Package/vmlinux-btf/description/Default
  Supplies kernel BTF information to ensure CO-RE eBPF compatibility across kernel updates.
endef

define Package/vmlinux-btf/postinst/Default
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	ln -sf /usr/lib/debug/boot/vmlinux /usr/lib/debug/boot/vmlinux-$$(uname -r)
fi
endef

define Package/vmlinux-btf/prerm/Default
#!/bin/sh
[ -d "/usr/lib/debug" ] && rm -rf /usr/lib/debug
endef

define Package/vmlinux-btf/install/Default
	$(INSTALL_DIR) $(1)/usr/lib/debug/boot
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/vmlinux-$(PKG_VERSION)-$(VARIANT) $(1)/usr/lib/debug/boot/vmlinux
	$(LN) vmlinux $(1)/usr/lib/debug/boot/vmlinux-$(LINUX_VERSION)
endef

### aarch64 variant
define Package/vmlinux-btf-aarch64
  $(call Package/vmlinux-btf/Default)
  DEPENDS:=@TARGET_rockchip
  VARIANT:=aarch64_generic
endef

define Package/vmlinux-btf-aarch64/description
  $(call Package/vmlinux-btf/description/Default)
endef

define Package/vmlinux-btf-aarch64/prerm
$(call Package/vmlinux-btf/prerm/Default)
endef

define Package/vmlinux-btf-aarch64/postinst
$(call Package/vmlinux-btf/postinst/Default)
endef

define Package/vmlinux-btf-aarch64/install
  $(call Package/vmlinux-btf/install/Default,$(1))
endef

### x86 variant
define Package/vmlinux-btf-x86
  $(call Package/vmlinux-btf/Default)
  DEPENDS:=@TARGET_x86_64
  VARIANT:=x86_64
endef

define Package/vmlinux-btf-x86/description
  $(call Package/vmlinux-btf/description/Default)
endef

define Package/vmlinux-btf-x86/prerm
$(call Package/vmlinux-btf/prerm/Default)
endef

define Package/vmlinux-btf-x86/postinst
$(call Package/vmlinux-btf/postinst/Default)
endef

define Package/vmlinux-btf-x86/install
  $(call Package/vmlinux-btf/install/Default,$(1))
endef

define Build/Compile
endef

$(eval $(call BuildPackage,vmlinux-btf-aarch64))
$(eval $(call BuildPackage,vmlinux-btf-x86))
