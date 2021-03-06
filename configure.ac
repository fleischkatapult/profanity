#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_INIT([profanity], [0.4.7], [boothj5web@gmail.com])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([foreign subdir-objects])

### Checks for programs.
AC_PROG_CC

### Get canonical host
AC_CANONICAL_HOST
PLATFORM="unknown"
AS_CASE([$host_os],
    [freebsd*], [PLATFORM="freebsd"],
    [darwin*], [PLATFORM="osx"],
    [cygwin], [PLATFORM="cygwin"],
    [PLATFORM="nix"])

PACKAGE_STATUS="development"

### Get git branch and revision if in development
if test "x$PACKAGE_STATUS" = xdevelopment; then
    AM_CONDITIONAL([INCLUDE_GIT_VERSION], [true])
    AC_DEFINE([HAVE_GIT_VERSION], [1], [Include git info])
else
    AM_CONDITIONAL([INCLUDE_GIT_VERSION], [false])
fi

AC_DEFINE_UNQUOTED([PACKAGE_STATUS], ["$PACKAGE_STATUS"], [Status of this build])

AS_IF([test "x$PLATFORM" = xcygwin],
    [AC_DEFINE([PLATFORM_CYGWIN], [1], [Cygwin])])
AS_IF([test "x$PLATFORM" = xosx],
    [AC_DEFINE([PLATFORM_OSX], [1], [OSx])])

### Options
AC_ARG_ENABLE([notifications],
    [AS_HELP_STRING([--enable-notifications], [enable desktop notifications])])
AC_ARG_ENABLE([otr],
    [AS_HELP_STRING([--enable-otr], [enable otr encryption])])
AC_ARG_WITH([libxml2],
    [AS_HELP_STRING([--with-libxml2], [link with libxml2 instead of expat])])
AC_ARG_WITH([xscreensaver],
    [AS_HELP_STRING([--with-xscreensaver], [use libXScrnSaver to determine idle time])])
AC_ARG_WITH([themes],
    [AS_HELP_STRING([--with-themes[[=PATH]]], [install themes (default yes)])])

# Use the CFLAGS and libs provided by libstrophe.pc if libstrophe has pkg-config support.
PKG_CHECK_MODULES([libstrophe], [libstrophe], [LIBS="$libstrophe_LIBS $LIBS"],
        [
	 if test "x$PLATFORM" != xfreebsd; then
		AC_CHECK_LIB([resolv], [res_query], [],
		    [AC_CHECK_LIB([resolv], [__res_query], [],
			[AC_MSG_ERROR([libresolv not found; libresolv required for profanity])])])
	 else
		AC_MSG_NOTICE([skipping libresolv checks for freebsd])
	 fi

         ### Check for libstrophe dependencies
         ### Select first existing xml library among expat and libxml2
         PARSER=""
         PARSER_LIBS=""
         PARSER_CFLAGS=""
         AS_IF([test "x$with_libxml2" != xyes],
             [PKG_CHECK_MODULES([expat], [expat],
                 [PARSER_CFLAGS="$expat_CFLAGS"; PARSER_LIBS="$expat_LIBS"; PARSER="expat"],
                 [AC_CHECK_LIB([expat], [XML_ParserCreate],
                     [PARSER_LIBS="-lexpat"; PARSER="expat"],
                     AS_IF([test "x$with_libxml2" = xno],
                         [AC_MSG_ERROR([expat is required but does not exist])]))])
             ])

         AS_IF([test "x$PARSER" = x -a "x$with_libxml2" != xno],
             [PKG_CHECK_MODULES([libxml2], [libxml-2.0],
                 [PARSER_CFLAGS="$libxml2_CFLAGS"; PARSER_LIBS="$libxml2_LIBS"; PARSER="libxml2"],
                 AS_IF([test "x$with_libxml2" = xyes],
                     [AC_MSG_ERROR([libxml2 is required but does not exist])]))
             ])

         AS_IF([test "x$PARSER" = x],
             [AC_MSG_ERROR([either expat or libxml2 is required for profanity])])
         AM_CPPFLAGS="$AM_CPPFLAGS $PARSER_CFLAGS"
         LIBS="$PARSER_LIBS $LIBS"

         PKG_CHECK_MODULES([openssl], [openssl], [],
                 [AC_CHECK_HEADER(openssl/ssl.h, [openssl_LIBS="-lssl -lcrypto"], [AC_MSG_ERROR([openssl not found; openssl required for profanity])])])
         AM_CPPFLAGS="$AM_CPPFLAGS $openssl_CFLAGS"
         LIBS="$openssl_LIBS $LIBS"

         CFLAGS_RESTORE="$CFLAGS"
         CFLAGS="$CFLAGS $AM_CPPFLAGS"
         AC_CHECK_LIB([strophe], [xmpp_ctx_new], [LIBS="-lstrophe $LIBS"; listrophe_CFLAGS=""],
                [AC_MSG_ERROR([libstrophe linked with $PARSER is required for profanity])])
         CFLAGS="$CFLAGS_RESTORE"
        ])
