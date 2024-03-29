##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_define([v_maj], [3])
m4_define([v_min], [3])
m4_define([v_mic], [1])
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_define([v_ver], [v_maj.v_min.v_mic]-yac2)
m4_define([lt_rev], m4_eval(v_maj + v_min))
m4_define([lt_cur], v_mic)
m4_define([lt_age], v_min)
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##

AC_INIT([zccminer], [v_ver], [mikaelh@mbnet.fi])

AC_PREREQ(2.59)
AC_CANONICAL_SYSTEM
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([zccminer.c])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([foreign])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])
AC_USE_SYSTEM_EXTENSIONS

##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
m4_ifdef([v_rev], , [m4_define([v_rev], [0])])
m4_ifdef([v_rel], , [m4_define([v_rel], [])])
AC_DEFINE_UNQUOTED(YACMINER_MAJOR_VERSION, [v_maj], [Major version])
AC_DEFINE_UNQUOTED(YACMINER_MINOR_VERSION, [v_min], [Minor version])
AC_DEFINE_UNQUOTED(YACMINER_MINOR_SUBVERSION, [v_mic], [Micro version])
version_info="lt_rev:lt_cur:lt_age"
release_info="v_rel"
AC_SUBST(version_info)
AC_SUBST(release_info)
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##--##
VMAJ=v_maj
AC_SUBST(VMAJ)

AC_CANONICAL_BUILD
AC_CANONICAL_HOST

dnl Make sure anyone changing configure.ac/Makefile.am has a clue
AM_MAINTAINER_MODE

dnl Checks for programs
AC_PROG_CC
gl_EARLY
AC_PROG_GCC_TRADITIONAL
AM_PROG_CC_C_O
AC_PROG_RANLIB

gl_INIT

dnl Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS(syslog.h)

AC_FUNC_ALLOCA

have_win32=false
PTHREAD_FLAGS="-lpthread"
DLOPEN_FLAGS="-ldl"
OPENCL_LIBS="-lOpenCL"
WS2_LIBS=""
MM_LIBS=""
MATH_LIBS="-lm"

case $target in
  amd64-*)
    have_x86_64=true
    ;;
  x86_64-*)
    have_x86_64=true
    ;;
  *)
    have_x86_64=false
    ;;
esac

case $target in
  *-*-mingw*)
    have_win32=true
    PTHREAD_FLAGS=""
    DLOPEN_FLAGS=""
    WS2_LIBS="-lws2_32"
    MM_LIBS="-lwinmm"
    AC_DEFINE([_WIN32_WINNT], [0x0501], "WinNT version for XP+ support")
    ;;
  powerpc-*-darwin*)
    CFLAGS="$CFLAGS -faltivec"
    OPENCL_LIBS=""
    PTHREAD_FLAGS=""
    ;;
  *-*-darwin*)
    OPENCL_LIBS="-framework OpenCL"
    PTHREAD_FLAGS=""
	;;
esac


if test "x$have_win32" != xtrue; then
	if test "x$have_x86_64" = xtrue; then
		ARCH_DIR=x86_64
	else
		ARCH_DIR=x86
	fi

	if test "x$ATISTREAMSDKROOT" != x; then
		OPENCL_FLAGS="-I$ATISTREAMSDKROOT/include $OPENCL_FLAGS"
		OPENCL_LIBS="-L$ATISTREAMSDKROOT/lib/$ARCH_DIR $OPENCL_LIBS"
	fi

	if test "x$AMDAPPSDKROOT" != x; then
		OPENCL_FLAGS="-I$AMDAPPSDKROOT/include $OPENCL_FLAGS"
		OPENCL_LIBS="-L$AMDAPPSDKROOT/lib/$ARCH_DIR $OPENCL_LIBS"
	fi
fi

have_zccminer_sdk=false
if test -n "$YACMINER_SDK"; then
	have_zccminer_sdk=true
	CPPFLAGS="-I$YACMINER_SDK/include $CPPFLAGS"
	LDFLAGS="-L$YACMINER_SDK/lib/$target $LDFLAGS"
fi

opencl="yes"

AC_ARG_ENABLE([opencl],
	[AC_HELP_STRING([--disable-opencl],[Override detection and disable building with opencl])],
	[opencl=$enableval]
	)
