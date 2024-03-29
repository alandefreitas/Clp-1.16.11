cmake_minimum_required(VERSION 3.0.2)

project(CoinUtils)

# CoinUtils options
option(COINUTILS_USE_THREADS              "enables compilation of thread aware CoinUtils (mempool so far)" OFF)
option(COINUTILS_USE_MEMPOOL_OVERRIDE_NEW "enables the CoinUtils mempool to override global new/delete" OFF)
option(COINUTILS_USE_MEMPOOL_MAXPOOLED    "Specify the default maximum memory allocation size that is served by the memory pool. If negative (or 'no') then the memory pool is disabled completely. Otherwise its value can be overridden at runtime using the COINUTILS_MEMPOOL_MAXPOOLED environment variable." OFF)
option(COINUTILS_HAS_ZLIB  "CoinUtils uses Zlib" OFF)
option(COINUTILS_HAS_BZLIB "CoinUtils uses BZlib" OFF)
option(COIN_HAS_LAPACK     "Enable the Lapack support of CoinUtils" OFF)
option(COIN_HAS_BLAS       "Enable the Blas support of CoinUtils" OFF)
option(COIN_HAS_MKL        "Use Intel MKL library (requires Intel compiler)" OFF)
option(COIN_USE_FAST_CODE  "Use fast code for CoinUtils" OFF)

set(COIN_COINUTILS_CHECKLEVEL "0" CACHE STRING "The debug sanity check level of CoinUtils (0 if no test)")
set(COIN_COINUTILS_VERBOSITY  "0" CACHE STRING "The debug verbosity level of CoinUtils (0 if no output)")

# Compilation options
option(ENABLE_SHARED_LIBRARIES       "Build libraries as shared libraries" OFF)
option(COIN_COMPILE_STATIC           "Activate the static linking" OFF)
option(COIN_COMPILE_COVERAGE         "Activate the code coverage compilation" OFF)
option(COIN_COMPILE_PROFILE          "Activate the code profiling compilation" OFF)
option(COIN_COMPILE_PROFILE_VALGRIND "Activate the code profiling compilation for valgrind" OFF)
option(COIN_COMPILE_LTO              "Activate the whole program optimization (using LTO)" OFF)
option(COIN_COMPILE_WHOPR            "Activate the whole program optimization (using WHOPR)" OFF)
option(COIN_COMPILE_WARNINGS         "Activate a set of warning options" OFF)
option(COIN_COMPILE_CXX11            "Activate C++11 compilation" OFF)

option(USE_PROCESSOR_EXTENSIONS      "Use sse / mmx / avx extensions during compilation" OFF)

# Threads path
set(COIN_THREADS_INC_PATH "None" CACHE PATH "The path to the PThreads header files")
set(COIN_THREADS_LIB_PATH "None" CACHE PATH "The path to the PThreads library files")

# Set paths of source
set(COIN_DIR "${CMAKE_SOURCE_DIR}" CACHE PATH "The CoinUtils Path")

mark_as_advanced(COIN_THREADS_INC_PATH
                 COIN_THREADS_LIB_PATH
                 ENABLE_SHARED_LIBRARIES
                 COIN_COMPILE_STATIC
                 COIN_COMPILE_COVERAGE
                 COIN_COMPILE_PROFILE
                 COIN_COMPILE_PROFILE_VALGRIND
                 COIN_COMPILE_LTO
                 COIN_COMPILE_WHOPR
                 COIN_COMPILE_WARNINGS
                 COIN_COMPILE_CXX11
                 COIN_DIR
                 USE_PROCESSOR_EXTENSIONS
                 COINUTILS_USE_THREADS
                 COINUTILS_USE_MEMPOOL_OVERRIDE_NEW
                 COINUTILS_USE_MEMPOOL_MAXPOOLED
                 COINUTILS_HAS_ZLIB
                 COINUTILS_HAS_BZLIB
                 COIN_HAS_LAPACK
                 COIN_HAS_BLAS
                 COIN_HAS_MKL
                 COIN_COINUTILS_CHECKLEVEL
                 COIN_COINUTILS_VERBOSITY
                 COIN_USE_FAST_CODE)

# Set paths for binary and library generation inside the build directory:
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
                      ${CMAKE_SOURCE_DIR}/cmake)

