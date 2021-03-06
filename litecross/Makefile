
OUTPUT = $(CURDIR)/output

BINUTILS_SRCDIR = BINUTILS_SRCDIR_not_set
GCC_SRCDIR = GCC_SRCDIR_not_set

GCC_CONFIG_FOR_TARGET = 
COMMON_CONFIG = 
GCC_CONFIG = 
MINGW_CRT_CONFIG =
TOOLCHAIN_CONFIG = 

XGCC_DIR = $(CURDIR)/obj_gcc/gcc
XGCC = $(XGCC_DIR)/xgcc -B $(XGCC_DIR) -B $(CURDIR)/obj_sysroot/$(TARGET)/lib
XCPP = $(XGCC_DIR)/cpp -B $(XGCC_DIR)
XAR  = $(CURDIR)/obj_binutils/binutils/ar
XAS  = $(CURDIR)/obj_binutils/gas/as-new
XRANLIB  = $(CURDIR)/obj_binutils/binutils/ranlib
XRC  = $(CURDIR)/obj_binutils/binutils/windres --preprocessor=$(XGCC_DIR)/xgcc --preprocessor-arg=-B --preprocessor-arg=$(XGCC_DIR) --preprocessor-arg=-I$(CURDIR)/obj_sysroot/$(TARGET)/include --preprocessor-arg=-E --preprocessor-arg=-xc-header --preprocessor-arg=-DRC_INVOKED -c 1252
DLLTOOL = $(CURDIR)/obj_binutils/binutils/dlltool

-include config.mak

ifneq ($(findstring mingw,$(TARGET)),)
MUSL_SRCDIR =
LINUX_SRCDIR =
endif

ifneq ($(findstring musl,$(TARGET)),)
MINGW_SRCDIR =
endif