if test "x$opencl" != xno; then
	# Check for OpenCL (the long way needed on mingw32 due to calling conventions)
	AC_MSG_CHECKING([for OpenCL])
	SAVED_LIBS=$LIBS
	SAVED_CFLAGS=$CFLAGS
	LIBS="$LIBS $OPENCL_LIBS"
	CFLAGS="$CFLAGS $OPENCL_FLAGS"
	AC_LINK_IFELSE(
	[AC_LANG_PROGRAM([[
		#ifdef __APPLE_CC__
		#include <OpenCL/opencl.h>
		#else
		#include <CL/cl.h>
		#endif
	]],
	[[return clSetKernelArg(0, 0, 0, 0); ]])],
	[AC_MSG_RESULT(yes)
	AC_DEFINE([HAVE_OPENCL], [1], [Defined to 1 if OpenCL is present on the system.])
	found_opencl=1
	],
	[AC_MSG_RESULT(no)
	OPENCL_FLAGS=
	OPENCL_LIBS=
	found_opencl=0])
	LIBS=$SAVED_LIBS
	CFLAGS=$SAVED_CFLAGS
else
	OPENCL_FLAGS=""
	OPENCL_LIBS=""
fi

has_winpthread=false
if test "x$have_win32" = xtrue; then
        has_winpthread=true
        AC_CHECK_LIB(winpthread, nanosleep, , has_winpthread=false)
        PTHREAD_LIBS=-lwinpthread
fi

if test "x$has_winpthread" != xtrue; then
        AC_CHECK_LIB(pthread, pthread_create, ,
                AC_MSG_ERROR([Could not find pthread library - please install libpthread]))
        PTHREAD_LIBS=-lpthread
fi

AC_CHECK_LIB(jansson, json_loads, request_jansson=false, request_jansson=true)

AC_ARG_ENABLE([adl],
	[AC_HELP_STRING([--disable-adl],[Override detection and disable building with adl])],
	[adl=$enableval]
	)

scrypt="no"

if test "$found_opencl" = 1; then
	if test "x$adl" != xno; then
		ADL_CPPFLAGS=
		AC_CHECK_FILE([$srcdir/ADL_SDK/adl_sdk.h], [have_adl=true; ADL_CPPFLAGS=-I$srcdir], have_adl=false,)
		if test x$have_adl+$have_zccminer_sdk = xfalse+true; then
			AC_CHECK_FILE([$YACMINER_SDK/include/ADL_SDK/adl_sdk.h], [have_adl=true; ADL_CPPFLAGS=-I$YACMINER_SDK/include], have_adl=false,)
		fi
		if test x$have_adl = xtrue
		then
			AC_DEFINE([HAVE_ADL], [1], [Defined if ADL headers were found])
		else
			DLOPEN_FLAGS=""
		fi
	fi

	AC_ARG_ENABLE([scrypt],
		[AC_HELP_STRING([--enable-scrypt],[Compile support for scrypt litecoin mining (default disabled)])],
		[scrypt=$enableval]
		)
	if test "x$scrypt" = xyes; then
		AC_DEFINE([USE_SCRYPT], [1], [Defined to 1 if scrypt support is wanted])
	fi
else
	DLOPEN_FLAGS=""
fi

AM_CONDITIONAL([HAS_SCRYPT], [test x$scrypt = xyes])

bflsc="no"

AC_ARG_ENABLE([bflsc],
	[AC_HELP_STRING([--enable-bflsc],[Compile support for BFL ASICs (default disabled)])],
	[bflsc=$enableval]
	)
if test "x$bflsc" = xyes; then
	AC_DEFINE([USE_BFLSC], [1], [Defined to 1 if BFL ASIC support is wanted])
fi
AM_CONDITIONAL([HAS_BFLSC], [test x$bflsc = xyes])

bitforce="no"

AC_ARG_ENABLE([bitforce],
	[AC_HELP_STRING([--enable-bitforce],[Compile support for BitForce FPGAs (default disabled)])],
	[bitforce=$enableval]
	)
if test "x$bitforce" = xyes; then
	AC_DEFINE([USE_BITFORCE], [1], [Defined to 1 if BitForce support is wanted])
fi
AM_CONDITIONAL([HAS_BITFORCE], [test x$bitforce = xyes])

icarus="no"

AC_ARG_ENABLE([icarus],
	[AC_HELP_STRING([--enable-icarus],[Compile support for Icarus (default disabled)])],
	[icarus=$enableval]
	)
