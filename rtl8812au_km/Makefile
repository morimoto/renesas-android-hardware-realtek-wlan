
# Delete default rules. We don't use them. This saves a bit of time.
.SUFFIXES:

# we want bash as shell
SHELL := $(shell if [ -x "$$BASH" ]; then echo $$BASH; \
	 else if [ -x /bin/bash ]; then echo /bin/bash; \
	 else echo sh; fi; fi)

# Set O variable if not already done on the command line;
# or avoid confusing packages that can use the O=<dir> syntax for out-of-tree
# build by preventing it from being forwarded to sub-make calls.
ifneq ("$(origin O)", "command line")
O := $(CURDIR)/output
endif

override O := $(patsubst %/,%,$(patsubst %.,%,$(O)))
# Make sure $(O) actually exists before calling realpath on it; this is to
# avoid empty CANONICAL_O in case on non-existing entry.
CANONICAL_O := $(shell mkdir -p $(O) >/dev/null 2>&1)$(realpath $(O))

# gcc fails to build when the srcdir contains a '@'
ifneq ($(findstring @,$(CANONICAL_O)),)
$(error The build directory can not contain a '@')
endif

CANONICAL_CURDIR = $(realpath $(CURDIR))

REQ_UMASK = 0022

# Make sure O= is passed (with its absolute canonical path) everywhere the
# toplevel makefile is called back.
EXTRAMAKEARGS := O=$(CANONICAL_O)

ifneq ($(shell umask):$(CURDIR):$(O),$(REQ_UMASK):$(CANONICAL_CURDIR):$(CANONICAL_O))

.PHONY: _all $(MAKECMDGOALS)

$(MAKECMDGOALS): _all
	@:

_all:
	@umask $(REQ_UMASK) && \
		$(MAKE) -C $(CANONICAL_CURDIR) --no-print-directory \
			$(MAKECMDGOALS) $(EXTRAMAKEARGS)

else # umask / $(CURDIR) / $(O)

# This is our default rule, so must come first
all: modules

.PHONY: all

# Set and export the version string
export BR2_VERSION := 2018.11.1
# Actual time the release is cut (for reproducible builds)
BR2_VERSION_EPOCH = 1545257000

# Save running make version since it's clobbered by the make package
RUNNING_MAKE_VERSION := $(MAKE_VERSION)

# Check for minimal make version (note: this check will break at make 10.x)
MIN_MAKE_VERSION = 3.81
ifneq ($(firstword $(sort $(RUNNING_MAKE_VERSION) $(MIN_MAKE_VERSION))),$(MIN_MAKE_VERSION))
$(error You have make '$(RUNNING_MAKE_VERSION)' installed. GNU make >= $(MIN_MAKE_VERSION) is required)
endif

# absolute path
TOPDIR := $(CURDIR)
CONFIG_CONFIG_IN = Kconfig
CONFIG = support/kconfig
DATE := $(shell date +%Y%m%d)

# List of targets and target patterns for which .config doesn't need to be read in
noconfig_targets := menuconfig nconfig gconfig xconfig config oldconfig randconfig \
	defconfig %_defconfig allyesconfig allnoconfig alldefconfig syncconfig \
	print-version olddefconfig distclean

# Some global targets do not trigger a build, but are used to collect
# metadata, or do various checks. When such targets are triggered,
# some packages should not do their configuration sanity
# checks. Provide them a BR_BUILDING variable set to 'y' when we're
# actually building and they should do their sanity checks.
#
# We're building in two situations: when MAKECMDGOALS is empty
# (default target is to build), or when MAKECMDGOALS contains
# something else than one of the nobuild_targets.
nobuild_targets := clean distclean help \
	list-defconfigs \
	savedefconfig
ifeq ($(MAKECMDGOALS),)
BR_BUILDING = y
else ifneq ($(filter-out $(nobuild_targets),$(MAKECMDGOALS)),)
BR_BUILDING = y
endif

# Include some helper macros and variables
-include support/misc/utils.mk

# Set variables related to in-tree or out-of-tree build.
# Here, both $(O) and $(CURDIR) are absolute canonical paths.
ifeq ($(O),$(CURDIR)/output)
CONFIG_DIR := $(CURDIR)
NEED_WRAPPER =
else
CONFIG_DIR := $(O)
NEED_WRAPPER = y
endif

BASE_DIR := $(CANONICAL_O)
$(if $(BASE_DIR),, $(error output directory "$(O)" does not exist))


# To make sure that the environment variable overrides the .config option,
# set this before including .config.
ifneq ($(BR2_DL_DIR),)
DL_DIR := $(BR2_DL_DIR)
endif
ifneq ($(BR2_CCACHE_DIR),)
BR_CACHE_DIR := $(BR2_CCACHE_DIR)
endif

BUILD_DIR := $(BASE_DIR)/build

BR2_CONFIG = $(CONFIG_DIR)/.config

# Pull in the user's configuration file
ifeq ($(filter $(noconfig_targets),$(MAKECMDGOALS)),)
-include $(BR2_CONFIG)
endif

# timezone and locale may affect build output
ifeq ($(BR2_REPRODUCIBLE),y)
export TZ = UTC
export LANG = C
export LC_ALL = C
export GZIP = -n
endif

# To put more focus on warnings, be less verbose as default
# Use 'make V=1' to see the full commands
ifeq ("$(origin V)", "command line")
  KBUILD_VERBOSE = $(V)
endif
ifndef KBUILD_VERBOSE
  KBUILD_VERBOSE = 0
endif

ifeq ($(KBUILD_VERBOSE),1)
  Q =
ifndef VERBOSE
  VERBOSE = 1
endif
export VERBOSE
else
  Q = @
