##                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.
#
# Copyright (c) 2011-2013 Universidad Rey Juan Carlos and
#                         Pekka Jääskeläinen / Tampere University of Technology
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

AC_PREREQ([2.64])
AC_INIT([pocl], [0.9-pre], [pocl-devel@lists.sourceforge.net])
AC_CONFIG_SRCDIR([config.h.in])
AC_CONFIG_HEADER([config.h])
AC_CONFIG_MACRO_DIR([../../m4])
AC_CANONICAL_HOST
AC_CANONICAL_TARGET
AM_INIT_AUTOMAKE([foreign])
AM_SILENT_RULES([AM_DEFAULT_VERBOSITY=0])

case $host_os in
  darwin* )
    AC_SUBST([LIBRARY_SUFFIX], [".dylib"] )
    ;;
  freebsd* )
    AC_SUBST([LIBRARY_SUFFIX], [".so"] )
    ;;
  *)
    AC_SUBST([LIBRARY_SUFFIX], [".so"] )
    ;;
esac

AC_PROG_CC
AC_PROG_CXX
AC_PROG_LN_S
AC_C_BIGENDIAN

LT_INIT


####################################################################
# LLVM installation detection

AC_ARG_VAR([LLVM_CONFIG], [Program used to retrieve LLVM options and binaries])
AC_PATH_PROGS([LLVM_CONFIG],
              [llvm-config \
               llvm-config-3.3 llvm-config33 \
               llvm-config-3.4 llvm-config34 \
               llvm-config-3.2 llvm-config32])
test -z "$LLVM_CONFIG" && AC_MSG_FAILURE([no llvm-config found in \$PATH])

LLVM_VERSION=`$LLVM_CONFIG --version`
LLVM_BINDIR=`$LLVM_CONFIG --bindir`
LLVM_LIBDIR=`$LLVM_CONFIG --libdir`

AC_SUBST([LLVM_VERSION], [$LLVM_VERSION])

case "$LLVM_VERSION" in
  3.2*)
    AC_DEFINE([LLVM_3_2], [], "Using LLVM 3.2")
    ;;
  3.3*)
    AC_DEFINE([LLVM_3_3], [], "Using LLVM 3.3 RC")
    ;;
  3.4*)
    AC_DEFINE([LLVM_3_4], [], "Using LLVM svn (upcoming 3.4)")
    AC_DEFINE([LLVM_SVN], [], "Using LLVM svn (upcoming 3.4)")
    ;;
  *)
    AC_MSG_ERROR([Unsupported LLVM version. Please use LLVM 3.2 or later.])
    LLVM_VERSION=
    ;;
esac

# Search for "libLLVM-$LLVM_VERSION$LIBRARY_SUFFIX" (eg. libLLVM-3.4.so)
LLVM_SHARED_LIB_FILE=$LLVM_LIBDIR/libLLVM-$LLVM_VERSION$LIBRARY_SUFFIX
if test ! -e $LLVM_SHARED_LIB_FILE; then
  AC_MSG_WARN([
$LLVM_SHARED_LIB_FILE not found.
Recompile and install LLVM after ./configure --enable-shared.
Read the INSTALL file for details.])
fi

# Check LLVM RTTI option
LLVM_RTTI_CHECK=$($LLVM_CONFIG --cxxflags | grep -i "\-fno-rtti")
if test x"$LLVM_RTTI_CHECK" != x; then
  AC_MSG_WARN([Your LLVM was not built with RTTI.
You should rebuild LLVM with 'make REQUIRES_RTTI=1'.
See the INSTALL file for more information.])
fi

# Temporary option to enable the switch from scripts to LLVM API build
use_llvm_api="no"
AC_ARG_ENABLE([llvmapi],
              [AS_HELP_STRING([--enable-llvmapi], [Use LLVM via API, not scripts. (work in progress - do not use!)])],
              [use_llvm_api="yes"])
if test "$use_llvm_api" = "yes"; then
  AC_DEFINE([USE_LLVM_API], [1], "Use LLVM via API, rather than scripts")
fi
AM_CONDITIONAL(USE_LLVM_API, test "$use_llvm_api" = "yes")

# The kernel compiler opt plugin shared library, however, changes more
# drastically. Let's try to follow the similar 'current' numbering as
# the pocl host API library and perhaps tune the 'revision' and 'age' later.
AC_SUBST([KERNEL_COMPILER_LIB_VERSION], ["3:0:0"])

AC_CONFIG_FILES([Makefile])

AC_OUTPUT