CFLAGS="$CFLAGS $libstrophe_CFLAGS"

### Check for ncurses library
PKG_CHECK_MODULES([ncursesw], [ncursesw],
    [NCURSES_CFLAGS="$ncursesw_CFLAGS"; NCURSES_LIBS="$ncursesw_LIBS"; NCURSES="ncursesw"],
    [PKG_CHECK_MODULES([ncurses], [ncurses],
        [NCURSES_CFLAGS="$ncurses_CFLAGS"; NCURSES_LIBS="$ncurses_LIBS"; NCURSES="ncurses"],
        [AC_CHECK_LIB([ncursesw], [main], [],
            [AC_CHECK_LIB([ncurses], [main], [],
            	[AC_MSG_ERROR([ncurses is required for profanity])])])])])
AM_CPPFLAGS="$AM_CPPFLAGS $NCURSES_CFLAGS"
LIBS="$NCURSES_LIBS $LIBS"
AS_IF([test "x$PLATFORM" = xosx], [LIBS="-lncurses $LIBS"])

### Check wide characters support in ncurses library
CFLAGS_RESTORE="$CFLAGS"
CFLAGS="$CFLAGS $NCURSES_CFLAGS"
AC_CACHE_CHECK([for wget_wch support in $NCURSES], ncurses_cv_wget_wch,
   [AC_LINK_IFELSE([AC_LANG_SOURCE([
       void wget_wch(void);
       int main() {
           wget_wch();
           return 0;
       }
       ])],
       [ncurses_cv_wget_wch=yes],
       [ncurses_cv_wget_wch=no])
   ])
CFLAGS="$CFLAGS_RESTORE"

AS_IF([test "x$ncurses_cv_wget_wch" != xyes],
    [AC_MSG_ERROR([ncurses does not support wide characters])])

### Check for other profanity dependencies
PKG_CHECK_MODULES([glib], [glib-2.0 >= 2.26], [],
    [AC_MSG_ERROR([glib 2.26 or higher is required for profanity])])
PKG_CHECK_MODULES([curl], [libcurl], [],
    [AC_MSG_ERROR([libcurl is required for profanity])])

AS_IF([test "x$PLATFORM" != xosx],
    [AC_CHECK_LIB([readline], [main], [],
        [AC_MSG_ERROR([libreadline is required for profanity])])],
    [AC_CHECK_FILE([/usr/local/opt/readline/lib],
        [LIBS="-lreadline $LIBS"
            AM_CPPFLAGS="-I/usr/local/opt/readline/include $AM_CPPFLAGS"
            AM_LDFLAGS="-L/usr/local/opt/readline/lib $AM_LDFLAGS"
            AC_SUBST(AM_LDFLAGS)],
        [AC_MSG_ERROR([libreadline is required for profanity])])])

AS_IF([test "x$PLATFORM" = xosx], [LIBS="-lcurl $LIBS"])

### Check for desktop notification support
### Linux/FreeBSD require libnotify
### Windows uses native OS calls
### OSX requires terminal-notifier

AS_IF([test "x$PLATFORM" = xosx],
        [AS_IF([test "x$enable_notifications" != xno],
            [NOTIFIER_PATH="no"
            AC_PATH_PROG(NOTIFIER_PATH, terminal-notifier, no)
            AS_IF([test "x$NOTIFIER_PATH" = xno],
                [AS_IF([test "x$enable_notifications" = xyes],
                    [AC_MSG_ERROR([terminal-notifier not found, required for desktop notifications.])],
                    [AC_MSG_NOTICE([Desktop notifications not supported.])])],
                [AC_DEFINE([HAVE_OSXNOTIFY], [1], [terminal notifier])])])],
    [test "x$PLATFORM" = xnix -o "x$PLATFORM" = xfreebsd],
        [AS_IF([test "x$enable_notifications" != xno],
            [PKG_CHECK_MODULES([libnotify], [libnotify],
                [AC_DEFINE([HAVE_LIBNOTIFY], [1], [libnotify module])],
                [AS_IF([test "x$enable_notifications" = xyes],
                    [AC_MSG_ERROR([libnotify is required but does not exist])],
                    [AC_MSG_NOTICE([libnotify support will be disabled])])])])])

# TODO: rewrite this
if test "x$with_xscreensaver" = xyes; then
    AC_CHECK_LIB([Xss], [main], [],
        [AC_MSG_ERROR([libXss is required for x autoaway support])])
    AC_CHECK_LIB([X11], [main], [],
        [AC_MSG_ERROR([libX11 is required for x autoaway support])])
