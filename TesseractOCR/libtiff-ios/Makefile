PNG_NAME        := libpng-1.6.34
JPEG_SRC_NAME   := jpegsrc.v9c
# folder name after the JPEG_SRC_NAME archive has been unpacked
JPEG_DIR_NAME   := jpeg-9c
TIFF_NAME       := tiff-4.0.9

XCODE_DEVELOPER_PATH="`xcode-select -p`"
XCODETOOLCHAIN_PATH=$(XCODE_DEVELOPER_PATH)/Toolchains/XcodeDefault.xctoolchain

IMAGE_SRC = $(shell pwd)
PNG_SRC   = $(IMAGE_SRC)/$(PNG_NAME)
JPEG_SRC = $(IMAGE_SRC)/$(JPEG_DIR_NAME)
TIFF_SRC = $(IMAGE_SRC)/$(TIFF_NAME)

libpngfiles = libpng.a
libjpegfiles = libjpeg.a
libtifffiles = libtiff.a libtiffxx.a

libpngconfig  = $(PNG_SRC)/configure
libjpegconfig = $(JPEG_SRC)/configure
libtiffconfig = $(TIFF_SRC)/configure

index = $(words $(shell a="$(2)";echo $${a/$(1)*/$(1)} ))
swap  = $(word $(call index,$(1),$(2)),$(3))

# platform specific config
#
# make platform=ios
ifeq ($(platform), ios)
	PLATFORM_PREFIX=ios
	SDK_IPHONEOS_PATH=$(shell xcrun --sdk iphoneos --show-sdk-path)
	SDK_IPHONESIMULATOR_PATH=$(shell xcrun --sdk iphonesimulator --show-sdk-path)
	IOS_DEPLOY_TGT="10.0"
	PLATFORM_VERSION_MIN=iphoneos-version-min=$(IOS_DEPLOY_TGT)

	sdks = $(SDK_IPHONEOS_PATH) $(SDK_IPHONEOS_PATH) $(SDK_IPHONEOS_PATH) $(SDK_IPHONESIMULATOR_PATH) $(SDK_IPHONESIMULATOR_PATH)
	archs_all = armv7 armv7s arm64 i386 x86_64
	arch_names_all = arm-apple-darwin7 arm-apple-darwin7s arm-apple-darwin64 i386-apple-darwin x86_64-apple-darwin
# make platform=macos
else ifeq ($(platform), macos)
	PLATFORM_PREFIX=macos
	SDK_MACOS_PATH=$(shell xcrun --sdk macosx --show-sdk-path)
	MACOS_DEPLOY_TGT="10.10"
	PLATFORM_VERSION_MIN=macosx-version-min=$(MACOS_DEPLOY_TGT)

	sdks = $(SDK_MACOS_PATH) $(SDK_MACOS_PATH)
	archs_all = i386 x86_64
	arch_names_all = i386-apple-darwin x86_64-apple-darwin
# make platform=all
else ifeq ($(platform), all)
	# we will call make for all platforms, so nothing to do for now
endif

# TODO: Maybe dependencies dir can be removed as it's unnecessary scoping
IMAGE_LIB_DIR = $(shell pwd)/$(PLATFORM_PREFIX)/dependencies/lib/
IMAGE_INC_DIR = $(shell pwd)/$(PLATFORM_PREFIX)/dependencies/include/

arch_names = $(foreach arch, $(ARCHS), $(call swap, $(arch), $(archs_all), $(arch_names_all) ) )
ARCHS ?= $(archs_all)

libpngfolders  = $(foreach arch, $(arch_names), $(PNG_SRC)/$(arch)/)
libjpegfolders = $(foreach arch, $(arch_names), $(JPEG_SRC)/$(arch)/)
libtifffolders = $(foreach arch, $(arch_names), $(TIFF_SRC)/$(arch)/)

libpngfolders_all  = $(foreach arch, $(arch_names_all), $(PNG_SRC)/$(arch)/)
libjpegfolders_all = $(foreach arch, $(arch_names_all), $(JPEG_SRC)/$(arch)/)
libtifffolders_all = $(foreach arch, $(arch_names_all), $(TIFF_SRC)/$(arch)/)

libpngmakefile  = $(foreach folder, $(libpngfolders), $(addprefix $(folder), Makefile) )
libjpegmakefile = $(foreach folder, $(libjpegfolders), $(addprefix $(folder), Makefile) )
libtiffmakefile = $(foreach folder, $(libtifffolders), $(addprefix $(folder), Makefile) )

libpngfat  = $(addprefix $(IMAGE_LIB_DIR), $(libpngfiles))
libjpegfat = $(addprefix $(IMAGE_LIB_DIR), $(libjpegfiles))
libtifffat = $(addprefix $(IMAGE_LIB_DIR), $(libtifffiles))

libpng     = $(foreach folder, $(libpngfolders), $(addprefix $(folder)/lib/, $(libpngfiles)) )
libjpeg    = $(foreach folder, $(libjpegfolders), $(addprefix $(folder)/lib/, $(libjpegfiles)) )
libtiff    = $(foreach folder, $(libtifffolders), $(addprefix $(folder)/lib/, $(libtifffiles)) )

dependant_libs = libpng libjpeg libtiff

ifneq (,$(filter $(platform),ios macos))
.PHONY : all
all : $(dependant_libs)
else
.PHONY : all
all :
	$(MAKE) platform=ios
	$(MAKE) platform=macos
