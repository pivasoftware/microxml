dnl
dnl Configuration script for microxml, a micro XML library.
dnl
dnl Copyright 2003-2011 by Michael R Sweet.
dnl Copyright 2011-2012 by Luka Perkov.
dnl
dnl These coded instructions, statements, and computer programs are the
dnl property of Michael R Sweet and Luka Perkov. They are protected by
dnl Federal copyright law. Distribution and use rights are outlined in
dnl the file "COPYING" which should have been included with this file.
dnl If this file is missing or damaged, see the license at:
dnl
dnl     http://www.minixml.org/
dnl

dnl Specify a source file from the distribution...
AC_INIT(microxml.h)

dnl Set the name of the config header file...
AC_CONFIG_HEADER(config.h)

dnl Version number...
VERSION=1.0
AC_SUBST(VERSION)
AC_DEFINE_UNQUOTED(MICROXML_VERSION, "microxml v$VERSION")

dnl Clear default debugging options and set normal optimization by
dnl default unless the user asks for debugging specifically.
CFLAGS="${CFLAGS:=}"
CXXFLAGS="${CXXFLAGS:=}"
LDFLAGS="${LDFLAGS:=}"
AC_SUBST(LDFLAGS)
OPTIM=""
AC_SUBST(OPTIM)

AC_ARG_WITH(ansi, [AS_HELP_STRING([--with-ansi], [set full ANSI C mode, default=no])],
	use_ansi="$withval",
	use_ansi="no")

AC_ARG_WITH(archflags, [AS_HELP_STRING([--with-archflags], [set additional architecture flags, default=none])],
	ARCHFLAGS="$withval",
	ARCHFLAGS="")
AC_SUBST(ARCHFLAGS)

AC_ARG_ENABLE(debug, [AS_HELP_STRING([--enable-debug], [turn on debugging, default=no])],
if eval "test x$enable_debug = xyes"; then
 	OPTIM="-g"
fi)

AC_ARG_WITH(vsnprintf, [AS_HELP_STRING([--with-vsnprintf], [use vsnprintf emulation functions, default=auto])],
	use_vsnprintf="$withval",
	use_vsnprintf="no")

dnl Get the operating system and version number...
uname=`uname`
uversion=`uname -r | sed -e '1,$s/[[^0-9]]//g'`
if test x$uname = xIRIX64; then
	uname="IRIX"
fi

dnl Checks for programs...
AC_PROG_CC
AC_PROG_CXX
AC_PROG_INSTALL
if test "$INSTALL" = "$ac_install_sh"; then
	# Use full path to install-sh script...
	INSTALL="`pwd`/install-sh -c"
fi
AC_PROG_RANLIB
AC_PATH_PROG(AR,ar)
AC_PATH_PROG(CP,cp)
AC_PATH_PROG(LN,ln)
AC_PATH_PROG(MKDIR,mkdir)
AC_PATH_PROG(RM,rm)

dnl Flags for "ar" command...
case "$uname" in
        Darwin* | *BSD*)
                ARFLAGS="-rcv"
                ;;
        *)
                ARFLAGS="crvs"
                ;;
esac

AC_SUBST(ARFLAGS)

dnl Inline functions...
AC_C_INLINE

dnl Checks for string functions.
if test "x$use_ansi" != xyes; then
	AC_CHECK_FUNCS(strdup)
fi

if test "x$use_vsnprintf" != xyes; then
	AC_CHECK_FUNCS(snprintf vsnprintf)
fi

dnl Check for "long long" support...
AC_CACHE_CHECK(for long long int, ac_cv_c_long_long,
	[if test "$GCC" = yes; then
		ac_cv_c_long_long=yes
	else
		AC_TRY_COMPILE(,[long long int i;],
			ac_cv_c_long_long=yes,
			ac_cv_c_long_long=no)
	fi])

if test $ac_cv_c_long_long = yes; then
	AC_DEFINE(HAVE_LONG_LONG)
fi

dnl Threading support
AC_ARG_ENABLE(threads, [AS_HELP_STRING([--enable-threads], [enable multi-threading support, default=no])])

have_pthread=no
PTHREAD_FLAGS=""
PTHREAD_LIBS=""

