cmake_minimum_required(VERSION 2.8)

project(Ipopt_Thirdparty)

# CMakeLists.txt designed for 
# downloading/configuring/building/installing
# Thirdparty dependencies

option(COIN_ENABLE_DOWNLOAD_MUMPS        "Enable the download / compilation of Mumps" OFF)
option(COIN_ENABLE_DOWNLOAD_METIS        "Enable the download / compilation of Metis" OFF)
option(COIN_ENABLE_DOWNLOAD_ASL          "Enable the download / compilation of Ampl Solver Library" OFF)
option(COIN_ENABLE_DOWNLOAD_MINGW_LAPACK "Enable the download / compilation of MinGW compiled Blas / Lapack" OFF)
option(COIN_ENABLE_DOWNLOAD_LAPACK       "Enable the download / compilation of Blas / Lapack" OFF)
option(COIN_ENABLE_DOWNLOAD_CLAPACK      "Enable the download / compilation of CBlas / CLapack" OFF)
option(COIN_ENABLE_COMPILE_HSL           "Enable the compilation of HSL" OFF)

set(COIN_HSL_PATH "None" CACHE PATH "The HSL source Path")

# Prevent the "make clean" from cleaning this directory
set_directory_properties(PROPERTIES CLEAN_NO_CUSTOM TRUE)

include(ExternalProject)

set(EP_InstallDir ${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR})

if (NOT EXISTS ${EP_InstallDir})
  make_directory(${EP_InstallDir})
endif ()
if (NOT EXISTS ${EP_InstallDir}/bin)
  make_directory(${EP_InstallDir}/bin)
endif ()
if (NOT EXISTS ${EP_InstallDir}/lib)
  make_directory(${EP_InstallDir}/lib)
endif ()
if (NOT EXISTS ${EP_InstallDir}/include)
  make_directory(${EP_InstallDir}/include)
endif ()

# #################################################
# ===>    CLapack
# #################################################

