# If not 1, append revision to the version number and enable other development stuff
m4_define([gala_released], [0])

m4_define([gala_major_version], [0])
m4_define([gala_minor_version], [1])
m4_define([gala_micro_version], [0])
m4_define([gala_nano_version], [0])

# If library source has changed since last release, increment revision
# If interfaces have been removed or changed since last release,
#  increment current and set revision to 0
# If interfaces have been added since last release, increment age
# If interfaces have been removed since last release, set age to 0
m4_define([gala_lt_current], [0])
m4_define([gala_lt_revision], [0])
m4_define([gala_lt_age], [0])

# Display the nano_version only if it's not '0'
m4_define([gala_base_version],
          gala_major_version.gala_minor_version.gala_micro_version)
m4_define([gala_full_version],
          [m4_if(gala_nano_version, [0],
                 gala_base_version, gala_base_version.gala_nano_version)])

# Extend version with bzr revision for non-release builds
m4_define([gala_version], m4_if(gala_released, [1],
          [gala_full_version], [m4_esyscmd([build-aux/bzr-version-gen gala_full_version])]))
m4_define([gala_major_minor_version], gala_major_version.gala_minor_version)
m4_define([gala_module_version], gala_lt_current)

AC_INIT([gala], gala_version,
        [https://bugs.launchpad.net/gala], [gala],
        [https://launchpad.net/gala])
AC_DEFINE(RELEASE_NAME, "Window Manager.", Codename of this release)
AC_DEFINE(VERSION_INFO, m4_if(gala_released, [1], "Release", "Development"), Version info for package)

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_PREREQ([2.65])

AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS(config.h)
AC_CONFIG_SRCDIR([configure.ac])
AM_INIT_AUTOMAKE([1.11 tar-ustar dist-xz no-dist-gzip -Wno-portability subdir-objects])
AM_MAINTAINER_MODE

AC_PROG_CC
AM_PROG_CC_C_O
AC_DISABLE_STATIC
LT_INIT
PKG_PROG_PKG_CONFIG([0.21])

GLIB_GSETTINGS

AC_SUBST([CFLAGS])
AC_SUBST([CPPFLAGS])
AC_SUBST([LDFLAGS])

LT_CURRENT=gala_lt_current
LT_REVISION=gala_lt_revision
LT_AGE=gala_lt_age
AC_SUBST([LT_CURRENT])
AC_SUBST([LT_REVISION])
AC_SUBST([LT_AGE])

GALA_MAJOR_MINOR_VERSION=gala_major_minor_version
AC_SUBST([GALA_MAJOR_MINOR_VERSION])

API_VERSION=0.2
AC_SUBST([API_VERSION])


# -----------------------------------------------------------
# Vala
# -----------------------------------------------------------
VALA_MIN_VERSION=0.22.0

AM_PROG_VALAC([$VALA_MIN_VERSION])
if test "x$VALAC" = "x"; then
    AC_MSG_ERROR([Vala requested but valac >= $VALA_MIN_VERSION is not installed])
fi

AC_PATH_PROG([VAPIGEN], [vapigen], [])
if test "x$VAPIGEN" = "x"; then
    AC_MSG_ERROR([Vala must be built with --enable-vapigen])
fi

# -----------------------------------------------------------
# Dependencies
# -----------------------------------------------------------
GTK_MIN_VERSION=3.4.0
GLIB_MIN_VERSION=2.32.0
PLANK_MIN_VERSION=0.3.0
MUTTER_MIN_VERSION=3.8.4
MUTTER_API="3.8"

GALA_CORE_PKGS="gobject-2.0 >= $GLIB_MIN_VERSION \
                 glib-2.0 >= $GLIB_MIN_VERSION \
                 gio-2.0 >= $GLIB_MIN_VERSION \
                 gio-unix-2.0 >= $GLIB_MIN_VERSION \
                 gmodule-2.0 \
                 gtk+-3.0 >= $GTK_MIN_VERSION \
                 gdk-x11-3.0 >= $GTK_MIN_VERSION \
                 libbamf3 \
                 clutter-gtk-1.0 \
                 granite \
                 libmutter >= $MUTTER_MIN_VERSION \
                 gee-0.8 \
                 gnome-desktop-3.0 \
                 plank >= $PLANK_MIN_VERSION"

GALA_CORE_VALA_PKGS="--pkg gio-unix-2.0 \
                      --pkg gmodule-2.0 \
                      --pkg gtk+-3.0 \
                      --pkg gdk-x11-3.0 \
                      --pkg libbamf3 \
                      --pkg libmutter \
                      --pkg clutter-gtk-1.0 \
                      --pkg granite \
                      --pkg gee-0.8 \
                      --pkg gdesktopenums-3.0 \
                      --pkg gnome-desktop-3.0 \
                      --pkg xfixes-4.0 \
                      --pkg plank"

VALAFLAGS="$VALAFLAGS --thread --target-glib=2.32"


PKG_CHECK_MODULES(GALA_CORE, $GALA_CORE_PKGS)

# Plugins
PLUGIN_LDFLAGS="-module -avoid-version"
AC_SUBST([PLUGIN_LDFLAGS])

# -----------------------------------------------------------
# Optional Dependencies
# -----------------------------------------------------------

# 3.10
PKG_CHECK_MODULES(MUTTER, [libmutter >= 3.9], [have_mutter310=yes], [have_mutter310=no])
if test "x$have_mutter310" = "xyes" ; then
    PKG_CHECK_MODULES(MUTTER, "libmutter >= 3.10.4")
    VALAFLAGS="$VALAFLAGS --define HAS_MUTTER310"
    MUTTER_API="3.10"
fi

# 3.12
PKG_CHECK_MODULES(MUTTER, [libmutter >= 3.11], [have_mutter312=yes], [have_mutter312=no])
if test "x$have_mutter312" = "xyes" ; then
    PKG_CHECK_MODULES(MUTTER, [libmutter >= 3.11.91])
    VALAFLAGS="$VALAFLAGS --define HAS_MUTTER312"
    MUTTER_API="3.12"
fi

# -----------------------------------------------------------
# Addional configure flags
# -----------------------------------------------------------

# Enable Vala debug-flag by default for non-release builds
AC_ARG_ENABLE([debug],
              AS_HELP_STRING([--disable-debug],[compile without debug code]),
              [enable_debug=$enableval], [m4_if(gala_released, [1], [enable_debug=no], [enable_debug=yes])])
if test "x$enable_debug" = "xyes" ; then
    VALAFLAGS="$VALAFLAGS -g"
fi

GALA_CORE_VALAFLAGS="$VALAFLAGS $GALA_CORE_VALA_PKGS"
AC_SUBST(GALA_CORE_VALAFLAGS)

# -----------------------------------------------------------
# Gettext
# -----------------------------------------------------------
GETTEXT_PACKAGE=AC_PACKAGE_NAME
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, ["$GETTEXT_PACKAGE"], [Define the gettext package to be used])
IT_PROG_INTLTOOL([0.35.0])


# -----------------------------------------------------------
# Final output
# -----------------------------------------------------------
AC_CONFIG_FILES([
Makefile
src/Makefile
lib/Makefile
data/Makefile
vapi/Makefile
plugins/Makefile
plugins/zoom/Makefile
po/Makefile.in
])

AC_OUTPUT

echo "
${PACKAGE} ${VERSION}

    Prefix......................:  ${prefix}
    Vala Compiler...............:  ${VALAC}
    Vala Flags..................:  ${VALAFLAGS}
    C Compiler Flags............:  ${CFLAGS}

    Using Mutter................:  ${MUTTER_API}
"