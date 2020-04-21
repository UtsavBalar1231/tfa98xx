# We can build either as part of a standalone Kernel build or as
# an external module.  Determine which mechanism is being used
ifeq ($(MODNAME),)
	KERNEL_BUILD := 1
else
	KERNEL_BUILD := 0
endif



ifeq ($(KERNEL_BUILD), 1)
	# These are configurable via Kconfig for kernel-based builds
	# Need to explicitly configure for Android-based builds
	AUDIO_BLD_DIR := $(ANDROID_BUILD_TOP)/kernel/msm-4.14
	AUDIO_ROOT := $(AUDIO_BLD_DIR)/techpack/audio
endif

ifeq ($(KERNEL_BUILD), 0)
	ifeq ($(CONFIG_ARCH_SM8150), y)
		include $(AUDIO_ROOT)/config/sm8150auto.conf
		export
		INCS    +=  -include $(AUDIO_ROOT)/config/sm8150autoconf.h
	endif
	ifeq ($(CONFIG_ARCH_SM6150), y)
		include $(AUDIO_ROOT)/config/sm8150auto.conf
		export
		INCS    +=  -include $(AUDIO_ROOT)/config/sm8150autoconf.h
	endif

	ifeq ($(CONFIG_ARCH_SDMSHRIKE), y)
		include $(AUDIO_ROOT)/config/sm8150auto.conf
		export
		INCS    +=  -include $(AUDIO_ROOT)/config/sm8150autoconf.h
	endif
endif

# As per target team, build is done as follows:
# Defconfig : build with default flags
# Slub      : defconfig  + CONFIG_SLUB_DEBUG := y +
#	      CONFIG_SLUB_DEBUG_ON := y + CONFIG_PAGE_POISONING := y
# Perf      : Using appropriate msmXXXX-perf_defconfig
#
# Shipment builds (user variants) should not have any debug feature
# enabled. This is identified using 'TARGET_BUILD_VARIANT'. Slub builds
# are identified using the CONFIG_SLUB_DEBUG_ON configuration. Since
# there is no other way to identify defconfig builds, QTI internal
# representation of perf builds (identified using the string 'perf'),
# is used to identify if the build is a slub or defconfig one. This
# way no critical debug feature will be enabled for perf and shipment
# builds. Other OEMs are also protected using the TARGET_BUILD_VARIANT
# config.

############ UAPI ############
UAPI_DIR :=	uapi
UAPI_INC :=	-I$(AUDIO_ROOT)/include/$(UAPI_DIR)

############ COMMON ############
COMMON_DIR :=	include
COMMON_INC :=	-I$(AUDIO_ROOT)/$(COMMON_DIR)
TFA_INC    :=   -I$(AUDIO_ROOT)/asoc/codecs/tfa98xx/inc

# for TFA9874 Amplifer
ifdef CONFIG_SND_SOC_TFA9874
	TFA98XX_OBJS += src/tfa98xx.o
	TFA98XX_OBJS += src/tfa_container.o
	TFA98XX_OBJS += src/tfa_dsp.o
	TFA98XX_OBJS += src/tfa_init.o
endif

LINUX_INC += -Iinclude/linux

INCS +=	$(COMMON_INC) \
	$(UAPI_INC) \
	$(TFA_INC)

EXTRA_CFLAGS += -I$(src)/inc

CDEFINES +=	-DANI_LITTLE_BYTE_ENDIAN \
		-DANI_LITTLE_BIT_ENDIAN \
		-DDOT11F_LITTLE_ENDIAN_HOST \
		-DANI_COMPILER_TYPE_GCC \
		-DANI_OS_TYPE_ANDROID=6 \
		-DPTT_SOCK_SVC_ENABLE \
		-Wall\
		-Werror\
		-D__linux__ \
		-DTFA_NON_DSP_SOLUTION

KBUILD_CPPFLAGS += $(CDEFINES)

# Currently, for versions of gcc which support it, the kernel Makefile
# is disabling the maybe-uninitialized warning.  Re-enable it for the
# AUDIO driver.  Note that we must use EXTRA_CFLAGS here so that it
# will override the kernel settings.
ifeq ($(call cc-option-yn, -Wmaybe-uninitialized),y)
EXTRA_CFLAGS += -Wmaybe-uninitialized
endif
#EXTRA_CFLAGS += -Wmissing-prototypes

ifeq ($(call cc-option-yn, -Wheader-guard),y)
EXTRA_CFLAGS += -Wheader-guard
endif

ifeq ($(KERNEL_BUILD), 0)
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/ipc/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/dsp/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/asoc/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/asoc/codecs/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/soc/Module.symvers
endif

# Module information used by KBuild framework
obj-$(CONFIG_SND_SOC_TFA9874)    += tfa98xx_dlkm.o
tfa98xx_dlkm-y := $(TFA98XX_OBJS)
# inject some build related information