if (COIN_ENABLE_DOWNLOAD_CLAPACK)
  set(CLAPACK_VERSION "3.2.1")
  
  if (UNIX)
    if (NOT HAVE_64_BIT)
      set(CLAPACK_C_FLAGS      "-fPIC")
      set(CLAPACK_LINKER_FLAGS "-fPIC")
    endif ()
    
    if (CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
      if (RM_COMPILE_LTO)
        set(CLAPACK_C_FLAGS      "${CLAPACK_C_FLAGS} -flto")
        set(CLAPACK_LINKER_FLAGS "${CLAPACK_LINKER_FLAGS} -flto")
      endif ()
      if (RM_COMPILE_WHOPR)
        set(CLAPACK_C_FLAGS      "${CLAPACK_C_FLAGS} -flto")
        set(CLAPACK_LINKER_FLAGS "${CLAPACK_LINKER_FLAGS} -flto")
      endif ()
    endif ()
  endif ()
  
  if (WIN32)
    if (MSVC)
      if (RM_COMPILE_LTO OR RM_COMPILE_WHOPR)
        set(CLAPACK_C_FLAGS      "/GL")
        set(CLAPACK_LINKER_FLAGS "/LTGC")
      endif ()
    endif ()
  endif ()
  
  set(COIN_HAS_LAPACK ON CACHE BOOL "Enable the Lapack support of CoinUtils" FORCE)
  set(COIN_HAS_BLAS   ON CACHE BOOL "Enable the Blas support of CoinUtils"   FORCE)
  
  set(CLAPACK_URL        "http://netlib.sandia.gov/clapack/clapack-${CLAPACK_VERSION}-CMAKE.tgz")
  set(CLAPACK_InstallDir "${EP_InstallDir}/CLapack-${CLAPACK_VERSION}/${CMAKE_CFG_INTDIR}/")
  
  if (WIN32 AND NOT MINGW)
    ExternalProject_Add(EP_Lapack
                        PREFIX            ${CLAPACK_InstallDir}
                        URL               ${CLAPACK_URL}
                        UPDATE_COMMAND    ""
                        PATCH_COMMAND     ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/f2c.h ${CLAPACK_InstallDir}/src/EP_Lapack/INCLUDE/
                                       && ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/f2c.h ${CLAPACK_InstallDir}/src/EP_Lapack/F2CLIBS/libf2c/
                        CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${CLAPACK_InstallDir}/src/EP_Lapack/build
                                       && ${CMAKE_COMMAND} -E chdir ${CLAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DCMAKE_C_FLAGS:String=${CLAPACK_C_FLAGS} -DCMAKE_EXE_LINKER_FLAGS:String=${CLAPACK_LINKER_FLAGS} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} -G ${CMAKE_GENERATOR} ..
                        BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${CLAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_MAKE_PROGRAM}
                        INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/SRC/lapack.lib            ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/F2CLIBS/libf2c/libf2c.lib ${EP_InstallDir}/lib/f2c.lib
                                       && ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/BLAS/SRC/blas.lib         ${EP_InstallDir}/lib
                        )
  elseif (UNIX)
    ExternalProject_Add(EP_Lapack
                        PREFIX            ${CLAPACK_InstallDir}
                        URL               ${CLAPACK_URL}
                        UPDATE_COMMAND    ""
                        PATCH_COMMAND     ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/f2c.h ${CLAPACK_InstallDir}/src/EP_Lapack/INCLUDE/
                                       && ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/f2c.h ${CLAPACK_InstallDir}/src/EP_Lapack/F2CLIBS/libf2c/
                        CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${CLAPACK_InstallDir}/src/EP_Lapack/build
                                       && ${CMAKE_COMMAND} -E chdir ${CLAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DCMAKE_C_FLAGS:String=${CLAPACK_C_FLAGS} -DCMAKE_EXE_LINKER_FLAGS:String=${CLAPACK_LINKER_FLAGS} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} ..
                        BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${CLAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_MAKE_PROGRAM}
                        INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/SRC/liblapack.a         ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/F2CLIBS/libf2c/libf2c.a ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${CLAPACK_InstallDir}/src/EP_Lapack/build/BLAS/SRC/libblas.a      ${EP_InstallDir}/lib
                        )
  endif ()
endif ()

# #################################################
# <===    CLapack
# #################################################

# #################################################
# ===>    Lapack
# #################################################

if (COIN_ENABLE_DOWNLOAD_LAPACK)
  set(LAPACK_VERSION "3.7.1")
  
  enable_language(Fortran)

  if (UNIX)
    if (NOT HAVE_64_BIT)
      set(LAPACK_C_FLAGS      "-fPIC")
      set(LAPACK_LINKER_FLAGS "-fPIC")
    endif ()
  endif ()
  
  set(COIN_HAS_LAPACK ON CACHE BOOL "Enable the Lapack support of CoinUtils" FORCE)
  set(COIN_HAS_BLAS   ON CACHE BOOL "Enable the Blas support of CoinUtils"   FORCE)
  
  set(LAPACK_URL        "http://www.netlib.org/lapack/lapack-${LAPACK_VERSION}.tgz")
  set(LAPACK_InstallDir "${EP_InstallDir}/Lapack-${LAPACK_VERSION}/${CMAKE_CFG_INTDIR}/")
  
  if (WIN32 AND NOT MINGW)
    ExternalProject_Add(EP_Lapack
                        PREFIX            ${LAPACK_InstallDir}
                        URL               ${LAPACK_URL}
                        UPDATE_COMMAND    ""
                        CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${LAPACK_InstallDir}/src/EP_Lapack/build
                                       && ${CMAKE_COMMAND} -E chdir ${LAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DBUILD_DEPRECATED:Bool=ON -DLAPACKE:Bool=ON -DCMAKE_C_FLAGS:String=${CLAPACK_C_FLAGS} -DCMAKE_EXE_LINKER_FLAGS:String=${LAPACK_LINKER_FLAGS} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} -G ${CMAKE_GENERATOR} ..
                        BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${LAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_MAKE_PROGRAM}
                        INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/lapack.lib  ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/lapacke.lib ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/blas.lib    ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/tmglib.lib  ${EP_InstallDir}/lib
                        )
  elseif (UNIX)
    ExternalProject_Add(EP_Lapack
                        PREFIX            ${LAPACK_InstallDir}
                        URL               ${LAPACK_URL}
                        UPDATE_COMMAND    ""
                        CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${LAPACK_InstallDir}/src/EP_Lapack/build
                                       && ${CMAKE_COMMAND} -E chdir ${LAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DBUILD_DEPRECATED:Bool=ON -DLAPACKE:Bool=ON -DCMAKE_C_FLAGS:String=${CLAPACK_C_FLAGS} -DCMAKE_EXE_LINKER_FLAGS:String=${LAPACK_LINKER_FLAGS} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} ..
                        BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${LAPACK_InstallDir}/src/EP_Lapack/build ${CMAKE_MAKE_PROGRAM}
                        INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/liblapack.a  ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/liblapacke.a ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/libblas.a    ${EP_InstallDir}/lib
                                       && ${CMAKE_COMMAND} -E copy ${LAPACK_InstallDir}/src/EP_Lapack/build/lib/libtmglib.a  ${EP_InstallDir}/lib
                        )
  endif ()
endif ()

if (COIN_ENABLE_FIND_LAPACK)
  find_package(LAPACK)
  find_package(BLAS)
  
  if (LAPACK_FOUND AND BLAS_FOUND)
    set(COIN_HAS_LAPACK ON CACHE BOOL "Enable the Lapack support of CoinUtils" FORCE)
    set(COIN_HAS_BLAS   ON CACHE BOOL "Enable the Blas support of CoinUtils"   FORCE)
  else ()
    set(COIN_HAS_LAPACK OFF CACHE BOOL "Enable the Lapack support of CoinUtils" FORCE)
    set(COIN_HAS_BLAS   OFF CACHE BOOL "Enable the Blas support of CoinUtils"   FORCE)
  endif ()
endif ()

# #################################################
# <===    Lapack
# #################################################

# #################################################
# ===>    MinGW Lapack
# #################################################

if (COIN_ENABLE_DOWNLOAD_MINGW_LAPACK)
  set(COIN_MINGW_LAPACK_ZIP_FILE "None" CACHE FILEPATH "Path to the zip file containing MinGW Lapack")
  
  set(COIN_HAS_LAPACK ON CACHE BOOL "Enable the Lapack support of CoinUtils" FORCE)
  set(COIN_HAS_BLAS   ON CACHE BOOL "Enable the Blas support of CoinUtils"   FORCE)

  ExternalProject_Add(EP_Lapack
                      PREFIX            ${EP_InstallDir}/MinGW-Lapack
                      URL               ${COIN_MINGW_LAPACK_ZIP_FILE}
                      UPDATE_COMMAND    ""
                      CONFIGURE_COMMAND ""
                      BUILD_COMMAND     ""
                      INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/lib/liblapack.lib       ${EP_InstallDir}/lib/lapack.lib
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/lib/libblas.lib         ${EP_InstallDir}/lib/blas.lib
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/lib/libtmglib.lib       ${EP_InstallDir}/lib/tmglib.lib
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/liblapack.dll       ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libblas.dll         ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libtmglib.dll       ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libgcc_s_seh-1.dll  ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libgfortran-3.dll   ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libquadmath-0.dll   ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libwinpthread-1.dll ${EP_InstallDir}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/liblapack.dll       ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libblas.dll         ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libtmglib.dll       ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libgcc_s_seh-1.dll  ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libgfortran-3.dll   ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libquadmath-0.dll   ${CMAKE_BINARY_DIR}/bin
                                     && ${CMAKE_COMMAND} -E copy ${EP_InstallDir}/MinGW-Lapack/src/EP_Lapack/bin/libwinpthread-1.dll ${CMAKE_BINARY_DIR}/bin
                      )
endif ()

# #################################################
# <===    MinGW Lapack
# #################################################

# #################################################
# ===> Metis compilation
# #################################################

if (COIN_ENABLE_DOWNLOAD_METIS)
  set(METIS_VERSION "4.0.3")
  
  if (NOT EXISTS ${EP_InstallDir}/include/metis)
    make_directory(${EP_InstallDir}/include/metis)
  endif ()
  
  set(METIS_URL        "http://glaros.dtc.umn.edu/gkhome/fetch/sw/metis/OLD/metis-${METIS_VERSION}.tar.gz")
  set(METIS_InstallDir "${EP_InstallDir}/Metis-${METIS_VERSION}/${CMAKE_CFG_INTDIR}/")
  
  ExternalProject_Add(EP_Metis
                      PREFIX            ${METIS_InstallDir}
                      URL               ${METIS_URL}
                      UPDATE_COMMAND    ""
                      PATCH_COMMAND     ""
                      CONFIGURE_COMMAND ""
                      BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${METIS_InstallDir}/src/EP_Metis/ ${CMAKE_MAKE_PROGRAM}
                      INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/libmetis.a   ${EP_InstallDir}/lib
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/macros.h ${EP_InstallDir}/include/metis/
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/struct.h ${EP_InstallDir}/include/metis/
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/proto.h  ${EP_InstallDir}/include/metis/
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/metis.h  ${EP_InstallDir}/include/metis/
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/defs.h   ${EP_InstallDir}/include/metis/
                                     && ${CMAKE_COMMAND} -E copy ${METIS_InstallDir}/src/EP_Metis/Lib/rename.h ${EP_InstallDir}/include/metis/
                      )
endif ()

# #################################################
# <=== Metis compilation
# #################################################

# #################################################
# ===> Mumps compilation
# #################################################

if (COIN_ENABLE_DOWNLOAD_MUMPS)
  set(MUMPS_VERSION "4.10.0")
  set(COIN_HAS_MUMPS ON CACHE BOOL "Clp uses the MUMPS linear solver" FORCE)
  
  if (COIN_ENABLE_DOWNLOAD_METIS)
    set(MUMPS_METIS_INC_PATH "${EP_InstallDir}/include/metis" CACHE PATH "The METIS library include Path" FORCE)
    set(MUMPS_METIS_LIB_PATH "${EP_InstallDir}/lib"           CACHE PATH "The METIS library library Path" FORCE)
  else ()
    set(MUMPS_METIS_INC_PATH "None" CACHE PATH "The METIS library include Path" FORCE)
    set(MUMPS_METIS_LIB_PATH "None" CACHE PATH "The METIS library library Path" FORCE)
  endif ()

  set(MUMPS_DEPENDS "")
  
  if (COIN_ENABLE_DOWNLOAD_CLAPACK OR COIN_ENABLE_DOWNLOAD_LAPACK)
    set(MUMPS_DEPENDS ${MUMPS_DEPENDS} EP_Lapack)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_METIS)
    set(MUMPS_DEPENDS ${MUMPS_DEPENDS} EP_Metis)
  endif ()
  
  set(MUMPS_URL        "http://mumps.enseeiht.fr/MUMPS_${MUMPS_VERSION}.tar.gz")
  set(MUMPS_InstallDir "${EP_InstallDir}/Mumps-${MUMPS_VERSION}/${CMAKE_CFG_INTDIR}/")
  
  ExternalProject_Add(EP_Mumps
                      PREFIX            ${MUMPS_InstallDir}
                      DEPENDS           ${MUMPS_DEPENDS}
                      URL               ${MUMPS_URL}
                      UPDATE_COMMAND    ""
                      PATCH_COMMAND     ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/CMakeLists_mumps.txt ${MUMPS_InstallDir}/src/EP_Mumps/CMakeLists.txt
                      CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${MUMPS_InstallDir}/src/EP_Mumps/build
                                     && ${CMAKE_COMMAND} -E chdir ${MUMPS_InstallDir}/src/EP_Mumps/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DMUMPS_USE_LIBSEQ=ON -DMUMPS_USE_METIS=${COIN_ENABLE_DOWNLOAD_METIS} -DMUMPS_LAPACK_LIB_PATH:Path=${EP_InstallDir}/lib -DMUMPS_METIS_INC_PATH:Path=${MUMPS_METIS_INC_PATH} -DMUMPS_METIS_LIB_PATH:Path=${MUMPS_METIS_LIB_PATH} -DMUMPS_INSTALL_COIN=ON -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} -G ${CMAKE_GENERATOR} ..
                      BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${MUMPS_InstallDir}/src/EP_Mumps/build ${CMAKE_MAKE_PROGRAM}
                      INSTALL_COMMAND   ${CMAKE_COMMAND} -E chdir ${MUMPS_InstallDir}/src/EP_Mumps/build ${CMAKE_MAKE_PROGRAM} install
                      )
