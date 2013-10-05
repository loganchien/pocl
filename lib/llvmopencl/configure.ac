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
AC_CANONICAL_TARGET
AC_CANONICAL_HOST
AM_INIT_AUTOMAKE([foreign])
AM_SILENT_RULES([AM_DEFAULT_VERBOSITY=0])

AM_CONDITIONAL([DARWIN_HOST_OS], false)
AM_CONDITIONAL([FREEBSD_HOST_OS], false)
case $host_os in
  darwin* )
    AM_CONDITIONAL([DARWIN_HOST_OS], true)
    AC_SUBST([LIBRARY_SUFFIX], [".dylib"] )
         ;;
  freebsd* )
    AM_CONDITIONAL([FREEBSD_HOST_OS], true)
    AC_SUBST([LIBRARY_SUFFIX], [".so"] )
         ;;
  *)
    AC_SUBST([LIBRARY_SUFFIX], [".so"] )
         ;;
esac

# Checks for programs.
AC_PROG_CC
AC_PROG_CXX
AC_PROG_LN_S

LT_INIT

# The extra LDFLAGS that must be given to link final binaries against pocl
LD_FLAGS_BIN=""

AC_ARG_VAR([LLVM_CONFIG], [Program used to retrieve LLVM options and binaries])
AC_PATH_PROGS([LLVM_CONFIG], [llvm-config llvm-config-3.3 llvm-config33 llvm-config-3.4 llvm-config34 llvm-config-3.2 llvm-config32 ])
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
   AC_MSG_ERROR(
   [
Unsupported LLVM version. Please use LLVM 3.2 or later.
   ])
   LLVM_VERSION=
   ;;
esac

LLVM_SHARED_LIB_FILE=$LLVM_LIBDIR/libLLVM-$LLVM_VERSION$LIBRARY_SUFFIX

if test ! -e $LLVM_SHARED_LIB_FILE;
then
        AC_MSG_WARN([
$LLVM_SHARED_LIB_FILE not found.
Recompile and install LLVM after ./configure --enable-shared.
Read the INSTALL file for details.])
fi

LLVM_RTTI_CHECK=$($LLVM_CONFIG --cxxflags | grep -i "\-fno-rtti")

if test x"$LLVM_RTTI_CHECK" != x; then
AC_MSG_WARN([Your LLVM was not built with RTTI.
You should rebuild LLVM with 'make REQUIRES_RTTI=1'.
See the INSTALL file for more information.])
fi

AC_ARG_ENABLE([region-allocator],
[AS_HELP_STRING([--disable-region-allocator],
 [Do not use a custom OpenCL optimized region-based memory allocator but allocate buffers with malloc directly.])],
[],
[AC_DEFINE([CUSTOM_BUFFER_ALLOCATOR], [], "Use a custom buffer allocator")])

#Temporary option to enable the switch from scripts to llvm API build
use_llvm_api="no"
AC_ARG_ENABLE([llvmapi],
              [AS_HELP_STRING([--enable-llvmapi], [Use LLVM via API, not scripts. (work in progress - do not use!)])],
              [use_llvm_api="yes"])
if test "$use_llvm_api" = "yes"; then
  AC_DEFINE([USE_LLVM_API], [1], "Use LLVM via API, rather than scripts")
fi
AM_CONDITIONAL(USE_LLVM_API, test "$use_llvm_api" = "yes")

AC_ARG_VAR([CLFLAGS], [Flags to be used when compiling CL sources])

AC_DEFINE_UNQUOTED([CLANG], ["$CLANG"], [clang executable.])
AC_DEFINE_UNQUOTED([CLFLAGS], ["$CLFLAGS"], [CL compiler flags.])

AC_DEFUN([LLVM_PROG], [
  AC_ARG_VAR([$1], [$3])
  AC_PATH_PROGS([$1], [$2 $2-$LLVM_VERSION], , [$LLVM_BINDIR:$PATH])
  AS_IF([test -z "$$1"], [
    AC_MSG_FAILURE([cannot find LLVM binary '$2' in '$LLVM_BINDIR' nor \$PATH])
  ])
])

AC_DEFUN([LLVM_PROG_WARN], [
  AC_ARG_VAR([$1], [$3])
  AC_PATH_PROGS([$1], [$2 $2-$LLVM_VERSION], , [$LLVM_BINDIR:$PATH])
  AS_IF([test -z "$$1"], [
    AC_MSG_WARN([cannot find LLVM binary '$2' in '$LLVM_BINDIR' nor \$PATH])
  ])
])

LLVM_PROG([CLANG], [clang], [Program to compile kernels to bytecode])

# Check for a functioning clang++. It is needed to compile faster math
# builtins from vecmathlib.
LLVM_PROG_WARN([CLANGXX], [clang++])