endif

# kconfig uses CONFIG_SHELL
CONFIG_SHELL := $(SHELL)

export SHELL CONFIG_SHELL Q KBUILD_VERBOSE

ifndef HOSTAR
HOSTAR := ar
endif
ifndef HOSTAS
HOSTAS := as
endif
ifndef HOSTCC
HOSTCC := gcc
HOSTCC := $(shell which $(HOSTCC) || type -p $(HOSTCC) || echo gcc)
endif
HOSTCC_NOCCACHE := $(HOSTCC)
ifndef HOSTCXX
HOSTCXX := g++
HOSTCXX := $(shell which $(HOSTCXX) || type -p $(HOSTCXX) || echo g++)
endif
HOSTCXX_NOCCACHE := $(HOSTCXX)
ifndef HOSTCPP
HOSTCPP := cpp
endif
ifndef HOSTLD
HOSTLD := ld
endif
ifndef HOSTLN
HOSTLN := ln
endif
ifndef HOSTNM
HOSTNM := nm
endif
ifndef HOSTOBJCOPY
HOSTOBJCOPY := objcopy
endif
ifndef HOSTRANLIB
HOSTRANLIB := ranlib
endif
HOSTAR := $(shell which $(HOSTAR) || type -p $(HOSTAR) || echo ar)
HOSTAS := $(shell which $(HOSTAS) || type -p $(HOSTAS) || echo as)
HOSTCPP := $(shell which $(HOSTCPP) || type -p $(HOSTCPP) || echo cpp)
HOSTLD := $(shell which $(HOSTLD) || type -p $(HOSTLD) || echo ld)
HOSTLN := $(shell which $(HOSTLN) || type -p $(HOSTLN) || echo ln)
HOSTNM := $(shell which $(HOSTNM) || type -p $(HOSTNM) || echo nm)
HOSTOBJCOPY := $(shell which $(HOSTOBJCOPY) || type -p $(HOSTOBJCOPY) || echo objcopy)
HOSTRANLIB := $(shell which $(HOSTRANLIB) || type -p $(HOSTRANLIB) || echo ranlib)
SED := $(shell which sed || type -p sed) -i -e

export HOSTAR HOSTAS HOSTCC HOSTCXX HOSTLD
export HOSTCC_NOCCACHE HOSTCXX_NOCCACHE

export HOSTARCH := $(shell LC_ALL=C $(HOSTCC_NOCCACHE) -v 2>&1 | \
	sed -e '/^Target: \([^-]*\).*/!d' \
	    -e 's//\1/' \
	    -e 's/i.86/x86/' \
	    -e 's/sun4u/sparc64/' \
	    -e 's/arm.*/arm/' \
	    -e 's/sa110/arm/' \
	    -e 's/ppc64/powerpc64/' \
	    -e 's/ppc/powerpc/' \
	    -e 's/macppc/powerpc/' \
	    -e 's/sh.*/sh/' )

# When adding a new host gcc version in Config.in,
# update the HOSTCC_MAX_VERSION variable:
HOSTCC_MAX_VERSION := 8

HOSTCC_VERSION := $(shell V=$$($(HOSTCC_NOCCACHE) --version | \
	sed -n -r 's/^.* ([0-9]*)\.([0-9]*)\.([0-9]*)[ ]*.*/\1 \2/p'); \
	[ "$${V%% *}" -le $(HOSTCC_MAX_VERSION) ] || V=$(HOSTCC_MAX_VERSION); \
	printf "%s" "$${V}")

# For gcc >= 5.x, we only need the major version.
ifneq ($(firstword $(HOSTCC_VERSION)),4)
HOSTCC_VERSION := $(firstword $(HOSTCC_VERSION))
endif

ifeq ($(BR2_HAVE_DOT_CONFIG),y)

################################################################################
#
# Hide troublesome environment variables from sub processes
#
################################################################################
unexport CROSS_COMPILE
unexport ARCH
unexport CC
unexport LD
unexport AR
unexport CXX
unexport CPP
unexport RANLIB
unexport CFLAGS
unexport CXXFLAGS
unexport GREP_OPTIONS
unexport TAR_OPTIONS
unexport CONFIG_SITE
unexport QMAKESPEC
unexport TERMINFO
unexport MACHINE
unexport O
unexport GCC_COLORS

# silent mode requested?
QUIET := $(if $(findstring s,$(filter-out --%,$(MAKEFLAGS))),-q)

# Strip off the annoying quoting
ARCH := $(call qstrip,$(BR2_ARCH))

KERNEL_ARCH := $(shell echo "$(ARCH)" | sed -e "s/-.*//" \
	-e s/i.86/i386/ -e s/sun4u/sparc64/ \
	-e s/arcle/arc/ \
	-e s/arceb/arc/ \
	-e s/arm.*/arm/ -e s/sa110/arm/ \
	-e s/aarch64.*/arm64/ \
	-e s/or1k/openrisc/ \
	-e s/parisc64/parisc/ \
	-e s/powerpc64.*/powerpc/ \
	-e s/ppc.*/powerpc/ -e s/mips.*/mips/ \
	-e s/riscv.*/riscv/ \
	-e s/sh.*/sh/ \
	-e s/microblazeel/microblaze/)

ZCAT := $(call qstrip,$(BR2_ZCAT))
BZCAT := $(call qstrip,$(BR2_BZCAT))
XZCAT := $(call qstrip,$(BR2_XZCAT))
LZCAT := $(call qstrip,$(BR2_LZCAT))
TAR_OPTIONS = $(call qstrip,$(BR2_TAR_OPTIONS)) -xf

