dnl Rgtsvm package
dnl
dnl Zhong Wang <zw355@cornell.edu> wrote originally modified the configure.ac
dnl file from the magma package
dnl
dnl Pariksheet Nanda <pariksheet.nanda@uconn.edu> rewrote the file to:
dnl - Support CUDA versions newer than 8.0
dnl - Import all CUDA compilation and link flags via pkg-config.
dnl - Use appropriate Autoconf macros.
dnl - Print configuration summary.

m4_define([pkgversion],
	m4_esyscmd([sed -E -n 's/Version: (.+)/\1/p' DESCRIPTION | tr -d '\n']))
AC_INIT([Rgtsvm], [pkgversion])
m4_include([tools/pkg.m4])
AC_CONFIG_FILES([src/Makefile])

AC_LANG([C++])
AC_PROG_CXX([g++])

AC_PROG_EGREP
AC_PROG_SED

dnl Verify CUDA root.
define([defaultcudaroot], [/usr/local/cuda])
AC_ARG_WITH([cuda-root],
	AC_HELP_STRING([--with-cuda-root=DIR],
		[full path to CUDA installation @<:@defaultcudaroot@:>@]),
	[cuda_ROOT_SYMLINK="$withval"], [cuda_ROOT_SYMLINK=defaultcudaroot])
AC_CHECK_FILE([${cuda_ROOT_SYMLINK}],
	[], AC_MSG_ERROR([CUDA root directory ${cuda_ROOT_SYMLINK} does not exist!]))

dnl Parse CUDA version for pkg-config.
dnl
dnl To involke pkg-config we need to know the CUDA major and minor versions.
dnl Use CUDA's version.txt file to detect them.
cuda_VERSION_FILE=${cuda_ROOT_SYMLINK}/version.txt
AC_CHECK_FILE([${cuda_VERSION_FILE}],
	[], AC_MSG_ERROR([Could not find CUDA version.txt]))
cuda_VERSION=`${EGREP} -o '[[^ ]]+$' ${cuda_VERSION_FILE}`
dnl The version.txt file gives us MAJOR.MINOR.PATCH; extract MAJOR.MINOR for
dnl pkg-config queries.  These sed commands are from
dnl https://www.gnu.org/software/autoconf-archive/ax_split_version.html
cuda_VERSION_MAJOR=`echo "$cuda_VERSION" | ${SED} 's/\([[^.]][[^.]]*\).*/\1/'`
cuda_VERSION_MINOR=`echo "$cuda_VERSION" | ${SED} 's/[[^.]][[^.]]*.\([[^.]][[^.]]*\).*/\1/'`
cuda_RELEASE=${cuda_VERSION_MAJOR}.${cuda_VERSION_MINOR}

dnl Use pkg-config to get appropriate flags and variables.
dnl
dnl CUDA, CUDA-RT and R.
cuda_MODULE=cudart-${cuda_RELEASE}
PKG_CHECK_MODULES([cuda], [${cuda_MODULE}])
PKG_CHECK_VAR([cuda_ROOT], [${cuda_MODULE}], [cudaroot])
dnl R 3.6.0 onwards validates shared libraries by loading them during
dnl installation which often fails on HPC installations due to environment
dnl isolation.  Therefore use the standard HPC practise of encoding the rpath
dnl into the shared library.
PKG_CHECK_VAR([cuda_LIBDIR], [${cuda_MODULE}], [libdir])
cuda_RPATH="-Wl,-rpath,${cuda_LIBDIR}"
cuda_NVCC=${cuda_ROOT}/bin/nvcc
dnl https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#building-for-maximum-compatibility
cuda_NVCCFLAGS=
cuda_NVCCFLAGS+="-gencode=arch=compute_30,code=sm_30"
cuda_NVCCFLAGS+=" -gencode=arch=compute_35,code=sm_35"
cuda_NVCCFLAGS+=" -gencode=arch=compute_50,code=sm_50"
cuda_NVCCFLAGS+=" -gencode=arch=compute_60,code=sm_60"
cuda_NVCCFLAGS+=" -gencode=arch=compute_70,code=sm_70"
cuda_NVCCFLAGS+=" -gencode=arch=compute_75,code=sm_75"
cuda_NVCCFLAGS+=" -gencode=arch=compute_75,code=compute_75"
dnl Boost.
AC_CHECK_HEADERS([boost/version.hpp])
dnl R.
PKG_CHECK_MODULES([r], [libR])

dnl Print configuration summary.
echo
echo "m4_text_box(Rgtsvm configuration)"
echo
echo "CUDA_VERSION = ${cuda_VERSION}"
echo "CUDA_ROOT    = ${cuda_ROOT}"
echo "CUDA_CFLAGS  = ${cuda_CFLAGS}"
echo "CUDA_LIBS    = ${cuda_LIBS}"
echo "CUDA_RPATH   = ${cuda_RPATH}"
echo "CXX          = ${CXX}"
echo "CXXFLAGS     = ${CXXFLAGS}"
echo "NVCC         = ${cuda_NVCC}"
echo "NVCCFLAGS    = ${cuda_NVCCFLAGS}"
echo "R_CFLAGS     = ${r_CFLAGS}"
echo "R_LIBS       = ${r_LIBS}"
echo

AC_SUBST([cuda_CFLAGS])
AC_SUBST([cuda_LIBS])
AC_SUBST([cuda_RPATH])
AC_SUBST([cuda_NVCC])
AC_SUBST([cuda_NVCCFLAGS])
AC_SUBST([r_CFLAGS])
AC_SUBST([r_LIBS])

AC_OUTPUT