include(coin-macros)
include(GetAcInitVersion)

#------------------------------------------------------------
# Detect 64 bits
#------------------------------------------------------------

if (CMAKE_SIZEOF_VOID_P EQUAL 4)
  set(HAVE_64_BIT 0)
else ()
  set(HAVE_64_BIT 1)
endif ()

# Name of package
set(PACKAGE           "CoinUtils")
# Define to the address where bug reports for this package should be sent.
set(PACKAGE_BUGREPORT "bugs@coin-or.org")
# Define to the full name of this package.
set(PACKAGE_NAME      "CoinUtils")
# Define to the full name and version of this package.
set(PACKAGE_STRING    "CoinUtils")
# Define to the one symbol short name of this package.
set(PACKAGE_TARNAME   "coinutils")

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/coinutils_binary" 
      CACHE PATH "CoinUtils install prefix" FORCE)
endif ()

include(CheckCCompilerFlag)

# Desactivate some relocation types for portability
if (UNIX)
  check_c_compiler_flag("-Wa,-mrelax-relocations=no" HAVE_RELAX_RELOC_FLAG)
  if (HAVE_RELAX_RELOC_FLAG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wa,-mrelax-relocations=no")
  endif ()
endif ()

# Check for readline
set(COIN_HAS_READLINE )
find_package(Readline)
if (READLINE_FOUND)
  set(COIN_HAS_READLINE "ON")
  
  include_directories(${READLINE_INCLUDE_DIR})
endif ()

if (USE_PROCESSOR_EXTENSIONS)
  # Check for SSE* and AVX*
  find_package(SSE)
  if (MMX_FOUND OR
      SSE2_FOUND OR SSE3_FOUND OR SSSE3_FOUND OR SSE4_1_FOUND OR SSE4_2_FOUND OR
      AVX_FOUND OR AVX2_FOUND)
    set(CMAKE_C_FLAGS       "${CMAKE_C_FLAGS} ${SSE_COMPILER_FLAGS}")
    set(CMAKE_CXX_FLAGS     "${CMAKE_CXX_FLAGS} ${SSE_COMPILER_FLAGS}")
    set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${SSE_COMPILER_FLAGS}")
  endif ()
endif ()

if ((NOT COIN_DIR STREQUAL CMAKE_SOURCE_DIR) AND (NOT EXISTS ${COIN_DIR}))
  message(FATAL_ERROR "Error: COIN_DIR = ${COIN_DIR} which is not an existing directory")
endif ()

if (NOT COINOR_DIR STREQUAL "None")
  link_directories(${COINOR_DIR})
endif ()

# Check for MKL
if ((CMAKE_CXX_COMPILER_ID STREQUAL "Intel") AND COIN_HAS_MKL)
  find_package(MKL)
  
  if (MKL_FOUND)
    message(status "MKL library found")
  else ()
    message(status "MKL library not found")
  endif ()
  
  # Copy libiomp5md.dll in the build directory
  if (WIN32)
    if (HAVE_64_BIT)
      set(MKL_DLL_DIR ${MKL_ROOT}/bin/intel64)
    else ()
      set(MKL_DLL_DIR ${MKL_ROOT}/bin/ia32)
    endif ()
    
    execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/bin/
                    COMMAND ${CMAKE_COMMAND} -E copy ${MKL_DLL_DIR}/libiomp5md.dll ${CMAKE_BINARY_DIR}/bin/
                    COMMAND ${CMAKE_COMMAND} -E echo "Copying ${MKL_DLL_DIR}/libiomp5md.dll into ${CMAKE_BINARY_DIR}/bin/")
  endif ()
  
  set(COIN_MKL_LIBS "${MKL_LIBRARIES}")
  if (WIN32)
    set(COIN_MKL_LIBS ${COIN_MKL_LIBS} mkl_intel_thread libiomp5md)
  else ()
    set(COIN_MKL_LIBS ${COIN_MKL_LIBS} mkl_gnu_thread gomp dl)
  endif ()
  
  include_directories(${MKL_INCLUDE_DIRS})
  
  if (HAVE_64_BIT)
    link_directories(${MKL_ROOT}/lib/intel64)
  else ()
    link_directories(${MKL_ROOT}/lib/ia32)
  endif ()
endif ()