endif

#######################
# Build libtiff and all of its dependencies
#######################
libtiff : $(libtifffat)

$(libtifffat) : $(libtiff)
	mkdir -p $(@D)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libtifffolders_all)) ) -create -output $@
	mkdir -p $(IMAGE_INC_DIR)
	cp -rvf $(firstword $(libtifffolders))/include/*.h $(IMAGE_INC_DIR)

$(libtiff) :  $(libtiffmakefile)
	cd $(abspath $(@D)/..) ; \
	$(MAKE) -sj8 && $(MAKE) install

$(TIFF_SRC)/%/Makefile : $(libtiffconfig)
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-Qunused-arguments -arch $(call swap, $*, $(arch_names_all), $(archs_all)) -pipe -no-cpp-precomp -isysroot $$SDKROOT -m$(PLATFORM_VERSION_MIN) -O2 -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="$$CFLAGS -Wno-deprecated-register"; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
	../configure --host=$* --enable-fast-install --enable-shared=no --prefix=`pwd` --without-x --with-jpeg-include-dir=$(abspath $(@D)/../../$(JPEG_DIR_NAME)/$*/include) --with-jpeg-lib-dir=$(abspath $(@D)/../../$(JPEG_DIR_NAME)/$*/lib)

libpng : $(libpngfat)

$(libpngfat) : $(libpng)
	mkdir -p $(@D)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libpngfolders_all)) ) -create -output $@
	mkdir -p $(IMAGE_INC_DIR)
	cp -rvf $(firstword $(libpngfolders))/include/*.h $(IMAGE_INC_DIR)

$(libpng) : $(libpngmakefile)
	cd $(abspath $(@D)/..) ; \
	$(MAKE) -sj8 && $(MAKE) install

$(PNG_SRC)/%/Makefile : $(libpngconfig)
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-Qunused-arguments -arch $(call swap, $*, $(arch_names_all), $(archs_all)) -pipe -no-cpp-precomp -isysroot $$SDKROOT -m$(PLATFORM_VERSION_MIN) -O2 -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="$$CFLAGS -Wno-deprecated-register"; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
	../configure --host=$* --enable-shared=no --prefix=`pwd`

libjpeg : $(libjpegfat)

$(libjpegfat) : $(libjpeg)
	mkdir -p $(@D)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libjpegfolders_all)) ) -create -output $@
	mkdir -p $(IMAGE_INC_DIR)
	cp -rvf $(firstword $(libjpegfolders))/include/*.h $(IMAGE_INC_DIR)

$(libjpeg) : $(libjpegmakefile)
	cd $(abspath $(@D)/..) ; \
	$(MAKE) -sj8 && $(MAKE) install

$(JPEG_SRC)/%/Makefile : $(libjpegconfig)
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-Qunused-arguments -arch $(call swap, $*, $(arch_names_all), $(archs_all)) -pipe -no-cpp-precomp -isysroot $$SDKROOT -m$(PLATFORM_VERSION_MIN) -O2 -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="$$CFLAGS -Wno-deprecated-register"; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
	../configure --host=$* --enable-shared=no --prefix=`pwd`

#######################
# Download sources
#######################
$(libtiffconfig) :
	curl http://download.osgeo.org/libtiff/$(TIFF_NAME).tar.gz | tar -xpf-

$(libjpegconfig) :
	curl http://www.ijg.org/files/$(JPEG_SRC_NAME).tar.gz | tar -xpf-

$(libpngconfig) :
	curl ftp://ftp.simplesystems.org/pub/libpng/png/src/libpng16/$(PNG_NAME).tar.gz | tar -xpf-

#######################
# Clean
#######################
.PHONY : clean
clean : cleanpng cleantiff cleanjpeg

.PHONY : cleanpng
cleanpng :
	for folder in $(realpath $(libpngfolders_all) ); do \
		cd $$folder; \
		$(MAKE) clean; \
	done

.PHONY : cleanjpeg
cleanjpeg :
	for folder in $(realpath $(libjpegfolders_all) ); do \
		cd $$folder; \
		$(MAKE) clean; \
	done

.PHONY : cleantiff
cleantiff :
	for folder in $(realpath $(libtifffolders_all) ); do \
		cd $$folder; \
		$(MAKE) clean; \
	done

.PHONY : mostlyclean
mostlyclean : mostlycleanpng mostlycleantiff mostlycleanjpeg

.PHONY : mostlycleanpng
mostlycleanpng :
	for folder in $(realpath $(libpngfolders) ); do \
		cd $$folder; \
		$(MAKE) mostlyclean; \
	done

.PHONY : mostlycleantiff
mostlycleantiff :
	for folder in $(realpath $(libtifffolders_all) ); do \
		cd $$folder; \
		$(MAKE) mostlyclean; \
	done

.PHONY : mostlycleanjpeg
mostlycleanjpeg :
	for folder in $(realpath $(libjpegfolders_all) ); do \
		cd $$folder; \
		$(MAKE) mostlyclean; \
	done

.PHONY : distclean
distclean :
	-rm -rf $(IMAGE_LIB_DIR)
	-rm -rf $(IMAGE_INC_DIR)
	-rm -rf $(PNG_SRC)
	-rm -rf $(JPEG_SRC)
	-rm -rf $(TIFF_SRC)