endif ()

# #################################################
# <=== Mumps compilation
# #################################################

# #################################################
# ===> ASL compilation
# #################################################

if (COIN_ENABLE_DOWNLOAD_ASL)
  set(ASL_VERSION "1.3.0")
  set(COIN_HAS_ASL ON CACHE BOOL "Enable the ASL support" FORCE)
  
  set(ASL_URL        "https://github.com/ampl/mp/archive/${ASL_VERSION}.tar.gz")
  set(ASL_InstallDir "${EP_InstallDir}/ASL-${ASL_VERSION}/${CMAKE_CFG_INTDIR}/")
  
  ExternalProject_Add(EP_ASL
                      PREFIX            ${ASL_InstallDir}
                      URL               ${ASL_URL}
                      UPDATE_COMMAND    ""
                      PATCH_COMMAND     ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/ASL/AMPLConfig.cmake.in ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/
                                     && ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/ASL/CMakeLists.txt      ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/
                                     && ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/ASL/FindDL.cmake        ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/
                                     && ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/ThirdParty/ASL/details.c0.cmake.in ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/
                      CONFIGURE_COMMAND ${CMAKE_COMMAND} -E make_directory ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/build
                                     && ${CMAKE_COMMAND} -E chdir ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/build ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${EP_InstallDir} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} -G ${CMAKE_GENERATOR} ..
                      BUILD_COMMAND     ${CMAKE_COMMAND} -E chdir ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/build ${CMAKE_MAKE_PROGRAM}
                      INSTALL_COMMAND   ${CMAKE_COMMAND} -E chdir ${ASL_InstallDir}/src/EP_ASL/src/asl/solvers/build ${CMAKE_MAKE_PROGRAM} install
                      )