#-----------------------------------------------------------------------------
# Manage compilation options
#-----------------------------------------------------------------------------

if (UNIX)
  if (COIN_COMPILE_STATIC)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND NOT APPLE)
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static -z muldefs")
    endif ()
  elseif (COIN_COMPILE_COVERAGE)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} --coverage -fno-inline")
    endif ()
    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fprofile-arcs -ftest-coverage -Xclang -coverage-cfg-checksum -Xclang -coverage-no-function-names-in-data -Xclang -coverage-version='408*'")
    endif ()
  elseif (COIN_COMPILE_PROFILE)
    set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -pg")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pg")
  elseif (COIN_COMPILE_PROFILE_VALGRIND)
    set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -p")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -p")
  else ()
    set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -Wno-deprecated -fPIC")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND NOT APPLE)
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -z muldefs")
    endif ()
  endif ()
  
  if (COIN_COMPILE_LTO)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
      set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -lto")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lto")
    endif ()
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      set(CMAKE_C_ARCHIVE_CREATE "<CMAKE_AR> qcs <TARGET> <LINK_FLAGS> <OBJECTS>")
      set(CMAKE_C_ARCHIVE_FINISH "")
      set(CMAKE_AR  "gcc-ar")
      set(CMAKE_NM  "gcc-nm")
      set(CMAKE_RANLIB  "gcc-ranlib")
      
      set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -flto")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -flto")
    endif ()
  elseif (COIN_COMPILE_WHOPR)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      set(CMAKE_CXX_FLAGS        "${CMAKE_CXX_FLAGS} -fwhopr")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fwhopr")
    endif ()
  endif ()
elseif (WIN32)
  if (MSVC)
    if (COIN_COMPILE_LTO OR COIN_COMPILE_WHOPR)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /GL ")
      set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} /GL ")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /LTCG ")
    endif ()
  endif ()
endif ()

#-----------------------------------------------------------------------------
# Manage specific VS flags
#-----------------------------------------------------------------------------

if (MSVC)
  # Avoid Warning C4530 by using the flag /EHsc
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc ")
  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} /EHsc ")
endif ()

#-----------------------------------------------------------------------------
# Manage threads include dir under Windows
#-----------------------------------------------------------------------------

if (WIN32)
  if (NOT COIN_THREADS_INC_PATH STREQUAL "None")
    include_directories(${COIN_THREADS_INC_PATH})
  endif ()
  if (NOT COIN_THREADS_LIB_PATH STREQUAL "None")
    link_directories(${COIN_THREADS_LIB_PATH})
  endif ()
endif ()

#-----------------------------------------------------------------------------
# Detect name mangling convention used between Fortran and C
#-----------------------------------------------------------------------------

if (CMAKE_Fortran_COMPILER)
  enable_language(Fortran)
  
  include(FortranCInterface)
  
  FortranCInterface_HEADER(${CMAKE_BINARY_DIR}/F77Mangle.h
                           MACRO_NAMESPACE "F77_"
                           SYMBOL_NAMESPACE "F77_"
                           SYMBOLS mysub mymod:my_sub)
  
  file(STRINGS ${CMAKE_BINARY_DIR}/F77Mangle.h CONTENTS REGEX "F77_GLOBAL\\(.*,.*\\) +(.*)")
  string(REGEX MATCH "F77_GLOBAL\\(.*,.*\\) +(.*)" RESULT ${CONTENTS})
  set(F77_FUNC "F77_FUNC(name,NAME) ${CMAKE_MATCH_1}")
  
  file(STRINGS ${CMAKE_BINARY_DIR}/F77Mangle.h CONTENTS REGEX "F77_GLOBAL_\\(.*,.*\\) +(.*)")
  string(REGEX MATCH "F77_GLOBAL_\\(.*,.*\\) +(.*)" RESULT ${CONTENTS})
  set(F77_FUNC_ "F77_FUNC_(name,NAME) ${CMAKE_MATCH_1}")
else ()
  set(F77_FUNC  "F77_FUNC(name,NAME)  name##_")
  set(F77_FUNC_ "F77_FUNC_(name,NAME) name##__")
endif ()