# Scripts in support/ or post-build scripts may need to reference
# these locations, so export them so it is easier to use
export BR2_CONFIG
export BR2_REPRODUCIBLE
export TARGET_DIR
export STAGING_DIR
export HOST_DIR
export BINARIES_DIR
export BASE_DIR

endif # ifeq ($(BR2_HAVE_DOT_CONFIG),y)
HOSTCFLAGS = $(CFLAGS_FOR_BUILD)
export HOSTCFLAGS

$(BUILD_DIR)/driver-config/%onf:
	$(Q)mkdir -p $(@D)/lxdialog
	$(Q) PKG_CONFIG_PATH="$(HOST_PKG_CONFIG_PATH)" $(MAKE) CC="$(HOSTCC_NOCCACHE)" HOSTCC="$(HOSTCC_NOCCACHE)" \
	    obj=$(@D) -C $(CONFIG) -f Makefile.br $(@F)

DEFCONFIG = $(call qstrip,$(BR2_DEFCONFIG))

# We don't want to fully expand BR2_DEFCONFIG here, so Kconfig will
# recognize that if it's still at its default $(CONFIG_DIR)/defconfig
COMMON_CONFIG_ENV = \
	BR2_DEFCONFIG='$(call qstrip,$(value BR2_DEFCONFIG))' \
	KCONFIG_AUTOCONFIG=$(BUILD_DIR)/driver-config/auto.conf \
	KCONFIG_AUTOHEADER=$(BUILD_DIR)/driver-config/autoconf.h \
	KCONFIG_TRISTATE=$(BUILD_DIR)/driver-config/tristate.config \
	BR2_CONFIG=$(BR2_CONFIG) \
	HOST_GCC_VERSION="$(HOSTCC_VERSION)" \
	BUILD_DIR=$(BUILD_DIR) \
	SKIP_LEGACY=

menuconfig: $(BUILD_DIR)/driver-config/mconf
	@$(COMMON_CONFIG_ENV) $< $(CONFIG_CONFIG_IN)

config: $(BUILD_DIR)/driver-config/conf
	@$(COMMON_CONFIG_ENV) $< $(CONFIG_CONFIG_IN)

randconfig allyesconfig alldefconfig allnoconfig: $(BUILD_DIR)/driver-config/conf
	@$(COMMON_CONFIG_ENV) SKIP_LEGACY=y $< --$@ $(CONFIG_CONFIG_IN)
	@$(COMMON_CONFIG_ENV) $< --olddefconfig $(CONFIG_CONFIG_IN) >/dev/null

oldconfig syncconfig olddefconfig: $(BUILD_DIR)/driver-config/conf
	@$(COMMON_CONFIG_ENV) $< --$@ $(CONFIG_CONFIG_IN)

defconfig: $(BUILD_DIR)/driver-config/conf
	@$(COMMON_CONFIG_ENV) $< --defconfig$(if $(DEFCONFIG),=$(DEFCONFIG)) $(CONFIG_CONFIG_IN)

define percent_defconfig
# Override the BR2_DEFCONFIG from COMMON_CONFIG_ENV with the new defconfig
%_defconfig: $(BUILD_DIR)/driver-config/conf $(1)/configs/%_defconfig
	@$$(COMMON_CONFIG_ENV) BR2_DEFCONFIG=$(1)/configs/$$@ \
		$$< --defconfig=$(1)/configs/$$@ $$(CONFIG_CONFIG_IN)
endef
$(eval $(foreach d,$(call reverse,$(TOPDIR) $(BR2_EXTERNAL_DIRS)),$(call percent_defconfig,$(d))$(sep)))

savedefconfig: $(BUILD_DIR)/driver-config/conf
	@$(COMMON_CONFIG_ENV) $< \
		--savedefconfig=$(if $(DEFCONFIG),$(DEFCONFIG),$(CONFIG_DIR)/defconfig) \
		$(CONFIG_CONFIG_IN)
	@$(SED) '/BR2_DEFCONFIG=/d' $(if $(DEFCONFIG),$(DEFCONFIG),$(CONFIG_DIR)/defconfig)

.PHONY: defconfig savedefconfig

.PHONY: clean
clean: modclean
	@ echo "Cleaning configuration artifacts..."
	$(Q) rm -rf $(BUILD_DIR)

.PHONY: distclean
distclean: clean
ifeq ($(O),$(CURDIR)/output)
	$(Q) rm -rf $(O)
endif
	$(Q) rm -rf $(TOPDIR)/dl $(BR2_CONFIG) $(CONFIG_DIR)/.config.old $(CONFIG_DIR)/..config.tmp \
		$(CONFIG_DIR)/.auto.deps $(BR2_EXTERNAL_FILE)

.PHONY: help
help:
	@echo 'Cleaning:'
	@echo '  clean                  - delete all files created by build'
	@echo '  distclean              - delete all non-source files (including .config)'
	@echo
	@echo 'Build:'
	@echo '  all                    - make world'
	@echo '  toolchain              - build toolchain'
	@echo '  sdk                    - build relocatable SDK'
	@echo
	@echo 'Configuration:'
	@echo '  menuconfig             - interactive curses-based configurator'
	@echo '  oldconfig              - resolve any unresolved symbols in .config'
	@echo '  syncconfig             - Same as oldconfig, but quietly, additionally update deps'
	@echo '  olddefconfig           - Same as syncconfig but sets new symbols to their default value'
	@echo '  randconfig             - New config with random answer to all options'
	@echo '  defconfig              - New config with default answer to all options;'
	@echo '                             BR2_DEFCONFIG, if set on the command line, is used as input'
	@echo '  savedefconfig          - Save current config to BR2_DEFCONFIG (minimal config)'
	@echo '  allyesconfig           - New config where all options are accepted with yes'
	@echo '  allnoconfig            - New config where all options are answered with no'
	@echo '  alldefconfig           - New config where all options are set to default'