ifneq ($(findstring fdpic,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --enable-fdpic
endif

ifneq ($(filter x86_64%x32,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-abi=x32
endif

ifneq ($(findstring powerpc64,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-abi=elfv2
endif

ifneq ($(findstring mips64,$(TARGET))$(findstring mipsisa64,$(TARGET)),)
ifneq ($(findstring n32,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-abi=n32
else
GCC_CONFIG_FOR_TARGET += --with-abi=64
endif
endif

ifneq ($(findstring s390x,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-long-double-128
endif

ifneq ($(filter %sf,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-float=soft
endif

ifneq ($(filter %hf,$(TARGET)),)
GCC_CONFIG_FOR_TARGET += --with-float=hard
endif

ifneq ($(LIBTOOL),)

ifeq  ($(LIBTOOL_STATIC),)
LIBTOOL_STATIC = $(LIBTOOL)-static
endif

LIBTOOL_ARG        = LIBTOOL=$(LIBTOOL)
LIBTOOL_STATIC_ARG = LIBTOOL=$(LIBTOOL_STATIC)

endif

MAKE += MULTILIB_OSDIRNAMES=
MAKE += INFO_DEPS= infodir=
MAKE += ac_cv_prog_lex_root=lex.yy.c
MAKE += MAKEINFO=false

FULL_BINUTILS_CONFIG = \
	$(COMMON_CONFIG) $(BINUTILS_CONFIG) $(TOOLCHAIN_CONFIG) \
	--disable-werror \
	--target=$(TARGET) --prefix= \
	--libdir=/lib --disable-multilib \
	--with-sysroot=$(SYSROOT)

FULL_GCC_CONFIG = --enable-languages=c,c++ \
	$(GCC_CONFIG_FOR_TARGET) \
	$(COMMON_CONFIG) $(GCC_CONFIG) $(TOOLCHAIN_CONFIG) \
	--disable-werror \
	--target=$(TARGET) --prefix= \
	--libdir=/lib --disable-multilib \
	--with-sysroot=$(SYSROOT) \
	--enable-tls \
	--disable-libmudflap --disable-libsanitizer \
	--disable-gnu-indirect-function \
	--disable-libmpx \
	--enable-deterministic-archives \
	--enable-libstdcxx-time

FULL_MUSL_CONFIG = $(MUSL_CONFIG) \
	--prefix= --host=$(TARGET)

FULL_MINGW_HEADERS_CONFIG = $(MINGW_CONFIG) \
	--prefix= --host=$(TARGET) --enable-sdk=all \
	--enable-idl --enable-secure-api --with-sysroot=$(SYSROOT)

FULL_MINGW_CRT_CONFIG = $(MINGW_CONFIG) $(MINGW_CRT_CONFIG) \
	--prefix= --host=$(TARGET) --with-sysroot=$(SYSROOT)

FULL_MINGW_PTHREADS_CONFIG = $(MINGW_CONFIG) \
	--prefix= --host=$(TARGET) --with-sysroot=$(SYSROOT)

ifneq ($(NATIVE),)
HOST:=$(TARGET)
endif

ifeq ($(HOST),)
SYSROOT = /$(TARGET)
FULL_GCC_CONFIG += --with-build-sysroot=$(CURDIR)/obj_sysroot \
	AR_FOR_TARGET=$(PWD)/obj_binutils/binutils/ar \
	AS_FOR_TARGET=$(PWD)/obj_binutils/gas/as-new \
	LD_FOR_TARGET=$(PWD)/obj_binutils/ld/ld-new \
	NM_FOR_TARGET=$(PWD)/obj_binutils/binutils/nm-new \
	OBJCOPY_FOR_TARGET=$(PWD)/obj_binutils/binutils/objcopy \
	OBJDUMP_FOR_TARGET=$(PWD)/obj_binutils/binutils/objdump \
	RANLIB_FOR_TARGET=$(PWD)/obj_binutils/binutils/ranlib \
	READELF_FOR_TARGET=$(PWD)/obj_binutils/binutils/readelf \
	STRIP_FOR_TARGET=$(PWD)/obj_binutils/binutils/strip-new \
	# end
FULL_MUSL_CONFIG += CC="$(XGCC)" LIBCC="../obj_gcc/$(TARGET)/libgcc/libgcc.a" 
MUSL_VARS = AR=../obj_binutils/binutils/ar RANLIB=../obj_binutils/binutils/ranlib
MINGW_VARS = AS="$(XAS)" AR="$(XAR)" CC="$(XGCC)" RANLIB="$(XRANLIB)"
FULL_MINGW_PTHREADS_CONFIG += RC="$(XRC)" CC="$(XGCC)" CPP="$(XCPP)" CPPFLAGS="-I$(CURDIR)/obj_sysroot/$(TARGET)/include $(CPPFLAGS)" LDFLAGS="-L$(CURDIR)/obj_sysroot/$(TARGET)/lib $(LDFLAGS)" DLLTOOL=$(DLLTOOL)
FULL_MINGW_CRT_CONFIG += CC="$(XGCC)" CPP="$(XCPP)" CPPFLAGS="-I$(CURDIR)/obj_sysroot/$(TARGET)/include $(CPPFLAGS)" DLLTOOL="$(DLLTOOL)" RC="$(XRC)"
obj_mingw_headers/.lc_configured: | obj_binutils/.lc_built
obj_mingw_crt/.lc_configured: | obj_sysroot/.lc_headers obj_gcc/gcc/.lc_built obj_binutils/.lc_built
obj_mingw_pthreads/.lc_configured: | obj_sysroot/.lc_headers obj_gcc/gcc/.lc_built obj_binutils/.lc_built obj_sysroot/.lc_libs_crt
obj_musl/.lc_configured: | obj_gcc/gcc/.lc_built
obj_musl/.lc_built: | obj_gcc/$(TARGET)/libgcc/libgcc.a
obj_gcc/gcc/.lc_built: | obj_sysroot/usr obj_sysroot/lib32 obj_sysroot/lib64 obj_sysroot/include
ifneq ($(MUSL_SRCDIR),)
obj_gcc/.lc_built: | obj_sysroot/.lc_libs obj_sysroot/.lc_headers
endif
ifneq ($(MINGW_SRCDIR),)
obj_gcc/.lc_built: | obj_sysroot/.lc_libs_crt obj_sysroot/.lc_headers
endif
obj_gcc/.lc_configured: obj_binutils/.lc_built
else
SYSROOT = /
FULL_BINUTILS_CONFIG += --host=$(HOST)
FULL_GCC_CONFIG += --host=$(HOST)
MUSL_VARS = 
endif

ifeq ($(TARGET),)

all:
	@echo TARGET must be set.
	@exit 1

install: all

else

ifneq ($(MUSL_SRCDIR),)
all: musl gcc binutils
endif

ifneq ($(MINGW_SRCDIR),)
all: mingw gcc binutils
endif

ifneq ($(MUSL_SRCDIR),)
install: install-musl install-gcc install-binutils
endif

ifneq ($(MINGW_SRCDIR),)
install: install-mingw install-gcc install-binutils
endif

musl: obj_musl/.lc_built

mingw: obj_mingw_headers/.lc_built obj_mingw_crt/.lc_built obj_mingw_pthreads/.lc_built

toolchain: gcc binutils

install-toolchain: install-gcc install-binutils

gcc: obj_gcc/.lc_built

binutils: obj_binutils/.lc_built

.PHONY: all musl mingw toolchain install-musl install-mingw install-toolchain clean

src_binutils: | $(BINUTILS_SRCDIR)
	ln -sf $(BINUTILS_SRCDIR) $@

src_gcc_base: | $(GCC_SRCDIR)
	ln -sf $(GCC_SRCDIR) $@

ifneq ($(MUSL_SRCDIR),)
src_musl: | $(MUSL_SRCDIR)
	ln -sf $(MUSL_SRCDIR) $@
endif

ifneq ($(MINGW_SRCDIR),)
src_mingw: | $(MINGW_SRCDIR)
	ln -sf $(MINGW_SRCDIR) $@
endif

ifneq ($(GMP_SRCDIR),)
src_gcc: src_gmp
src_gmp: | $(GMP_SRCDIR)
	ln -sf "$(GMP_SRCDIR)" $@
endif

ifneq ($(MPC_SRCDIR),)
src_gcc: src_mpc
src_mpc: | $(MPC_SRCDIR)
	ln -sf "$(MPC_SRCDIR)" $@
endif

ifneq ($(MPFR_SRCDIR),)
src_gcc: src_mpfr
src_mpfr: | $(MPFR_SRCDIR)
	ln -sf "$(MPFR_SRCDIR)" $@
endif

ifneq ($(ISL_SRCDIR),)
src_gcc: src_isl
src_isl: | $(ISL_SRCDIR)
	ln -sf "$(ISL_SRCDIR)" $@
endif

src_gcc: src_gcc_base
	rm -rf $@ $@.tmp
	mkdir $@.tmp
	cd $@.tmp && ln -sf ../src_gcc_base/* .
	$(if $(GMP_SRCDIR),cd $@.tmp && ln -sf ../src_gmp gmp)
	$(if $(MPC_SRCDIR),cd $@.tmp && ln -sf ../src_mpc mpc)
	$(if $(MPFR_SRCDIR),cd $@.tmp && ln -sf ../src_mpfr mpfr)
	$(if $(ISL_SRCDIR),cd $@.tmp && ln -sf ../src_isl isl)
	mv $@.tmp $@

obj_%:
	mkdir -p $@

obj_sysroot/include:
	mkdir -p $@

obj_sysroot/usr: | obj_sysroot
	ln -sf . $@

obj_sysroot/lib32: | obj_sysroot
	ln -sf lib $@

obj_sysroot/lib64: | obj_sysroot
	ln -sf lib $@

obj_binutils/.lc_configured: | obj_binutils src_binutils
	cd obj_binutils && ../src_binutils/configure $(FULL_BINUTILS_CONFIG)
	touch $@

obj_binutils/.lc_built: | obj_binutils/.lc_configured
	cd obj_binutils && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_STATIC_ARG)" all
	touch $@

obj_gcc/.lc_configured: | obj_gcc src_gcc
	cd obj_gcc && ../src_gcc/configure $(FULL_GCC_CONFIG)
	touch $@

obj_gcc/.lc_gmp_built: | obj_gcc/.lc_configured
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_STATIC_ARG)" all-gmp
	touch $@

obj_gcc/.lc_mpfr_built: | obj_gcc/.lc_gmp_built
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_STATIC_ARG)" all-mpfr
	touch $@

obj_gcc/.lc_mpc_built: | obj_gcc/.lc_mpfr_built
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_STATIC_ARG)" all-mpc
	touch $@

ifneq ($(MUSL_SRCDIR),)
obj_gcc/gcc/.lc_built: | obj_gcc/.lc_mpc_built
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_ARG)" all-gcc
	touch $@
endif

ifneq ($(MINGW_SRCDIR),)
obj_gcc/gcc/.lc_built: | obj_gcc/.lc_mpc_built obj_sysroot/.lc_headers
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_ARG)" all-gcc
	touch $@
endif

ifneq ($(MUSL_SRCDIR),)
obj_musl/.lc_configured: | obj_musl src_musl
	cd obj_musl && ../src_musl/configure $(FULL_MUSL_CONFIG)
	touch $@
endif

ifneq ($(MINGW_SRCDIR),)
obj_mingw/.lc_configured: | obj_mingw src_mingw obj_mingw_headers/.lc_configured obj_mingw_crt/.lc_configured
	touch $@
endif

ifneq ($(MUSL_SRCDIR),)
obj_sysroot/.lc_headers: | obj_musl/.lc_configured obj_sysroot
	cd obj_musl && $(MAKE) DESTDIR=$(CURDIR)/obj_sysroot install-headers
	touch $@
endif

ifneq ($(MINGW_SRCDIR),)
obj_sysroot/.lc_headers: | obj_mingw_headers/.lc_configured obj_sysroot
	cd obj_mingw_headers && $(MAKE) DESTDIR=$(CURDIR)/obj_sysroot/$(TARGET) install
	rm -f $(CURDIR)/obj_sysroot/mingw
	ln -s $(TARGET) $(CURDIR)/obj_sysroot/mingw
	touch $@
endif

obj_gcc/$(TARGET)/libgcc/libgcc.a: | obj_sysroot/.lc_headers
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) enable_shared=no $(LIBTOOL_ARG)" all-target-libgcc

obj_musl/.lc_built: | obj_musl/.lc_configured
	cd obj_musl && $(MAKE) $(MUSL_VARS)
	touch $@

ifneq ($(MUSL_SRCDIR),)
obj_sysroot/.lc_libs: | obj_musl/.lc_built
	cd obj_musl && $(MAKE) $(MUSL_VARS) DESTDIR=$(CURDIR)/obj_sysroot install
	touch $@
endif

ifneq ($(MINGW_SRCDIR),)
obj_sysroot/.lc_libs: | obj_sysroot/.lc_libs_pthreads obj_sysroot/.lc_libs_crt
	touch $@
endif

obj_gcc/.lc_built: | obj_gcc/.lc_configured obj_gcc/gcc/.lc_built
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_ARG)"
	touch $@

install-musl: | obj_musl/.lc_built
	cd obj_musl && $(MAKE) $(MUSL_VARS) DESTDIR=$(DESTDIR)$(OUTPUT)$(SYSROOT) install

install-binutils: | obj_binutils/.lc_built
	cd obj_binutils && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_STATIC_ARG)" DESTDIR=$(DESTDIR)$(OUTPUT) install

install-gcc: | obj_gcc/.lc_built
	cd obj_gcc && $(MAKE) MAKE="$(MAKE) $(LIBTOOL_ARG)" DESTDIR=$(DESTDIR)$(OUTPUT) install
	ln -sf $(TARGET)-gcc $(DESTDIR)$(OUTPUT)/bin/$(TARGET)-cc

obj_mingw_headers/.lc_configured: | obj_mingw_headers src_mingw
	cd obj_mingw_headers && ../src_mingw/mingw-w64-headers/configure $(FULL_MINGW_HEADERS_CONFIG)
	touch $@

obj_mingw_headers/.lc_built: | obj_mingw_headers/.lc_configured
	cd obj_mingw_headers && $(MAKE) $(MINGW_VARS)
	touch $@

obj_mingw_crt/.lc_configured: | obj_sysroot/.lc_headers obj_binutils/.lc_built obj_gcc/gcc/.lc_built obj_mingw_crt src_mingw
	cd obj_mingw_crt && ../src_mingw/mingw-w64-crt/configure $(FULL_MINGW_CRT_CONFIG)
	touch $@

obj_mingw_crt/.lc_built: | obj_mingw_crt/.lc_configured
	cd obj_mingw_crt && $(MAKE) $(MINGW_VARS)
	touch $@

obj_mingw_pthreads/.lc_configured: | obj_mingw_crt/.lc_built obj_mingw_pthreads src_mingw
	cd obj_mingw_pthreads && ../src_mingw/mingw-w64-libraries/winpthreads/configure $(FULL_MINGW_PTHREADS_CONFIG)
	touch $@

obj_mingw_pthreads/.lc_built: | obj_mingw_pthreads/.lc_configured
	cd obj_mingw_pthreads && $(MAKE) $(MINGW_VARS)
	touch $@

obj_sysroot/.lc_libs_pthreads: | obj_mingw_pthreads/.lc_built
	cd obj_mingw_pthreads && $(MAKE) $(MINGW_VARS) DESTDIR=$(CURDIR)/obj_sysroot/$(TARGET) install
	touch $@

obj_sysroot/.lc_libs_crt: | obj_mingw_crt/.lc_built
	cd obj_mingw_crt && $(MAKE) $(MINGW_VARS) DESTDIR=$(CURDIR)/obj_sysroot/$(TARGET) install
	touch $@

install-mingw-headers: | obj_mingw_headers/.lc_configured
	cd obj_mingw_headers && $(MAKE) $(MINGW_VARS) DESTDIR=$(DESTDIR)$(OUTPUT)$(SYSROOT) install

install-mingw-crt: | obj_mingw_crt/.lc_built
	cd obj_mingw_crt && $(MAKE) $(MINGW_VARS) DESTDIR=$(DESTDIR)$(OUTPUT)$(SYSROOT) install

install-mingw-pthreads: | obj_mingw_pthreads/.lc_built
	cd obj_mingw_pthreads && $(MAKE) $(MINGW_VARS) DESTDIR=$(DESTDIR)$(OUTPUT)$(SYSROOT) install

install-mingw: install-mingw-headers install-mingw-crt install-mingw-pthreads

ifneq ($(LINUX_SRCDIR),)
TARGET_ARCH = $(firstword $(subst -, ,$(TARGET)))
TARGET_ARCH_MANGLED = $(patsubst i%86,x86,$(patsubst aarch64%,arm64%,$(TARGET_ARCH)))
LINUX_ARCH_LIST = $(sort $(notdir $(wildcard $(LINUX_SRCDIR)/arch/*)))
LINUX_ARCH = $(lastword $(foreach a,$(LINUX_ARCH_LIST),$(if $(filter $(a)%,$(TARGET_ARCH_MANGLED)),$(a))))
ifneq ($(LINUX_ARCH),)
all: kernel-headers
install: install-kernel-headers
kernel-headers: | obj_kernel_headers/.lc_built
src_kernel_headers: | $(LINUX_SRCDIR)
	ln -sf "$(LINUX_SRCDIR)" $@
obj_kernel_headers/.lc_built: | src_kernel_headers
	mkdir -p $(CURDIR)/obj_kernel_headers/staged
	cd src_kernel_headers && $(MAKE) ARCH=$(LINUX_ARCH) O=$(CURDIR)/obj_kernel_headers INSTALL_HDR_PATH=$(CURDIR)/obj_kernel_headers/staged headers_install
	find obj_kernel_headers/staged/include '(' -name .install -o -name ..install.cmd ')' -exec rm {} +
	touch $@
install-kernel-headers: | obj_kernel_headers/.lc_built
	mkdir -p $(DESTDIR)$(OUTPUT)$(SYSROOT)/include
	cp -R obj_kernel_headers/staged/include/* $(DESTDIR)$(OUTPUT)$(SYSROOT)/include
endif
endif

endif

clean:
	rm -rf src_* obj_*