set(F77_DUMMY_MAIN "" CACHE STRING "Define to dummy 'main' function (if any) required to link to the Fortran libraries.")
set(FC_DUMMY_MAIN  "" CACHE STRING "Define to dummy 'main' function (if any) required to link to the Fortran libraries.")
option(FC_DUMMY_MAIN_EQ_F77 "Define if F77 and FC dummy 'main' functions are identical." OFF)

if (FC_DUMMY_MAIN_EQ_F77)
  set(FC_DUMMY_MAIN "${F77_DUMMY_MAIN}")
endif ()

mark_as_advanced(F77_FUNC
                 F77_FUNC_
                 F77_DUMMY_MAIN
                 FC_DUMMY_MAIN
                 FC_DUMMY_MAIN_EQ_F77)

# End detect fortran mangling

if (NOT WIN32 AND COIN_COMPILE_WARNINGS)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated -fPIC ")
  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wno-deprecated -fPIC ")
  # Try to locate unitizalized variables
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wshadow -Wuninitialized ")
  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wall -Wshadow -Wmissing-prototypes -Wuninitialized ")
endif ()

# Manage coverage via lcov automatically

if (COIN_COMPILE_COVERAGE)
  if (WIN32)
    message(FATAL_ERROR "Error: code coverage analysis is only available under Linux for now.")
  endif ()
  
  find_program(GCOV_PATH gcov)
  find_program(LCOV_PATH lcov)
  find_program(GENHTML_PATH genhtml)

  if (NOT GCOV_PATH)
    message(FATAL_ERROR "gcov not found! Please install lcov and gcov. Aborting...")
  endif ()
  
  if (NOT LCOV_PATH)
    message(FATAL_ERROR "lcov not found! Please install lcov and gcov. Aborting...")
  endif ()
  
  if (NOT GENHTML_PATH)
    message(FATAL_ERROR "genhtml not found! Please install lcov and gcov. Aborting...")
  endif ()
  
  # /!\ FAILURE IF AT LEAST ONE TEST FAILS
  # Capturing lcov counters and generating report
  add_custom_target(coverage
                    COMMAND ${LCOV_PATH} --directory . --zerocounters
                    COMMAND ${CMAKE_COMMAND} -E chdir ${CMAKE_BINARY_DIR} ${CMAKE_CTEST_COMMAND} -LE "(LONG|FAIL)" -L SAMPLE
                    COMMAND ${LCOV_PATH} --capture --directory . --output-file ${CMAKE_BINARY_DIR}/coverage.info
                    COMMAND ${LCOV_PATH} --remove ${CMAKE_BINARY_DIR}/coverage.info '/usr/*' '${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR}/*' --output-file ${CMAKE_BINARY_DIR}/coverage.info.cleaned
                    COMMAND ${GENHTML_PATH} -o ${CMAKE_BINARY_DIR}/coverage ${CMAKE_BINARY_DIR}/coverage.info.cleaned
                    COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_BINARY_DIR}/coverage.info ${CMAKE_BINARY_DIR}/coverage.info.cleaned
                    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
                    COMMENT "Resetting code coverage counters to zero.
Processing code coverage counters and generating report.
You can zip the directory ${CMAKE_BINARY_DIR}/coverage and upload the content to a web server.")
endif ()

# End of coverage

# Directories where to find the source code

set(BuildTools_DIR    ${COIN_DIR}/BuildTools)
set(CoinUtils_DIR     ${COIN_DIR}/CoinUtils)
set(CoinUtilsTest_DIR ${CoinUtils_DIR}/test)

include(CheckIncludeFileCXX)
include(CheckIncludeFile)
include(VA_COPY)