# List the defconfig files
# $(1): base directory
# $(2): br2-external name, empty for bundled
define list-defconfigs
	@first=true; \
	for defconfig in $(1)/configs/*_defconfig; do \
		[ -f "$${defconfig}" ] || continue; \
		if $${first}; then \
			if [ "$(2)" ]; then \
				printf 'External configs in "$(call qstrip,$(2))":\n'; \
			else \
				printf "Built-in configs:\n"; \
			fi; \
			first=false; \
		fi; \
		defconfig="$${defconfig##*/}"; \
		printf "  %-35s - Build for %s\n" "$${defconfig}" "$${defconfig%_defconfig}"; \
	done; \
	$${first} || printf "\n"
endef

# We iterate over BR2_EXTERNAL_NAMES rather than BR2_EXTERNAL_DIRS,
# because we want to display the name of the br2-external tree.
.PHONY: list-defconfigs
list-defconfigs:
	$(call list-defconfigs,$(TOPDIR))
	$(foreach name,$(BR2_EXTERNAL_NAMES),\
		$(call list-defconfigs,$(BR2_EXTERNAL_$(name)_PATH),\
			$(BR2_EXTERNAL_$(name)_DESC))$(sep))

.PHONY: $(noconfig_targets)

############### DRIVER BUILD #############
-include $(WORKDIR)/.config

ifeq ($(CONFIG_USB_HCI), y)
	HCI_NAME = usb
endif

ifeq ($(CONFIG_GSPI_HCI), y)
	HCI_NAME = gspi
endif

ifeq ($(CONFIG_SDIO_HCI), y)
	HCI_NAME = sdio
endif

ifeq ($(CONFIG_PCI_HCI), y)
	HCI_NAME = pci
endif

_OS_INTFS_FILES := os_dep/osdep_service.o \
		os_dep/linux/os_intfs.o \
		os_dep/linux/$(HCI_NAME)_intf.o \
		os_dep/linux/$(HCI_NAME)_ops_linux.o \
		os_dep/linux/ioctl_linux.o \
		os_dep/linux/xmit_linux.o \
		os_dep/linux/mlme_linux.o \
		os_dep/linux/recv_linux.o \
		os_dep/linux/ioctl_cfg80211.o \
		os_dep/linux/rtw_cfgvendor.o \
		os_dep/linux/wifi_regd.o \
		os_dep/linux/rtw_android.o \
		os_dep/linux/rtw_radiotap.o \
		os_dep/linux/rtw_proc.o

ifeq ($(CONFIG_SDIO_HCI), y)
	_OS_INTFS_FILES += os_dep/linux/custom_gpio_linux.o
	_OS_INTFS_FILES += os_dep/linux/$(HCI_NAME)_ops_linux.o
endif

ifeq ($(CONFIG_GSPI_HCI), y)
	_OS_INTFS_FILES += os_dep/linux/custom_gpio_linux.o
	_OS_INTFS_FILES += os_dep/linux/$(HCI_NAME)_ops_linux.o
endif


_HAL_INTFS_FILES := hal/hal_intf.o \
		hal/hal_com.o \
		hal/hal_com_phycfg.o \
		hal/hal_phy.o \
		hal/hal_dm.o \
		hal/hal_btcoex.o \
		hal/hal_mp.o \
		hal/hal_mcc.o \
		hal/hal_hci/hal_$(HCI_NAME).o \
		hal/led/hal_$(HCI_NAME)_led.o


_OUTSRC_FILES := hal/phydm/phydm_debug.o	\
		hal/phydm/phydm_antdiv.o\
		hal/phydm/phydm_antdect.o\
		hal/phydm/phydm_interface.o\
		hal/phydm/phydm_hwconfig.o\
		hal/phydm/phydm.o\
		hal/phydm/halphyrf_ce.o\
		hal/phydm/phydm_edcaturbocheck.o\
		hal/phydm/phydm_dig.o\
		hal/phydm/phydm_pathdiv.o\
		hal/phydm/phydm_rainfo.o\
		hal/phydm/phydm_dynamicbbpowersaving.o\
		hal/phydm/phydm_powertracking_ce.o\
		hal/phydm/phydm_dynamictxpower.o\
		hal/phydm/phydm_adaptivity.o\
		hal/phydm/phydm_cfotracking.o\
		hal/phydm/phydm_noisemonitor.o\
		hal/phydm/phydm_acs.o\
		hal/phydm/phydm_beamforming.o\
		hal/phydm/phydm_dfs.o\
		hal/phydm/txbf/halcomtxbf.o\
		hal/phydm/txbf/haltxbfinterface.o\
		hal/phydm/txbf/phydm_hal_txbf_api.o\
		hal/phydm/phydm_kfree.o\
		hal/phydm/phydm_ccx.o


EXTRA_CFLAGS += -I$(src)/platform
_PLATFORM_FILES := platform/platform_ops.o

ifeq ($(CONFIG_BT_COEXIST), y)
	EXTRA_CFLAGS += -I$(src)/hal/btc
	_OUTSRC_FILES += hal/btc/HalBtc8192e1Ant.o \
			hal/btc/HalBtc8192e2Ant.o \
			hal/btc/HalBtc8723b1Ant.o \
			hal/btc/HalBtc8723b2Ant.o \
			hal/btc/HalBtc8812a1Ant.o \
			hal/btc/HalBtc8812a2Ant.o \
			hal/btc/HalBtc8821a1Ant.o \
			hal/btc/HalBtc8821a2Ant.o \
			hal/btc/HalBtc8821aCsr2Ant.o \
			hal/btc/HalBtc8703b1Ant.o \
			hal/btc/halbtc8723d1ant.o \
			hal/btc/halbtc8723d2ant.o \
			hal/btc/HalBtc8822b1Ant.o \
			hal/btc/halbtc8821c1ant.o \
			hal/btc/halbtc8821c2ant.o