if test "x$icarus" = xyes; then
	AC_DEFINE([USE_ICARUS], [1], [Defined to 1 if Icarus support is wanted])
fi
AM_CONDITIONAL([HAS_ICARUS], [test x$icarus = xyes])

avalon="no"

AC_ARG_ENABLE([avalon],
	[AC_HELP_STRING([--enable-avalon],[Compile support for Avalon (default disabled)])],
	[avalon=$enableval]
	)
if test "x$avalon" = xyes; then
	AC_DEFINE([USE_AVALON], [1], [Defined to 1 if Avalon support is wanted])
fi
AM_CONDITIONAL([HAS_AVALON], [test x$avalon = xyes])

modminer="no"

AC_ARG_ENABLE([modminer],
	[AC_HELP_STRING([--enable-modminer],[Compile support for ModMiner FPGAs(default disabled)])],
	[modminer=$enableval]
	)
if test "x$modminer" = xyes; then
	AC_DEFINE([USE_MODMINER], [1], [Defined to 1 if ModMiner support is wanted])
fi
AM_CONDITIONAL([HAS_MODMINER], [test x$modminer = xyes])

ztex="no"

AC_ARG_ENABLE([ztex],
	[AC_HELP_STRING([--enable-ztex],[Compile support for Ztex (default disabled)])],
	[ztex=$enableval]
	)
if test "x$ztex" = xyes; then
	AC_DEFINE([USE_ZTEX], [1], [Defined to 1 if Ztex support is wanted])
fi
AM_CONDITIONAL([HAS_ZTEX], [test x$ztex = xyes])

curses="auto"

AC_ARG_WITH([curses],
	[AC_HELP_STRING([--without-curses],[Compile support for curses TUI (default enabled)])],
	[curses=$withval]
	)
if test "x$curses" = "xno"; then
	cursesmsg='User specified --without-curses. TUI support DISABLED'
else
	AC_SEARCH_LIBS(addstr, ncurses pdcurses, [
		curses=yes
		cursesmsg="FOUND: ${ac_cv_search_addstr}"
		AC_DEFINE([HAVE_CURSES], [1], [Defined to 1 if curses TUI support is wanted])
	], [
		if test "x$curses" = "xyes"; then
			AC_MSG_ERROR([Could not find curses library - please install libncurses-dev or pdcurses-dev (or configure --without-curses)])
		else
			AC_MSG_WARN([Could not find curses library - if you want a TUI, install libncurses-dev or pdcurses-dev])
			curses=no
			cursesmsg='NOT FOUND. TUI support DISABLED'
		fi
	])
fi

AM_CONDITIONAL([NEED_FPGAUTILS], [test x$modminer$ztex != xnono])
AM_CONDITIONAL([NEED_USBUTILS_C], [test x$avalon$bitforce$modminer$bflsc$icarus != xnonononono])
AM_CONDITIONAL([HAVE_CURSES], [test x$curses = xyes])
AM_CONDITIONAL([WANT_JANSSON], [test x$request_jansson = xtrue])
AM_CONDITIONAL([HAVE_WINDOWS], [test x$have_win32 = xtrue])
AM_CONDITIONAL([HAVE_x86_64], [test x$have_x86_64 = xtrue])

if test "x$avalon$bitforce$modminer$bflsc$icarus" != xnonononono; then
	AC_DEFINE([USE_USBUTILS], [1], [Defined to 1 if usbutils support required])
fi

if test x$request_jansson = xtrue
then
	JANSSON_LIBS="compat/jansson/libjansson.a"
else
	JANSSON_LIBS=-ljansson
fi

PKG_PROG_PKG_CONFIG()

if test "x$have_zccminer_sdk" = "xtrue"; then
	if test "x$have_x86_64" = xtrue; then
		ARCH_DIR=x86_64
	else
		ARCH_DIR=x86
	fi
	PKG_CONFIG="${PKG_CONFIG:-pkg-config} --define-variable=arch=$ARCH_DIR --define-variable=target=$target --define-variable=zccminersdkdir=$YACMINER_SDK"
	PKG_CONFIG_PATH="$YACMINER_SDK/lib/pkgconfig${PKG_CONFIG_PATH:+:$PKG_CONFIG_PATH}"
fi