check_include_file("assert.h"    HAVE_ASSERT_H)
check_include_file("bzlib.h"     HAVE_BZLIB_H)
check_include_file("ctype.h"     HAVE_CTYPE_H)
check_include_file("dlfcn.h"     HAVE_DLFCN_H)
check_include_file("endian.h"    HAVE_ENDIAN_H)
check_include_file("float.h"     HAVE_FLOAT_H)
check_include_file("ieeefp.h"    HAVE_IEEEFP_H)
check_include_file("inttypes.h"  HAVE_INTTYPES_H)
check_include_file("math.h"      HAVE_MATH_H)
check_include_file("memory.h"    HAVE_MEMORY_H)
check_include_file("readline/readline.h" HAVE_READLINE_READLINE_H)
check_include_file("stdint.h"    HAVE_STDINT_H)
check_include_file("stdlib.h"    HAVE_STDLIB_H)
check_include_file("stdio.h"     HAVE_STDIO_H)
check_include_file("stdarg.h"    HAVE_STDARG_H)
check_include_file("stddef.h"    HAVE_STDDEF_H)
check_include_file("strings.h"   HAVE_STRINGS_H)
check_include_file("string.h"    HAVE_STRING_H)
check_include_file("sys/stat.h"  HAVE_SYS_STAT_H)
check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
check_include_file("time.h"      HAVE_TIME_H)
check_include_file("unistd.h"    HAVE_UNISTD_H)
check_include_file("windows.h"   HAVE_WINDOWS_H)
check_include_file("zlib.h"      HAVE_ZLIB_H)
check_include_file_cxx("cctype"    HAVE_CCTYPE)
check_include_file_cxx("cmath"     HAVE_CMATH)
check_include_file_cxx("cieeefp"   HAVE_CIEEEFP)
check_include_file_cxx("cfloat"    HAVE_CFLOAT)
check_include_file_cxx("cinttypes" HAVE_CINTTYPES)
check_include_file_cxx("cassert"   HAVE_CASSERT)
check_include_file_cxx("cstdio"    HAVE_CSTDIO)
check_include_file_cxx("cstdlib"   HAVE_CSTDLIB)
check_include_file_cxx("cstdarg"   HAVE_CSTDARG)
check_include_file_cxx("cstddef"   HAVE_CSTDDEF)
check_include_file_cxx("cstring"   HAVE_CSTRING)
check_include_file_cxx("ctime"     HAVE_CTIME)

string(SUBSTRING ${CMAKE_SHARED_LIBRARY_SUFFIX} 1 -1 SHAREDLIBEXT)

include(CheckCXXCompilerFlag)

if (COIN_COMPILE_CXX11 AND NOT WIN32)
  # VS2013 activate C++11 by default
  check_cxx_compiler_flag(-std=c++11 HAVE_STD_CPP11_FLAG)
  if (HAVE_STD_CPP11_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  else ()
    check_cxx_compiler_flag(-std=c++0x HAVE_STD_CPP0X_FLAG)
    if (HAVE_STD_CPP0X_FLAG)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    endif ()
  endif ()
endif ()

check_cxx_compiler_flag(-Qunused-arguments HAVE_QUNUSED_ARGUMENTS)
if (HAVE_QUNUSED_ARGUMENTS)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Qunused-arguments")
endif ()

include(CheckTypeSize)

check_type_size("int64_t"   SIZEOF_INT64_T)
check_type_size("int"       SIZEOF_INT)
check_type_size("int *"     SIZEOF_INT_P)
check_type_size("long"      SIZEOF_LONG)
check_type_size("long long" SIZEOF_LONG_LONG)

set(COIN_INTPTR_T "int")
set(COIN_INT64_T  "int")
set(COIN_UINT64_T "unsigned int")

if (SIZEOF_LONG_LONG EQUAL "8")
  set(COIN_INTPTR_T "long long")
  set(COIN_INT64_T  "long long")
  set(COIN_UINT64_T "unsigned long long")
elseif (SIZEOF_LONG EQUAL "8")
  set(COIN_INTPTR_T "long")
  set(COIN_INT64_T  "long")
  set(COIN_UINT64_T "unsigned long")
elseif (SIZEOF_INT EQUAL "8")
  set(COIN_INTPTR_T "int")
  set(COIN_INT64_T  "int")
  set(COIN_UINT64_T "unsigned int")
endif ()

include(AC_HEADER_STDC)

if (WIN32)
  # From Linux endian.h
  #define __LITTLE_ENDIAN 1234
  #define __BIG_ENDIAN    4321
  # Under Windows __BYTE_ORDER is not defined and endian.h doesn't exist
  add_definitions(-D__BYTE_ORDER=1234)
endif ()

if (CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
  add_definitions(-DINTEL_COMPILER)
endif ()

include(CheckSymbolExists)
include(CheckFunctionExists)

check_function_exists(memcpy USE_MEMCPY)

if (USE_MEMCPY)
  add_definitions(-DUSE_MEMCPY)
endif ()

if (HAVE_CMATH)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "cmath")
endif ()
if (HAVE_MATH_H)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "math.h")
endif ()
if (HAVE_CFLOAT)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "cfloat")
endif ()
if (HAVE_FLOAT_H)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "float.h")
endif ()
if (HAVE_CIEEEFP)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "cieeefp")
endif ()
if (HAVE_IEEEFP_H)
  set(TEST_INCLUDE_LIST ${TEST_INCLUDE_LIST}
                        "ieeefp.h")