endif ()

# #################################################
# <=== ASL compilation
# #################################################

# #################################################
# ===>    HSL
# #################################################

if (COIN_ENABLE_COMPILE_HSL AND NOT COIN_HSL_PATH STREQUAL "None")
  set(COIN_HAS_HSL ON CACHE BOOL "Enable the HSL support" FORCE)

  set(COINHSL_HAS_MA27  ON)
  set(COINHSL_HAS_MA28  ON)
  set(COINHSL_HAS_MC19  ON)
  set(COINHSL_HAS_MA57  OFF)
  set(COINHSL_HAS_MA86  OFF)
  set(COINHSL_HAS_METIS OFF)
  set(COINHSL_HAS_MA77  OFF)
  set(COINHSL_HSL2013   OFF)
  set(COINHSL_HAS_MC68  OFF)
  set(COINHSL_HAS_MA97  OFF)
  
  set(COINHSL_VERSION "2014.01.17")
  set(COINHSL_VERSION_MAJOR   2014)
  set(COINHSL_VERSION_MINOR   1)
  set(COINHSL_VERSION_RELEASE 17)
  
  if (COINHSL_HAS_MA57)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA57)
  endif ()
  if (COINHSL_HAS_MA28)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA28)
  endif ()
  if (COINHSL_HAS_MA86)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA86)
  endif ()
  if (COINHSL_HAS_METIS)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_METIS)
  endif ()
  if (COINHSL_HAS_MC19)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MC19)
  endif ()
  if (COINHSL_HAS_MA77)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA77)
  endif ()
  if (COINHSL_HSL2013)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HSL2013)
  endif ()
  if (COINHSL_HAS_MC68)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MC68)
  endif ()
  if (COINHSL_HAS_MA97)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA97)
  endif ()
  if (COINHSL_HAS_MA27)
    set(COIN_HAS_HSL ON)
    add_definitions(-DCOINHSL_HAS_MA27)
  endif ()
  
  configure_file(${CMAKE_SOURCE_DIR}/Ipopt/include/config_coinhsl.h.in ${CMAKE_BINARY_DIR}/Ipopt/include/config_coinhsl.h)
  
  set(HSL_SRCS ${COIN_HSL_PATH}/ma28/ma28d.f
               ${COIN_HSL_PATH}/ma27/ma27d.f
               ${COIN_HSL_PATH}/common/deps.f
               ${COIN_HSL_PATH}/mc19/mc19d.f)
  
  add_library(hsl SHARED ${HSL_SRCS})