if test "x$enable_threads" != xno; then
	AC_CHECK_HEADER(pthread.h, AC_DEFINE(HAVE_PTHREAD_H))

	if test x$ac_cv_header_pthread_h = xyes; then
		dnl Check various threading options for the platforms we support
		for flag in -lpthreads -lpthread -pthread; do
        		AC_MSG_CHECKING([for pthread_create using $flag])
			SAVELIBS="$LIBS"
			LIBS="$flag $LIBS"
        		AC_TRY_LINK([#include <pthread.h>],
				[pthread_create(0, 0, 0, 0);],
        			have_pthread=yes)
        		AC_MSG_RESULT([$have_pthread])
			LIBS="$SAVELIBS"

			if test $have_pthread = yes; then
				PTHREAD_FLAGS="-D_THREAD_SAFE -D_REENTRANT"
				PTHREAD_LIBS="$flag"

				# Solaris requires -D_POSIX_PTHREAD_SEMANTICS to
				# be POSIX-compliant... :(
				if test $uname = SunOS; then
					PTHREAD_FLAGS="$PTHREAD_FLAGS -D_POSIX_PTHREAD_SEMANTICS"
				fi
				break
			fi
		done
	fi
fi

AC_SUBST(PTHREAD_FLAGS)
AC_SUBST(PTHREAD_LIBS)

dnl Static library support...
AC_ARG_ENABLE(static, [AS_HELP_STRING([--enable-static], [turn on static libraries, default=yes])])
LIBMICROXML_STATIC=""

if test x$enable_static != xno; then
	PICFLAG=0
	LIBMICROXML_STATIC="libmicroxml.a"
fi

dnl Shared library support...
DSO="${DSO:=:}"
DSOFLAGS="${DSOFLAGS:=}"

AC_ARG_ENABLE(shared, [AS_HELP_STRING([--enable-shared], [turn on shared libraries, default=yes])])
LIBMICROXML_SHARED=""

if test x$enable_shared != xno; then
	AC_MSG_CHECKING(for shared library support)
	PICFLAG=1

	case "$uname" in
		SunOS* | UNIX_S*)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.so.1.0"
			DSO="\$(CC)"
			DSOFLAGS="$DSOFLAGS -Wl,-h,libmicroxml.so.1 -G -R\$(libdir) \$(OPTIM)"
			LDFLAGS="$LDFLAGS -R\$(libdir)"
			;;

		HP-UX*)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.sl.1"
			DSO="ld"
			DSOFLAGS="$DSOFLAGS -b -z +h libmicroxml.sl.1 +s +b \$(libdir)"
			LDFLAGS="$LDFLAGS -Wl,+s,+b,\$(libdir)"
			;;

		IRIX)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.so.1.0"
			DSO="\$(CC)"
			DSOFLAGS="$DSOFLAGS -Wl,-rpath,\$(libdir),-set_version,sgi1.0,-soname,libmicroxml.so.1 -shared \$(OPTIM)"
			;;

		OSF1* | Linux | GNU)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.so.1.0"
			DSO="\$(CC)"
			DSOFLAGS="$DSOFLAGS -Wl,-soname,libmicroxml.so.1,-rpath,\$(libdir) -shared \$(OPTIM)"
			LDFLAGS="$LDFLAGS -Wl,-rpath,\$(libdir)"
			;;

		*BSD*)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.so.1.0"
			DSO="\$(CC)"
			DSOFLAGS="$DSOFLAGS -Wl,-soname,libmicroxml.so.1,-R\$(libdir) -shared \$(OPTIM)"
			LDFLAGS="$LDFLAGS -Wl,-R\$(libdir)"
			;;

		Darwin*)
			AC_MSG_RESULT(yes)
			LIBMICROXML_SHARED="libmicroxml.1.dylib"
			DSO="\$(CC)"
			DSOFLAGS="$DSOFLAGS \$(RC_CFLAGS) -dynamiclib -lc"
			;;

		*)
			AC_MSG_RESULT(no)
			AC_MSG_WARN(shared libraries not supported on this platform.)
			LIBMICROXML_SHARED=""
			;;
	esac
fi

AC_SUBST(DSO)
AC_SUBST(DSOFLAGS)
AC_SUBST(LIBMICROXML_SHARED)
AC_SUBST(LIBMICROXML_STATIC)
AC_SUBST(PICFLAG)

dnl Add -Wall for GCC...
if test -n "$GCC"; then
	CFLAGS="-Wall $CFLAGS"

	if test "x$OPTIM" = x; then
		OPTIM="-Os -g"
	fi

	if test "x$use_ansi" = xyes; then
		CFLAGS="-ansi -pedantic $CFLAGS"
	fi

	if test $PICFLAG = 1 -a $uname != AIX; then
    		OPTIM="-fPIC $OPTIM"
	fi
else
	case $uname in
		HP-UX*)
			CFLAGS="-Ae $CFLAGS"

			if test "x$OPTIM" = x; then
				OPTIM="-O"
			fi

			OPTIM="+DAportable $OPTIM"

			if test $PICFLAG = 1; then
				OPTIM="+z $OPTIM"
			fi
			;;

		UNIX_SVR* | SunOS*)
			if test "x$OPTIM" = x; then
				OPTIM="-O"
			fi

			if test $PICFLAG = 1; then
				OPTIM="-KPIC $OPTIM"
			fi
			;;

		*)
			if test "x$OPTIM" = x; then
				OPTIM="-O"
			fi
			;;
	esac
fi

dnl Fix "prefix" variable if it hasn't been specified...
if test "$prefix" = "NONE"; then
	prefix="/usr/local"
fi

dnl Fix "exec_prefix" variable if it hasn't been specified...
if test "$exec_prefix" = "NONE"; then
	exec_prefix="$prefix"
fi

dnl pkg-config stuff...
if test "$includedir" != /usr/include; then
	PC_CFLAGS="-I$includedir"
else
	PC_CFLAGS=""
fi

if test "$libdir" != /usr/lib; then
	PC_LIBS="-L$libdir -lmicroxml"
else
	PC_LIBS="-lmicroxml"
fi

AC_SUBST(PC_CFLAGS)
AC_SUBST(PC_LIBS)

dnl Output the makefile, etc...
AC_OUTPUT(Makefile microxml.list microxml.pc)

