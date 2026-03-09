#
# Copyright (C) 2015-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v3.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=vmlinux-btf
PKG_VERSION:=$(LINUX_VERSION)
PKG_PATCHVER:=$(word 1,$(subst ., ,$(PKG_VERSION))).$(word 2,$(subst ., ,$(PKG_VERSION)))

PKG_BUILD_DEPENDS:=dwarves/host
PKG_BUILD_PARALLEL:=1

PKG_SOURCE:=$(LINUX_SOURCE)
PKG_SOURCE_URL:=$(LINUX_SITE)
PKG_HASH:=$(LINUX_KERNEL_HASH)
PKG_BUILD_DIR:=$(KERNEL_BUILD_DIR)/vmlinux-btf/linux-$(PKG_VERSION)

GENERIC_PLATFORM_DIR := $(TOPDIR)/target/linux/generic
GENERIC_BACKPORT_DIR := $(GENERIC_PLATFORM_DIR)/backport$(if $(wildcard $(GENERIC_PLATFORM_DIR)/backport-$(PKG_PATCHVER)),-$(PKG_PATCHVER))
GENERIC_PATCH_DIR := $(GENERIC_PLATFORM_DIR)/pending$(if $(wildcard $(GENERIC_PLATFORM_DIR)/pending-$(PKG_PATCHVER)),-$(PKG_PATCHVER))
GENERIC_HACK_DIR := $(GENERIC_PLATFORM_DIR)/hack$(if $(wildcard $(GENERIC_PLATFORM_DIR)/hack-$(PKG_PATCHVER)),-$(PKG_PATCHVER))
GENERIC_FILES_DIR := $(foreach dir,$(wildcard $(GENERIC_PLATFORM_DIR)/files $(GENERIC_PLATFORM_DIR)/files-$(PKG_PATCHVER)),"$(dir)")

include $(INCLUDE_DIR)/package.mk

define Package/vmlinux-btf
  SECTION:=kernel
  CATEGORY:=Kernel
  TITLE:=Kernel vmlinux.btf for CO-RE compatibility
endef

define Package/vmlinux-btf/description
  BPF CO-RE stand for Compile Once - Run Everywhere It's a concept to
  build cross-version kernel eBPF application by building in a single
  binary by bringing together the BTF type information, libbpf and the
  compiler.
endef

define Package/vmlinux-btf/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	ln -sf /usr/lib/debug/boot/vmlinux /usr/lib/debug/boot/vmlinux-$$(uname -r)
fi
endef

define Package/vmlinux-btf/prerm
#!/bin/sh
[ -d "/usr/lib/debug" ] && rm -rf /usr/lib/debug
endef

define Build/Prepare
	$(call Build/Prepare/Default)

	[ -d "$(GENERIC_BACKPORT_DIR)" ] && $(foreach p,$(wildcard $(GENERIC_BACKPORT_DIR)/*.patch),$(PATCH) -p1 -d $(PKG_BUILD_DIR) < $(p);)
	[ -d "$(GENERIC_PATCH_DIR)" ] && $(foreach p,$(wildcard $(GENERIC_PATCH_DIR)/*.patch),$(PATCH) -p1 -d $(PKG_BUILD_DIR) < $(p);)
	[ -d "$(GENERIC_HACK_DIR)" ] && $(foreach p,$(wildcard $(GENERIC_HACK_DIR)/*.patch),$(PATCH) -p1 -d $(PKG_BUILD_DIR) < $(p);)

	$(foreach dir,$(GENERIC_FILES_DIR),[ -d $(dir) ] && $(CP) $(dir)/* $(PKG_BUILD_DIR)/;)
endef

define Build/Compile
	$(CP) $(LINUX_DIR)/.config $(PKG_BUILD_DIR)

	$(PKG_BUILD_DIR)/scripts/config --file $(PKG_BUILD_DIR)/.config \
		--disable DEBUG_INFO_NONE \
		--disable DEBUG_INFO_SPLIT \
		--disable DEBUG_INFO_REDUCED \
		--enable BPF_EVENTS \
		--enable BPF_JIT_ALWAYS_ON \
		--enable DEBUG_INFO_BTF \
		--enable MODULE_ALLOW_BTF_MISMATCH \
		--enable PAHOLE_HAS_LANG_EXCLUDE \
		--enable PAHOLE_HAS_SPLIT_BTF \
		--enable XDP_SOCKETS \
		--enable XDP_SOCKETS_DIAG

	+$(MAKE) -C $(PKG_BUILD_DIR) $(PKG_JOBS) $(KERNEL_MAKE_FLAGS) olddefconfig
	+$(MAKE) -C $(PKG_BUILD_DIR) $(PKG_JOBS) $(KERNEL_MAKE_FLAGS) vmlinux
	pahole --btf_encode_detached $(PKG_BUILD_DIR)/vmlinux.btf $(PKG_BUILD_DIR)/vmlinux
endef

define Package/vmlinux-btf/install
	$(INSTALL_DIR) $(1)/usr/lib/debug/boot
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/vmlinux.btf $(1)/usr/lib/debug/boot/vmlinux
	$(LN) vmlinux $(1)/usr/lib/debug/boot/vmlinux-$(LINUX_VERSION)
endef

$(eval $(call BuildPackage,vmlinux-btf))