if test "x$avalon$ztex$modminer$bitforce$bflsc$icarus" != xnononononono; then
  case $target in
    *-*-freebsd*)
      LIBUSB_LIBS="-lusb"
      LIBUSB_CFLAGS=""
      AC_DEFINE(HAVE_LIBUSB, 1, [Define if you have libusb-1.0])
      ;;
    *)
      PKG_CHECK_MODULES(LIBUSB, libusb-1.0, [AC_DEFINE(HAVE_LIBUSB, 1, [Define if you have libusb-1.0])], [AC_MSG_ERROR([Could not find usb library - please install libusb-1.0])])
      ;;
  esac
fi
AC_SUBST(LIBUSB_LIBS)
AC_SUBST(LIBUSB_CFLAGS)

if test "x$have_win32" != xtrue; then
	PKG_CHECK_MODULES([LIBCURL], [libcurl >= 7.25.0], [AC_DEFINE([CURL_HAS_KEEPALIVE], [1], [Defined if version of curl supports keepalive.])],
		[PKG_CHECK_MODULES([LIBCURL], [libcurl >= 7.18.2], ,[AC_MSG_ERROR([Missing required libcurl dev >= 7.18.2])])])
else
	PKG_CHECK_MODULES([LIBCURL], [libcurl >= 7.25.0], ,[AC_MSG_ERROR([Missing required libcurl dev >= 7.25.0])])
	AC_DEFINE([CURL_HAS_KEEPALIVE], [1])
fi
AC_SUBST(LIBCURL_LIBS)

#check execv signature
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
		   #include <process.h>
		   int execv(const char*, const char*const*);
		   ])],
		   AC_DEFINE([EXECV_2ND_ARG_TYPE], [const char* const*], [int execv(const char*, const char*const*);]),
		   AC_DEFINE([EXECV_2ND_ARG_TYPE], [char* const*], [int execv(const char*, char*const*);]))

dnl CCAN wants to know a lot of vars.
# All the configuration checks.  Regrettably, the __attribute__ checks will
# give false positives on old GCCs, since they just cause warnings.  But that's
# fairly harmless.
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((cold)) cleanup(void) { }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_COLD], [1],
                                    [Define if __attribute__((cold))]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((const)) cleanup(void) { }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_CONST], [1],
                                    [Define if __attribute__((const))]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((noreturn)) cleanup(void) { exit(1); }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_NORETURN], [1],
                                    [Define if __attribute__((noreturn))]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((format(__printf__, 1, 2))) cleanup(const char *fmt, ...) { }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_PRINTF], [1],
                                    [Define if __attribute__((format(__printf__)))]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((unused)) cleanup(void) { }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_UNUSED], [1],
                                    [Define if __attribute__((unused))]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static void __attribute__((used)) cleanup(void) { }])],
			  AC_DEFINE([HAVE_ATTRIBUTE_USED], [1],
                                    [Define if __attribute__((used))]))
AC_LINK_IFELSE([AC_LANG_SOURCE([int main(void) { return __builtin_constant_p(1) ? 0 : 1; }])],
			  AC_DEFINE([HAVE_BUILTIN_CONSTANT_P], [1],
                                    [Define if have __builtin_constant_p]))
AC_LINK_IFELSE([AC_LANG_SOURCE([int main(void) { return __builtin_types_compatible_p(char *, int) ? 1 : 0; }])],
			  AC_DEFINE([HAVE_BUILTIN_TYPES_COMPATIBLE_P], [1],
                                    [Define if have __builtin_types_compatible_p]))
AC_COMPILE_IFELSE([AC_LANG_SOURCE([static int __attribute__((warn_unused_result)) func(int x) { return x; }])],
			  AC_DEFINE([HAVE_WARN_UNUSED_RESULT], [1],
                                    [Define if __attribute__((warn_unused_result))]))

if test "x$prefix" = xNONE; then
	prefix=/usr/local
fi

AC_DEFINE_UNQUOTED([YACMINER_PREFIX], ["$prefix/bin"], [Path to zccminer install])

AC_DEFINE_UNQUOTED([PHATK_KERNNAME], ["phatk121016"], [Filename for phatk kernel])
AC_DEFINE_UNQUOTED([POCLBM_KERNNAME], ["poclbm130302"], [Filename for poclbm kernel])
AC_DEFINE_UNQUOTED([DIAKGCN_KERNNAME], ["diakgcn121016"], [Filename for diakgcn kernel])
AC_DEFINE_UNQUOTED([DIABLO_KERNNAME], ["diablo130302"], [Filename for diablo kernel])
AC_DEFINE_UNQUOTED([SCRYPT_KERNNAME], ["scrypt-chacha"], [Filename for scrypt kernel])


