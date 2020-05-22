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

AC_PROG_SED

dnl Verify CUDA root.
define([defaultcudaroot], [/usr/local/cuda])
AC_ARG_WITH([cuda-root],
	AC_HELP_STRING([--with-cuda-root=DIR],
		[full path to CUDA installation @<:@defaultcudaroot@:>@]),
	[CUDA_ROOT_SYMLINK="$withval"], [CUDA_ROOT_SYMLINK=defaultcudaroot])
AC_CHECK_FILE([${CUDA_ROOT_SYMLINK}],
	[], AC_MSG_ERROR([CUDA root directory ${CUDA_ROOT_SYMLINK} does not exist!]))

dnl Parse CUDA version for pkg-config.
dnl
dnl To involke pkg-config we need to know the CUDA major and minor versions.
dnl Use CUDA's version.txt file to detect them.
CUDA_VERSION_FILE=${CUDA_ROOT_SYMLINK}/version.txt
AC_CHECK_FILE([${CUDA_VERSION_FILE}],
	[], AC_MSG_ERROR([Could not find CUDA version.txt]))
CUDA_VERSION=`${SED} -E 's/CUDA Version (.+)/\1/' ${CUDA_VERSION_FILE}`
dnl The version.txt file gives us MAJOR.MINOR.PATCH; extract MAJOR.MINOR for
dnl pkg-config queries.  These sed commands are from
dnl https://www.gnu.org/software/autoconf-archive/ax_split_version.html
CUDA_VERSION_MAJOR=`echo "$CUDA_VERSION" | ${SED} 's/\([[^.]][[^.]]*\).*/\1/'`
CUDA_VERSION_MINOR=`echo "$CUDA_VERSION" | ${SED} 's/[[^.]][[^.]]*.\([[^.]][[^.]]*\).*/\1/'`
CUDA_RELEASE=${CUDA_VERSION_MAJOR}.${CUDA_VERSION_MINOR}

dnl Use pkg-config to get appropriate flags and variables.
dnl
dnl CUDA-RT and R.
CUDA_MODULE=cudart-${CUDA_RELEASE}
PKG_CHECK_MODULES([CUDA], [${CUDA_MODULE}])
PKG_CHECK_VAR([CUDA_ROOT], [${CUDA_MODULE}], [cudaroot])
dnl R 3.6.0 onwards validates shared libraries by loading them during
dnl installation which often fails on HPC installations due to environment
dnl isolation.  Therefore use the standard HPC practise of encoding the rpath
dnl into the shared library.
PKG_CHECK_VAR([CUDA_LIBDIR], [${CUDA_MODULE}], [libdir])
CUDA_RPATH="-Xlinker -rpath,${CUDA_LIBDIR}"
NVCC=${CUDA_ROOT}/bin/nvcc
dnl https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#building-for-maximum-compatibility
NVCCFLAGS=
NVCCFLAGS+="-gencode=arch=compute_30,code=sm_30"
NVCCFLAGS+=" -gencode=arch=compute_35,code=sm_35"
NVCCFLAGS+=" -gencode=arch=compute_50,code=sm_50"
NVCCFLAGS+=" -gencode=arch=compute_60,code=sm_60"
NVCCFLAGS+=" -gencode=arch=compute_70,code=sm_70"
NVCCFLAGS+=" -gencode=arch=compute_75,code=sm_75"
NVCCFLAGS+=" -gencode=arch=compute_75,code=compute_75"
dnl Boost.
AC_CHECK_HEADERS([boost/version.hpp])
dnl R.
PKG_CHECK_MODULES([LIBR], [libR])
dnl Get CXXPICFLAGS from R config like gpuRcuda package.
PKG_CHECK_VAR([LIBR_HOME], [libR], [rhome])
R=${LIBR_HOME}/bin/R
AC_CHECK_FILE([${R}])
LIBR_CXXPICFLAGS=`${R} CMD config CXXPICFLAGS`
LIBR_CXXFLAGS=`${R} CMD config CXXFLAGS`

dnl Tweak flags for nvcc.
dnl
dnl Uses -Xlinker instead of -Wl to pass flags to the linker.
LIBR_LIBS=`echo ${LIBR_LIBS} | ${SED} 's/-Wl,/-Xlinker /g'`
dnl Add -Xcompiler to pass unknown flag -fopenmp to the compiler.
LIBR_LIBS=`echo ${LIBR_LIBS} | ${SED} 's/-fopenmp/-Xcompiler -fopenmp/'`

dnl Print configuration summary.
echo
echo "m4_text_box(Rgtsvm configuration)"
echo
echo "CUDA_VERSION     = ${CUDA_VERSION}"
echo "CUDA_ROOT        = ${CUDA_ROOT}"
echo "CUDA_CFLAGS      = ${CUDA_CFLAGS}"
echo "CUDA_LIBS        = ${CUDA_LIBS}"
echo "CUDA_RPATH       = ${CUDA_RPATH}"
echo
echo "CXX              = ${CXX}"
echo "CXXFLAGS         = ${CXXFLAGS}"
echo
echo "NVCC             = ${NVCC}"
echo "NVCCFLAGS        = ${NVCCFLAGS}"
echo
echo "LIBR_CFLAGS      = ${LIBR_CFLAGS}"
echo "LIBR_CXXFLAGS    = ${LIBR_CXXFLAGS}"
echo "LIBR_CXXPICFLAGS = ${LIBR_CXXPICFLAGS}"
echo "LIBR_LIBS        = ${LIBR_LIBS}"
echo

AC_SUBST([CUDA_CFLAGS])
AC_SUBST([CUDA_LIBS])
AC_SUBST([CUDA_RPATH])
AC_SUBST([NVCC])
AC_SUBST([NVCCFLAGS])
AC_SUBST([LIBR_CFLAGS])
AC_SUBST([LIBR_CXXFLAGS])
AC_SUBST([LIBR_CXXPICFLAGS])
AC_SUBST([LIBR_LIBS])

AC_OUTPUT