endif ()

# #################################################
# <===    HSL
# #################################################

# #################################################
# ===>   Restart cmake
# #################################################

## Overload some CMake command to avoid modif all sub-projects CMakLists.txt file
## The original built-in commands are prefixed with an underscore if overriding any of them
## Prevent the modification of all sub-project

macro(add_library _target)
  _add_library (${_target} ${ARGN})
  
  if (COIN_ENABLE_DOWNLOAD_MUMPS)
    add_dependencies(${_target} EP_Mumps)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_METIS)
    add_dependencies(${_target} EP_Metis)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_ASL)
    add_dependencies(${_target} EP_ASL)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_LAPACK OR COIN_ENABLE_DOWLOAD_CLAPACK)
    add_dependencies(${_target} EP_Lapack)
  endif ()
endmacro ()

macro(add_executable _target)
  _add_executable (${_target} ${ARGN})
  
  if (COIN_ENABLE_DOWNLOAD_MUMPS)
    add_dependencies(${_target} EP_Mumps)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_METIS)
    add_dependencies(${_target} EP_Metis)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_ASL)
    add_dependencies(${_target} EP_ASL)
  endif ()
  if (COIN_ENABLE_DOWNLOAD_LAPACK OR COIN_ENABLE_DOWLOAD_CLAPACK)
    add_dependencies(${_target} EP_Lapack)
  endif ()
endmacro ()

# #################################################
# <===   Restart cmake 
# #################################################