AC_SUBST(OPENCL_LIBS)
AC_SUBST(OPENCL_FLAGS)
AC_SUBST(JANSSON_LIBS)
AC_SUBST(PTHREAD_FLAGS)
AC_SUBST(DLOPEN_FLAGS)
AC_SUBST(PTHREAD_LIBS)
AC_SUBST(NCURSES_LIBS)
AC_SUBST(PDCURSES_LIBS)
AC_SUBST(WS2_LIBS)
AC_SUBST(MM_LIBS)
AC_SUBST(MATH_LIBS)
AC_SUBST(ADL_CPPFLAGS)

AC_CONFIG_FILES([
	Makefile
	compat/Makefile
	compat/jansson/Makefile
	ccan/Makefile
	lib/Makefile
	])
AC_OUTPUT


echo
echo
echo
echo "------------------------------------------------------------------------"
echo "$PACKAGE $VERSION"
echo "------------------------------------------------------------------------"
echo
echo
echo "Configuration Options Summary:"
echo

echo "  curses.TUI...........: $cursesmsg"

if test "x$opencl" != xno; then
	if test $found_opencl = 1; then
		echo "  OpenCL...............: FOUND. GPU mining support enabled"
	if test "x$scrypt" != xno; then
		echo "  scrypt...............: Enabled"
	else
		echo "  scrypt...............: Disabled"
	fi

	else
		echo "  OpenCL...............: NOT FOUND. GPU mining support DISABLED"
		if test "x$bitforce$avalon$icarus$ztex$modminer$bflsc" = xnononononono; then
			AC_MSG_ERROR([No mining configured in])
		fi
		echo "  scrypt...............: Disabled (needs OpenCL)"
	fi
else
	echo "  OpenCL...............: Detection overrided. GPU mining support DISABLED"
	if test "x$bitforce$icarus$avalon$ztex$modminer$bflsc" = xnononononono; then
		AC_MSG_ERROR([No mining configured in])
	fi
	echo "  scrypt...............: Disabled (needs OpenCL)"
fi

if test "x$adl" != xno; then
	if test x$have_adl = xtrue; then
		echo "  ADL..................: SDK found, GPU monitoring support enabled"
	else
		echo "  ADL..................: SDK NOT found, GPU monitoring support DISABLED"
	fi
else
	echo "  ADL..................: Detection overrided. GPU monitoring support DISABLED"
fi

echo
if test "x$avalon" = xyes; then
	echo "  Avalon.ASICs.........: Enabled"
else
	echo "  Avalon.ASICs.........: Disabled"
fi

if test "x$bflsc" = xyes; then
	echo "  BFL.ASICs............: Enabled"
else
	echo "  BFL.ASICs............: Disabled"
fi

if test "x$bitforce" = xyes; then
	echo "  BitForce.FPGAs.......: Enabled"
else
	echo "  BitForce.FPGAs.......: Disabled"
fi

if test "x$icarus" = xyes; then
	echo "  Icarus.FPGAs.........: Enabled"
else
	echo "  Icarus.FPGAs.........: Disabled"
fi

if test "x$modminer" = xyes; then
	echo "  ModMiner.FPGAs.......: Enabled"
else
	echo "  ModMiner.FPGAs.......: Disabled"
fi

if test "x$ztex" = xyes; then
	echo "  Ztex.FPGAs...........: Enabled"
else
	echo "  Ztex.FPGAs...........: Disabled"
fi

echo
echo "Compilation............: make (or gmake)"
echo "  CPPFLAGS.............: $CPPFLAGS"
echo "  CFLAGS...............: $CFLAGS $LIBUSB_CFLAGS"
echo "  LDFLAGS..............: $LDFLAGS $PTHREAD_FLAGS"
echo "  LDADD................: $DLOPEN_FLAGS $LIBCURL_LIBS $JANSSON_LIBS $PTHREAD_LIBS $OPENCL_LIBS $NCURSES_LIBS $PDCURSES_LIBS $WS2_LIBS $MATH_LIBS $LIBUSB_LIBS"
echo
echo "Installation...........: make install (as root if needed, with 'su' or 'sudo')"
echo "  prefix...............: $prefix"
echo