endif ()

set(CMAKE_REQUIRED_LIBRARIES m)
check_symbol_exists(isfinite "${TEST_INCLUDE_LIST}" COIN_C_FINITE_R)
if (COIN_C_FINITE_R)
  set(COIN_C_FINITE isfinite)
endif ()
if (NOT COIN_C_FINITE_R)
  set(CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(_finite "${TEST_INCLUDE_LIST}" COIN_C_FINITE_R)
  if (COIN_C_FINITE_R)
    set(COIN_C_FINITE _finite)
  endif ()
endif ()
if (NOT COIN_C_FINITE_R)
  set(CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(finite "${TEST_INCLUDE_LIST}" COIN_C_FINITE_R)
  if (COIN_C_FINITE_R)
    set(COIN_C_FINITE ifinite)
  endif ()
endif ()

set(CMAKE_REQUIRED_LIBRARIES m)
check_symbol_exists(isnan "${TEST_INCLUDE_LIST}" COIN_C_ISNAN_R)
if (COIN_C_ISNAN_R)
  set(COIN_C_NAN isnan)
endif ()
if (NOT COIN_C_ISNAN_R)
  set(CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(std::isnan "${TEST_INCLUDE_LIST}" COIN_C_ISNAN_R)
  if (COIN_C_NAN_R)
    set(COIN_C_NAN std::isnan)
  endif ()
endif ()
if (NOT COIN_C_ISNAN_R)
  set(CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(_isnan "${TEST_INCLUDE_LIST}" COIN_C_ISNAN_R)
  if (COIN_C_NAN_R)
    set(COIN_C_NAN _isnan)
  endif ()
endif ()
if (NOT COIN_C_ISNAN_R)
  set(CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(isnand "${TEST_INCLUDE_LIST}" COIN_C_ISNAN_R)
  if (COIN_C_NAN_R)
    set(COIN_C_NAN isnand)
  endif ()
endif ()

check_symbol_exists(clock_gettime sys/time.h HAVE_CLOCK_GETTIME)
check_symbol_exists(gettimeofday  sys/time.h HAVE_GETTIMEOFDAY)

if (COIN_USE_FAST_CODE)
  add_definitions(-DCOIN_FAST_CODE -DCOIN_USE_RESTRICT -DCOIN_NOTEST_DUPLICATE)
  
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fomit-frame-pointer ")
  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -fomit-frame-pointer ")
  
  if (CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-math-errno -fp-trap=none ")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -fno-math-errno -fp-trap=none ")
  endif ()

  if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffast-math -fno-math-errno -fno-trapping-math ")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -ffast-math -fno-math-errno -fno-trapping-math ")
  endif ()
endif ()

if (COINUTILS_USE_THREADS)
  set(COINUTILS_PTHREADS 1)
endif ()
if (COINUTILS_USE_MEMPOOL_OVERRIDE_NEW)
  set(COINUTILS_MEMPOOL_OVERRIDE_NEW ON)
endif ()
if (COINUTILS_USE_MEMPOOL_MAXPOOLED)
  set(COINUTILS_MEMPOOL_MAXPOOLED 4096)
else ()
  set(COINUTILS_MEMPOOL_MAXPOOLED -1)
endif ()

add_definitions(-DHAVE_CONFIG_H)

#-----------------------------------------------
# Doxygen documentation
#-----------------------------------------------

set(doc_dir doc)
find_package(Doxygen)

if (DOXYGEN AND ENABLE_DOXYGEN)
  configure_file(${CMAKE_SOURCE_DIR}/doxydoc/doxygen.conf.in           ${CMAKE_SOURCE_DIR}/doxydoc/doxygen.conf)
  configure_file(${CMAKE_SOURCE_DIR}/CoinUtils/doxydoc/doxygen.conf.in ${CMAKE_SOURCE_DIR}/CoinUtils/doxydoc/doxygen.conf)

  add_custom_target(docs 
                    ${DOXYGEN_EXECUTABLE} ${CMAKE_SOURCE_DIR}/doxydoc/doxygen.conf
                    ${DOXYGEN_EXECUTABLE} ${CMAKE_SOURCE_DIR}/CoinUtils/doxydoc/doxygen.conf)
endif ()

subdirs(ThirdParty)

include_directories(${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR}/include)
link_directories(${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR}/lib)

if (COIN_ENABLE_DOWNLOAD_ZLIB)
  set(COINUTILS_HAS_ZLIB ON)
endif ()
if (COIN_ENABLE_DOWNLOAD_BZLIB)
  set(COINUTILS_HAS_BZLIB ON)
endif ()

subdirs(CoinUtils)

#
# Install dll dependencies
#

if (WIN32)
  install(DIRECTORY ${CMAKE_BINARY_DIR}/bin
          DESTINATION bin
          PATTERN "*.dll")
endif ()

#
# Packaging
#

set(CPACK_PACKAGE_NAME      "${PACKAGE_NAME}")
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${COINUTILS_VERSION}-${CMAKE_SYSTEM_NAME}")

message(STATUS "Package filename: ${CPACK_PACKAGE_FILE_NAME}")

set(CPACK_PACKAGE_INSTALL_DIRECTORY "${PACKAGE_NAME}")
set(CPACK_PACKAGE_VERSION "${COINUTILS_VERSION}")

set(CPACK_PACKAGE_RELOCATABLE TRUE)

if (WIN32)
  set(CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}")
  set(CPACK_MONOLITHIC_INSTALL ON)
  set(CPACK_NSIS_CONTACT "${PACKAGE_BUGREPORT}")
  set(CPACK_NSIS_MODIFY_PATH ON)
  set(CPACK_NSIS_PACKAGE_NAME "CoinUtils ${COINUTILS_VERSION}")
elseif (APPLE)
  set(CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
  set(CPACK_PACKAGE_DEFAULT_LOCATION "/opt/${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGING_INSTALL_PREFIX "/")
else (WIN32)
  set(CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
  set(CPACK_PACKAGE_DEFAULT_LOCATION "/opt/${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGING_INSTALL_PREFIX "/")
  
  set(CPACK_DEBIAN_PACKAGE_SECTION "Libraries")
  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${PACKAGE_BUGREPORT}")
  
  set(CPACK_RPM_COMPONENT_INSTALL ON)
  set(CPACK_RPM_PACKAGE_RELOCATABLE ON)
  set(CPACK_RPM_PACKAGE_LICENSE "Copyrighted")
  set(CPACK_RPM_PACKAGE_GROUP "Development/Libraries")
  set(CPACK_RPM_PACKAGE_URL "${PACKAGE_URL}")
  set(CPACK_RPM_PACKAGE_SUMMARY "CoinUtils is a linear algebra library from CoinOR.")
  set(CPACK_RPM_PACKAGE_DESCRIPTION 
    "The CoinUtils suite includes:

    * A set of linear algebra routines.
"
    )
endif ()

include(CPack)
include(CTest)

###########################
#                         #
# Sum-up of configuration #
#                         #
###########################

# available colors: {u,b,bg, } black, red, green, yellow, blue, purple, cyan, white, reset

color_message("${color_green}************************")
color_message("*                      *")
color_message("* Configuration sum-up *")
color_message("*                      *")
color_message("************************")
color_message("${color_reset} ")

color_message("${color_cyan}Installation directory:${color_reset} ${CMAKE_INSTALL_PREFIX}")
color_message("${color_cyan}Build type:${color_reset} ${CMAKE_BUILD_TYPE}")

get_directory_property(ALL_INCLUDES INCLUDE_DIRECTORIES)
get_directory_property(ALL_LINKS    LINK_DIRECTORIES)
get_directory_property(ALL_DEFS     COMPILE_DEFINITIONS)

message(STATUS "${color_cyan}Include directories:${color_reset}     ${ALL_INCLUDES}")
message(STATUS "${color_cyan}Link directories:${color_reset}        ${ALL_LINKS}")
message(STATUS "${color_cyan}Compilation definitions:${color_reset} ${ALL_DEFS}")