endif

########### HAL_RTL8192E #################################
ifeq ($(CONFIG_RTL8192E), y)
	RTL871X = rtl8192e
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8192es
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8192eu
	endif

	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8192ee
	endif
	EXTRA_CFLAGS += -DCONFIG_RTL8192E
	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8192EPwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_xmit.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/hal8192e_fw.o \
			hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_recv.o

	ifeq ($(CONFIG_SDIO_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	else
		ifeq ($(CONFIG_GSPI_HCI), y)
			_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
		else
			_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
		endif
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8192E_USB.o
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8192E_PCIE.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8192e_mac.o\
		hal/phydm/$(RTL871X)/halhwimg8192e_bb.o\
		hal/phydm/$(RTL871X)/halhwimg8192e_rf.o\
		hal/phydm/$(RTL871X)/halhwimg8192e_fw.o\
		hal/phydm/$(RTL871X)/halphyrf_8192e_ce.o\
		hal/phydm/$(RTL871X)/phydm_regconfig8192e.o\
		hal/phydm/$(RTL871X)/phydm_rtl8192e.o

endif

########### HAL_RTL8812A_RTL8821A #################################

ifneq ($(CONFIG_RTL8812A)_$(CONFIG_RTL8821A), n_n)

	RTL871X = rtl8812a
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8812au
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8812ae
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8812as
	endif

	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8812PwrSeq.o \
			hal/$(RTL871X)/Hal8821APwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_xmit.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES +=hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_recv.o

	ifeq ($(CONFIG_SDIO_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	else
		ifeq ($(CONFIG_GSPI_HCI), y)
			_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
		else
			_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
		endif
	endif

	ifeq ($(CONFIG_RTL8812A), y)
		ifeq ($(CONFIG_USB_HCI), y)
			_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8812A_USB.o
		endif
		ifeq ($(CONFIG_PCI_HCI), y)
			_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8812A_PCIE.o
		endif
	endif

	ifeq ($(CONFIG_RTL8821A), y)
		ifeq ($(CONFIG_USB_HCI), y)
			_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8821A_USB.o
		endif
		ifeq ($(CONFIG_PCI_HCI), y)
			_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8821A_PCIE.o
		endif
	endif

	ifeq ($(CONFIG_RTL8812A), y)
		EXTRA_CFLAGS += -DCONFIG_RTL8812A
		_HAL_INTFS_FILES +=	hal/rtl8812a/hal8812a_fw.o

		_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8812a_fw.o\
				hal/phydm/$(RTL871X)/halhwimg8812a_mac.o\
				hal/phydm/$(RTL871X)/halhwimg8812a_bb.o\
				hal/phydm/$(RTL871X)/halhwimg8812a_rf.o\
				hal/phydm/$(RTL871X)/halphyrf_8812a_ce.o\
				hal/phydm/$(RTL871X)/phydm_regconfig8812a.o\
				hal/phydm/$(RTL871X)/phydm_rtl8812a.o\
				hal/phydm/txbf/haltxbfjaguar.o
	endif

	ifeq ($(CONFIG_RTL8821A), y)
		ifeq ($(CONFIG_RTL8812A), n)
			RTL871X = rtl8821a
			ifeq ($(CONFIG_USB_HCI), y)
				ifeq ($(CONFIG_BT_COEXIST), y)
					MODULE_NAME := 8821au
				else
					MODULE_NAME := 8811au
				endif
			endif

			ifeq ($(CONFIG_PCI_HCI), y)
				MODULE_NAME := 8821ae
			endif
			ifeq ($(CONFIG_SDIO_HCI), y)
				MODULE_NAME := 8821as
			endif
		endif

	EXTRA_CFLAGS += -DCONFIG_RTL8821A

	_HAL_INTFS_FILES += hal/rtl8812a/hal8821a_fw.o
	_OUTSRC_FILES += hal/phydm/rtl8821a/halhwimg8821a_fw.o\
			hal/phydm/rtl8821a/halhwimg8821a_mac.o\
			hal/phydm/rtl8821a/halhwimg8821a_bb.o\
			hal/phydm/rtl8821a/halhwimg8821a_rf.o\
			hal/phydm/rtl8812a/halphyrf_8812a_ce.o\
			hal/phydm/rtl8821a/halphyrf_8821a_ce.o\
			hal/phydm/rtl8821a/phydm_regconfig8821a.o\
			hal/phydm/rtl8821a/phydm_rtl8821a.o\
			hal/phydm/rtl8821a/phydm_iqk_8821a_ce.o\
			hal/phydm/txbf/haltxbfjaguar.o
	endif
endif

########### HAL_RTL8723B #################################
ifeq ($(CONFIG_RTL8723B), y)
	RTL871X = rtl8723b
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8723bu
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8723be
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8723bs
	endif

	EXTRA_CFLAGS += -DCONFIG_RTL8723B

	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8723BPwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/hal8723b_fw.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_recv.o

	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
	else
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8723B_USB.o
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8723B_PCIE.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8723b_bb.o\
			hal/phydm/$(RTL871X)/halhwimg8723b_mac.o\
			hal/phydm/$(RTL871X)/halhwimg8723b_rf.o\
			hal/phydm/$(RTL871X)/halhwimg8723b_fw.o\
			hal/phydm/$(RTL871X)/halhwimg8723b_mp.o\
			hal/phydm/$(RTL871X)/phydm_regconfig8723b.o\
			hal/phydm/$(RTL871X)/halphyrf_8723b_ce.o\
			hal/phydm/$(RTL871X)/phydm_rtl8723b.o
endif

########### HAL_RTL8814A #################################
ifeq ($(CONFIG_RTL8814A), y)
	## ADD NEW VHT MP HW TX MODE ##
	EXTRA_CFLAGS += -DCONFIG_MP_VHT_HW_TX_MODE
	CONFIG_MP_VHT_HW_TX_MODE = y
	##########################################
	RTL871X = rtl8814a
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8814au
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8814ae
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8814as
	endif

	EXTRA_CFLAGS += -DCONFIG_RTL8814A

	_HAL_INTFS_FILES +=  hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8814PwrSeq.o \
			hal/$(RTL871X)/$(RTL871X)_xmit.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o
#			hal/$(RTL871X)/hal8814a_fw.o


	_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_recv.o

	ifeq ($(CONFIG_SDIO_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	else
		ifeq ($(CONFIG_GSPI_HCI), y)
			_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
		else
			HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
		endif
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8814A_USB.o
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8814A_PCIE.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8814a_bb.o\
			hal/phydm/$(RTL871X)/halhwimg8814a_mac.o\
			hal/phydm/$(RTL871X)/halhwimg8814a_rf.o\
			hal/phydm/$(RTL871X)/halhwimg8814a_fw.o\
			hal/phydm/$(RTL871X)/phydm_iqk_8814a.o\
			hal/phydm/$(RTL871X)/phydm_regconfig8814a.o\
			hal/phydm/$(RTL871X)/halphyrf_8814a_ce.o\
			hal/phydm/$(RTL871X)/phydm_rtl8814a.o\
			hal/phydm/txbf/haltxbf8814a.o

endif

########### HAL_RTL8723C #################################
ifeq ($(CONFIG_RTL8723C), y)
	RTL871X = rtl8703b
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8723cu
		MODULE_SUB_NAME = 8703bu
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8723ce
		MODULE_SUB_NAME = 8703be
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8723cs
		MODULE_SUB_NAME = 8703bs
	endif

	EXTRA_CFLAGS += -DCONFIG_RTL8703B

	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8703BPwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/hal8703b_fw.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_recv.o

	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
	else
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8703B_USB.o
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8703B_PCIE.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8703b_bb.o\
			hal/phydm/$(RTL871X)/halhwimg8703b_mac.o\
			hal/phydm/$(RTL871X)/halhwimg8703b_rf.o\
			hal/phydm/$(RTL871X)/halhwimg8703b_fw.o\
			hal/phydm/$(RTL871X)/phydm_regconfig8703b.o\
			hal/phydm/$(RTL871X)/halphyrf_8703b.o
endif

########### HAL_RTL8723D #################################
ifeq ($(CONFIG_RTL8723D), y)
	RTL871X = rtl8723d
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8723du
		MODULE_SUB_NAME = 8723du
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8723de
		MODULE_SUB_NAME = 8723de
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8723ds
		MODULE_SUB_NAME = 8723ds
	endif

	EXTRA_CFLAGS += -DCONFIG_RTL8723D

	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8723DPwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/hal8723d_fw.o \
			hal/$(RTL871X)/$(RTL871X)_lps_poff.o


	_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_SUB_NAME)_recv.o

	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
	else
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8723D_USB.o
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8723D_PCIE.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8723d_bb.o\
			hal/phydm/$(RTL871X)/halhwimg8723d_mac.o\
			hal/phydm/$(RTL871X)/halhwimg8723d_rf.o\
			hal/phydm/$(RTL871X)/halhwimg8723d_fw.o\
			hal/phydm/$(RTL871X)/phydm_regconfig8723d.o\
			hal/phydm/$(RTL871X)/phydm_rtl8723d.o\
			hal/phydm/$(RTL871X)/halphyrf_8723d.o
endif

########### HAL_RTL8188F #################################
ifeq ($(CONFIG_RTL8188F), y)
	RTL871X = rtl8188f
	ifeq ($(CONFIG_USB_HCI), y)
		MODULE_NAME = 8188fu
	endif
	ifeq ($(CONFIG_PCI_HCI), y)
		MODULE_NAME = 8188fe
	endif
	ifeq ($(CONFIG_SDIO_HCI), y)
		MODULE_NAME = 8189fs
	endif

	EXTRA_CFLAGS += -DCONFIG_RTL8188F

	_HAL_INTFS_FILES += hal/HalPwrSeqCmd.o \
			hal/$(RTL871X)/Hal8188FPwrSeq.o\
			hal/$(RTL871X)/$(RTL871X)_sreset.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(RTL871X)_hal_init.o \
			hal/$(RTL871X)/$(RTL871X)_phycfg.o \
			hal/$(RTL871X)/$(RTL871X)_rf6052.o \
			hal/$(RTL871X)/$(RTL871X)_dm.o \
			hal/$(RTL871X)/$(RTL871X)_rxdesc.o \
			hal/$(RTL871X)/$(RTL871X)_cmd.o \
			hal/$(RTL871X)/hal8188f_fw.o

	_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_halinit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_led.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_xmit.o \
			hal/$(RTL871X)/$(HCI_NAME)/rtl$(MODULE_NAME)_recv.o

	ifeq ($(CONFIG_PCI_HCI), y)
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops_linux.o
	else
		_HAL_INTFS_FILES += hal/$(RTL871X)/$(HCI_NAME)/$(HCI_NAME)_ops.o
	endif

	ifeq ($(CONFIG_USB_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8188F_USB.o
	endif

	ifeq ($(CONFIG_SDIO_HCI), y)
		_HAL_INTFS_FILES +=hal/efuse/$(RTL871X)/HalEfuseMask8188F_SDIO.o
	endif

	_OUTSRC_FILES += hal/phydm/$(RTL871X)/halhwimg8188f_bb.o\
			hal/phydm/$(RTL871X)/halhwimg8188f_mac.o\
			hal/phydm/$(RTL871X)/halhwimg8188f_rf.o\
			hal/phydm/$(RTL871X)/halhwimg8188f_fw.o\
			hal/phydm/$(RTL871X)/phydm_regconfig8188f.o\
			hal/phydm/$(RTL871X)/halphyrf_8188f.o \
			hal/phydm/$(RTL871X)/phydm_rtl8188f.o

endif

########### HAL_RTL8822B #################################
ifeq ($(CONFIG_RTL8822B), y)
	include $(shell pwd)/rtl8822b.mk
endif

########### HAL_RTL8821C #################################
ifeq ($(CONFIG_RTL8821C), y)
	include $(shell pwd)/rtl8821c.mk

	_OUTSRC_FILES += hal/phydm/rtl8821c/halhwimg8821c_testchip_bb.o \
			hal/phydm/rtl8821c/halhwimg8821c_testchip_mac.o \
			hal/phydm/rtl8821c/halhwimg8821c_testchip_rf.o \
			hal/phydm/rtl8821c/phydm_hal_api8821c.o \
			hal/phydm/rtl8821c/phydm_regconfig8821c.o\
			hal/phydm/rtl8821c/phydm_iqk_8821c.o
endif

_OS_INTFS_FILES += os_dep/linux/ioctl_mp.o core/rtw_mp.o

########### END OF PATH  #################################

##### DEBUG CONFIGURATION #####
ifeq ($(DEBUG), 1)
	EXTRA_CFLAGS += -DDBG=1 -DCONFIG_RTW_DEBUG -DCONFIG_DBG_COUNTER -DRTW_LOG_LEVEL=4
	EXTRA_CFLAGS += -DCONFIG_RADIOTAP_WITH_RXDESC
else ifeq ($(DEBUG), 2)
	EXTRA_CFLAGS += -DDBG=1 -DCONFIG_RTW_DEBUG -DCONFIG_DBG_COUNTER -DRTW_LOG_LEVEL=5
	EXTRA_CFLAGS += -DCONFIG_DEBUG_RTL871X
	EXTRA_CFLAGS += -DCONFIG_RADIOTAP_WITH_RXDESC
else
	EXTRA_CFLAGS += -DDBG=0
endif

##### Extra flags setup ######

EXTRA_CFLAGS += $(USER_EXTRA_CFLAGS) -O1
EXTRA_CFLAGS += -Wall
EXTRA_CFLAGS += -Wextra
EXTRA_CFLAGS += -Wno-unused-variable
EXTRA_CFLAGS += -Wno-unused-value
EXTRA_CFLAGS += -Wno-unused-label
EXTRA_CFLAGS += -Wno-unused-parameter
EXTRA_CFLAGS += -Wno-unused-function
EXTRA_CFLAGS += -Wno-unused
EXTRA_CFLAGS += -Wno-date-time
EXTRA_CFLAGS += -Wno-misleading-indentation
EXTRA_CFLAGS += -Wno-uninitialized
EXTRA_CFLAGS += -Wno-unknown-warning-option

GCC_VER_49 := $(shell echo `$(CC) -dumpversion | cut -f1-2 -d.` \>= 4.9 | bc )
ifeq ($(GCC_VER_49),1)
	EXTRA_CFLAGS += -Wno-date-time	# Fix compile error && warning on gcc 4.9 and later
endif
# Relax some warnings from '-Wextra' so we won't get flooded with warnings
EXTRA_CFLAGS += -Wno-sign-compare
EXTRA_CFLAGS += -Wno-missing-field-initializers
EXTRA_CFLAGS += -Wno-type-limits

EXTRA_CFLAGS += -I$(src)/include
EXTRA_CFLAGS += -I$(src)/hal/phydm

EXTRA_LDFLAGS += --strip-debug

EXTRA_CFLAGS += -DCONFIG_DISABLE_REGD_C
EXTRA_CFLAGS += -DCONFIG_CALIBRATE_TX_POWER_TO_MAX
EXTRA_CFLAGS += -DCONFIG_RTW_ADAPTIVITY_EN=0
EXTRA_CFLAGS += -DCONFIG_RTW_ADAPTIVITY_MODE=0
EXTRA_CFLAGS += -DCONFIG_MP_INCLUDED

ifeq ($(CONFIG_ENABLE_DRIVER_WARNINGS), y)
	EXTRA_CFLAGS += -DWARN_ENABLE=1
else
	EXTRA_CFLAGS += -DWARN_ENABLE=0
endif

ifeq ($(CONFIG_USB2_EXTERNAL_POWER), y)
	EXTRA_CFLAGS += -DCONFIG_USE_EXTERNAL_POWER
endif

ifeq ($(CONFIG_USB_HCI), y)
	ifeq ($(CONFIG_USB_AUTOSUSPEND), y)
		EXTRA_CFLAGS += -DCONFIG_USB_AUTOSUSPEND
	endif
endif

ifeq ($(CONFIG_POWER_SAVING), y)
	EXTRA_CFLAGS += -DCONFIG_POWER_SAVING
endif

ifeq ($(CONFIG_HW_PWRP_DETECTION), y)
	EXTRA_CFLAGS += -DCONFIG_HW_PWRP_DETECTION
endif

ifeq ($(CONFIG_BT_COEXIST), y)
	EXTRA_CFLAGS += -DCONFIG_BT_COEXIST
endif

ifeq ($(CONFIG_EFUSE_CONFIG_FILE), y)
	EXTRA_CFLAGS += -DCONFIG_EFUSE_CONFIG_FILE

	#EFUSE_MAP_PATH
	USER_EFUSE_MAP_PATH ?=
	ifneq ($(USER_EFUSE_MAP_PATH),)
		EXTRA_CFLAGS += -DEFUSE_MAP_PATH=\"$(USER_EFUSE_MAP_PATH)\"
	else ifeq ($(MODULE_NAME), 8189es)
		EXTRA_CFLAGS += -DEFUSE_MAP_PATH=\"/system/etc/wifi/wifi_efuse_8189e.map\"
	else ifeq ($(MODULE_NAME), 8723bs)
		EXTRA_CFLAGS += -DEFUSE_MAP_PATH=\"/system/etc/wifi/wifi_efuse_8723bs.map\"
	else
		EXTRA_CFLAGS += -DEFUSE_MAP_PATH=\"/system/etc/wifi/wifi_efuse_$(MODULE_NAME).map\"
	endif

	#WIFIMAC_PATH
	USER_WIFIMAC_PATH ?=
	ifneq ($(USER_WIFIMAC_PATH),)
		EXTRA_CFLAGS += -DWIFIMAC_PATH=\"$(USER_WIFIMAC_PATH)\"
	else
		EXTRA_CFLAGS += -DWIFIMAC_PATH=\"/data/wifimac.txt\"
	endif
endif

ifeq ($(CONFIG_80211W), y)
	EXTRA_CFLAGS += -DCONFIG_IEEE80211W
endif

ifeq ($(CONFIG_BR_EXT), y)
	BR_NAME = br0
	EXTRA_CFLAGS += -DCONFIG_BR_EXT
	EXTRA_CFLAGS += '-DCONFIG_BR_EXT_BRNAME="'$(BR_NAME)'"'
endif

ifeq ($(CONFIG_ANTENNA_DIVERSITY), y)
	EXTRA_CFLAGS += -DCONFIG_ANTENNA_DIVERSITY
endif

ifeq ($(CONFIG_TDLS), y)
	EXTRA_CFLAGS += -DCONFIG_TDLS
endif

EXTRA_CFLAGS += -DDM_ODM_SUPPORT_TYPE=0x04

EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
EXTRA_CFLAGS += -DCONFIG_IOCTL_CFG80211 -DRTW_USE_CFG80211_STA_EVENT
ARCH ?= arm64

ifneq ($(USER_MODULE_NAME),)
	MODULE_NAME := $(USER_MODULE_NAME)
endif

ifneq ($(KERNELRELEASE),)
	rtk_core := core/rtw_cmd.o \
		core/rtw_security.o \
		core/rtw_io.o \
		core/rtw_debug.o \
		core/rtw_ioctl_query.o \
		core/rtw_ioctl_set.o \
		core/rtw_ieee80211.o \
		core/rtw_mlme.o \
		core/rtw_mlme_ext.o \
		core/rtw_mi.o \
		core/rtw_wlan_util.o \
		core/rtw_vht.o \
		core/rtw_pwrctrl.o \
		core/rtw_rf.o \
		core/rtw_recv.o \
		core/rtw_sta_mgt.o \
		core/rtw_ap.o \
		core/rtw_p2p.o \
		core/rtw_tdls.o \
		core/rtw_xmit.o \
		core/rtw_br_ext.o \
		core/rtw_iol.o \
		core/rtw_sreset.o \
		core/rtw_btcoex.o \
		core/rtw_beamforming.o \
		core/rtw_odm.o \
		core/efuse/rtw_efuse.o

	ifeq ($(CONFIG_SDIO_HCI), y)
		rtk_core += core/rtw_sdio.o
	endif

	$(MODULE_NAME)-y += $(rtk_core)
	$(MODULE_NAME)-y += $(_OS_INTFS_FILES)
	$(MODULE_NAME)-y += $(_HAL_INTFS_FILES)
	$(MODULE_NAME)-y += $(_OUTSRC_FILES)
	$(MODULE_NAME)-y += $(_PLATFORM_FILES)

	obj-m := $(MODULE_NAME).o

else

########### COMMON  #################################

modules:
	make -C $(KERNELDIR) CROSS_COMPILE=$(CROSS_COMPILE) ARCH="arm64" M=$(WORKDIR) modules
.PHONY: modules modclean

modclean:
	$(Q) echo "Cleaning module build artifacts"
	$(Q) cd hal ; rm -fr */*/*/*.mod.c */*/*/*.mod */*/*/*.o */*/*/.*.cmd */*/*/*.ko
	$(Q) cd hal ; rm -fr */*/*.mod.c */*/*.mod */*/*.o */*/.*.cmd */*/*.ko
	$(Q) cd hal ; rm -fr */*.mod.c */*.mod */*.o */.*.cmd */*.ko
	$(Q) cd hal ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) cd core/efuse ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) cd core ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) cd os_dep/linux ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) cd os_dep ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) cd platform ; rm -fr *.mod.c *.mod *.o .*.cmd *.ko
	$(Q) rm -fr Module.symvers ; rm -fr Module.markers ; rm -fr modules.order
	$(Q) rm -fr *.mod.c *.mod *.o .*.cmd *.ko *~
	$(Q) rm -fr .tmp_versions
	$(Q) rm -rf .tmpconfig*
	$(Q) rm -rf *.tmp
endif

endif #umask / $(CURDIR) / $(O)