elif test "x$with_xscreensaver" = x; then
    AC_CHECK_LIB([Xss], [main], [],
        [AC_MSG_NOTICE([libXss not found, falling back to profanity auto-away])])
    AC_CHECK_LIB([X11], [main], [],
        [AC_MSG_NOTICE([libX11 not found, falling back to profanity auto-away])])
fi

AM_CONDITIONAL([BUILD_OTR], [false])
AM_CONDITIONAL([BUILD_OTR3], [false])
AM_CONDITIONAL([BUILD_OTR4], [false])
if test "x$enable_otr" = xyes; then
    AC_SEARCH_LIBS([otrl_init], [otr],
        [AC_COMPILE_IFELSE(
            [AC_LANG_PROGRAM([[
            #include <libotr/version.h>
            ]],[[
            #if OTRL_VERSION_MAJOR == 4
            // OK
            #else
            # assume version 3
            #endif
            ]])],
            [AM_CONDITIONAL([BUILD_OTR], [true]) AM_CONDITIONAL([BUILD_OTR4], [true]) AC_DEFINE([HAVE_LIBOTR], [1], [Have libotr])],
            [AM_CONDITIONAL([BUILD_OTR], [true]) AM_CONDITIONAL([BUILD_OTR3], [true]) AC_DEFINE([HAVE_LIBOTR], [1], [Have libotr])])],
        [AC_MSG_ERROR([libotr is required for otr encryption support])])
elif test "x$enable_otr" = x; then
    AC_SEARCH_LIBS([otrl_init], [otr],
        [AC_COMPILE_IFELSE(
            [AC_LANG_PROGRAM([[
            #include <libotr/version.h>
            ]],[[
            #if OTRL_VERSION_MAJOR == 4
            // OK
            #else
            # assume version 3
            #endif
            ]])],
            [AM_CONDITIONAL([BUILD_OTR], [true]) AM_CONDITIONAL([BUILD_OTR4], [true]) AC_DEFINE([HAVE_LIBOTR], [1], [Have libotr])],
            [AM_CONDITIONAL([BUILD_OTR], [true]) AM_CONDITIONAL([BUILD_OTR3], [true]) AC_DEFINE([HAVE_LIBOTR], [1], [Have libotr])])],
        [AC_MSG_NOTICE([libotr not found, otr encryption support not enabled])])
fi

AS_IF([test "x$with_themes" = xno],
    [THEMES_INSTALL="false"],
    [THEMES_INSTALL="true"])
AS_IF([test "x$with_themes" = xno -o "x$with_themes" = xyes -o "x$with_themes" = x],
    [THEMES_PATH='${pkgdatadir}/themes'],
    [THEMES_PATH="$with_themes"])
AC_SUBST(THEMES_PATH)
AM_CONDITIONAL([THEMES_INSTALL], "$THEMES_INSTALL")

### cmocka is required only for tests, profanity shouldn't be linked with it
### TODO: pass cmocka_CFLAGS and cmocka_LIBS to Makefile.am
PKG_CHECK_MODULES([cmocka], [cmocka], [],
    [AC_MSG_NOTICE([cmocka is not found, will not be able to run tests])])

### Check for ncursesw/ncurses.h first, Arch linux uses ncurses.h for ncursesw
AC_CHECK_HEADERS([ncursesw/ncurses.h], [], [])
AC_CHECK_HEADERS([ncurses.h], [], [])

### Default parameters
AM_CFLAGS="-Wall -Wno-deprecated-declarations"
AS_IF([test "x$PACKAGE_STATUS" = xdevelopment],
    [AM_CFLAGS="$AM_CFLAGS -Wunused -Werror"])
AM_CPPFLAGS="$AM_CPPFLAGS $glib_CFLAGS $curl_CFLAGS $libnotify_CFLAGS"
AM_CPPFLAGS="$AM_CPPFLAGS -DTHEMES_PATH=\"\\\"$THEMES_PATH\\\"\""
LIBS="$glib_LIBS $curl_LIBS $libnotify_LIBS $LIBS"

AC_SUBST(AM_CFLAGS)
AC_SUBST(AM_CPPFLAGS)

### Checks for library functions.
AC_CHECK_FUNCS([atexit memset strdup strstr])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT

echo ""
echo "PLATFORM       : $host_os"
echo "PACKAGE_STATUS : $PACKAGE_STATUS"
echo "AM_CFLAGS      : $AM_CFLAGS"
echo "AM_CPPFLAGS    : $AM_CPPFLAGS"
echo "LIBS           : $LIBS"
echo "XML Parser     : $PARSER"
echo "Install themes : $THEMES_INSTALL"
echo "Themes path    : $THEMES_PATH"
echo ""
echo "Now you can run \`make' to build profanity"