LLVM_PROG([OPT], [opt], [LLVM optimizer])
LLVM_PROG([LLC], [llc], [LLVM static compiler])
LLVM_PROG([LLVM_AS], [llvm-as], [LLVM assembler])
LLVM_PROG([LLVM_AR], [llvm-ar], [LLVM archiver])
LLVM_PROG([LLVM_LINK], [llvm-link], [LLVM IR linker])


# $CLANG and $LLC are needed by config/xclang
export CLANG
export LLC

AC_DEFINE_UNQUOTED([LLC], ["$LLC"], [LLVM compiler executable.])

AC_MSG_CHECKING([for aligned_alloc])
AC_COMPILE_IFELSE(
    [AC_LANG_PROGRAM(
        [[ #include <stdlib.h> ]],
        [[ void* p = aligned_alloc(0, 0); ]]
    )],
    [AC_MSG_RESULT(yes)
    AC_DEFINE([HAVE_ALIGNED_ALLOC], [], [Defined if aligned_alloc is available.])],
    [AC_MSG_RESULT(no)])

AC_MSG_CHECKING([for posix_memalign])
AC_COMPILE_IFELSE(
    [AC_LANG_PROGRAM(
        [[ #include <stdlib.h> ]],
        [[ int r = posix_memalign(NULL, 0, 0); ]]
    )],
    [AC_MSG_RESULT(yes)
    AC_DEFINE([HAVE_POSIX_MEMALIGN], [], [Defined if posix_memalign is available.])],
    [AC_MSG_RESULT(no)])

# Checks for typedefs, structures, and compiler characteristics.
AC_MSG_NOTICE([Checking target compiler characteristics])
CFLAGS="-h $target"
AC_CHECK_SIZEOF([long])
AC_SUBST([TARGET_SIZEOF_LONG], [$ac_cv_sizeof_long])
AC_CHECK_SIZEOF([__fp16])
AC_SUBST([TARGET_SIZEOF_HALF], [$ac_cv_sizeof___fp16])
AC_CHECK_SIZEOF([double])
AC_SUBST([TARGET_SIZEOF_DOUBLE], [$ac_cv_sizeof_double])
AC_CHECK_SIZEOF([void *])
AC_SUBST([TARGET_SIZEOF_VOID_P], [$ac_cv_sizeof_void_p])


AS_UNSET([ac_cv_sizeof_long])
AS_UNSET([ac_cv_sizeof___fp16])
AS_UNSET([ac_cv_sizeof_double])
AS_UNSET([ac_cv_sizeof_void_p])

AC_MSG_NOTICE([Checking host compiler characteristics])
CFLAGS="-h $host"
AC_CHECK_SIZEOF([long])
AC_SUBST([HOST_SIZEOF_LONG], [$ac_cv_sizeof_long])
AC_CHECK_SIZEOF([__fp16])
AC_SUBST([HOST_SIZEOF_HALF], [$ac_cv_sizeof___fp16])
AC_CHECK_SIZEOF([double])
AC_SUBST([HOST_SIZEOF_DOUBLE], [$ac_cv_sizeof_double])
AC_CHECK_SIZEOF([void *])
AC_SUBST([HOST_SIZEOF_VOID_P], [$ac_cv_sizeof_void_p])

AC_CHECK_ALIGNOF([float16], [typedef float float16  __attribute__((__ext_vector_type__(16)));])
AC_CHECK_ALIGNOF([double16], [typedef double double16  __attribute__((__ext_vector_type__(16)));])
AS_IF([test $ac_cv_alignof_float16 -gt $ac_cv_alignof_double16],
      [AC_DEFINE([MAX_EXTENDED_ALIGNMENT], [ALIGNOF_FLOAT16], [Defined to greatest expected alignment for extended types, in bytes.])],
      [AC_DEFINE([MAX_EXTENDED_ALIGNMENT], [ALIGNOF_DOUBLE16], [Defined to greatest expected alignment for extended types, in bytes.])])

AC_C_BIGENDIAN

# The libtool library version string to use (passed to -version-info).
# See: http://www.nondot.org/sabre/Mirrored/libtool-2.1a/libtool_6.html
# libpocl.so should get only API additions as we are implementing a standard.
# 0:0:0 == 0.6
# 1:0:0 == 0.7 (not backwards compatible with 0:0:0 due to the ICD)
# 2:0:1 == 0.8 (currently backwards compatible with 0.7, thus age = 1).
# 3:0:2 == 0.9 (currently backwards compatible with 0.7, thus age = 2).
AC_SUBST([LIB_VERSION], ["3:0:2"])

# The kernel compiler opt plugin shared library, however, changes more
# drastically. Let's try to follow the similar 'current' numbering as
# the pocl host API library and perhaps tune the 'revision' and 'age' later.
AC_SUBST([KERNEL_COMPILER_LIB_VERSION], ["3:0:0"])

AC_SUBST([LD_FLAGS_BIN])

AC_CONFIG_FILES([Makefile])

AC_OUTPUT
