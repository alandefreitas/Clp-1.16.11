get_ac_init_version(${CMAKE_CURRENT_SOURCE_DIR}/configure.ac COINUTILS)

option(COINUTILS_ENABLE_UNITTEST "Enable building the unit test executable" OFF)

set(COINUTILS_VERSION_MAJOR   "${COINUTILS_MAJOR_VERSION}"  CACHE STRING "The COINUTILS major version number")
set(COINUTILS_VERSION_MINOR   "${COINUTILS_MINOR_VERSION}"  CACHE STRING "The COINUTILS minor version number")
set(COINUTILS_VERSION_RELEASE "${COINUTILS_PATCH_VERSION}"  CACHE STRING "The COINUTILS patch version number")
set(COINUTILS_VERSION         "${COINUTILS_VERSION_STRING}" CACHE STRING "The COINUTILS version")

option(COINUTILS_HAS_ZLIB "CoinUtils uses the Zlib library" OFF)
set(COINUTILS_HAS_ZLIB_INCLUDE_PATH "None" CACHE STRING "The Zlib library include Path")
set(COINUTILS_HAS_ZLIB_LIBRARY_PATH "None" CACHE STRING "The Zlib library absolute library Path")

option(COINUTILS_HAS_BZLIB "CoinUtils uses the BZlib library" OFF)
set(COINUTILS_HAS_BZLIB_INCLUDE_PATH "None" CACHE STRING "The BZlib library include Path")
set(COINUTILS_HAS_BZLIB_LIBRARY_PATH "None" CACHE STRING "The BZlib library absolute library Path")

if (WIN32 AND NOT MINGW)
  set(ZLIB_NAME  "zlibstatic.lib" CACHE FILEPATH "The Zlib library name"  FORCE)
  set(BZLIB_NAME "libbz2.lib"     CACHE FILEPATH "The BZlib library name" FORCE)
elseif (MINGW)
  set(ZLIB_NAME  "libzlibstatic.a" CACHE FILEPATH "The Zlib library name"  FORCE)
  set(BZLIB_NAME "libbz2.a"        CACHE FILEPATH "The BZlib library name" FORCE)
else ()
  set(ZLIB_NAME  "libz.a"   CACHE FILEPATH "The Zlib library name"  FORCE)
  set(BZLIB_NAME "libbz2.a" CACHE FILEPATH "The BZlib library name" FORCE)
endif ()

find_package(Subversion)
if (Subversion_FOUND)
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/.svn")
    Subversion_WC_INFO(${CMAKE_CURRENT_SOURCE_DIR} COINUTILS)
  else ()
    set(COINUTILS_WC_REVISION "0")
  endif ()
else ()
  set(COINUTILS_WC_REVISION "0")
endif ()
message(STATUS "Current COINUTILS revision is ${COINUTILS_WC_REVISION}")
set(COINUTILS_SVN_REV "${COINUTILS_WC_REVISION}" CACHE STRING "The COINUTILS subversion revision")

mark_as_advanced(COINUTILS_VERSION_MAJOR
                 COINUTILS_VERSION_MINOR
                 COINUTILS_VERSION_RELEASE
                 COINUTILS_VERSION
                 COINUTILS_SVN_REV
                 COINUTILS_HAS_ZLIB
                 COINUTILS_HAS_ZLIB_INCLUDE_PATH
                 COINUTILS_HAS_ZLIB_LIBRARY_PATH
                 ZLIB_NAME
                 COINUTILS_HAS_BZLIB
                 COINUTILS_HAS_BZLIB_INCLUDE_PATH
                 COINUTILS_HAS_BZLIB_LIBRARY_PATH
                 BZLIB_NAME)

include(coin-macros)

add_definitions(-DCOINUTILS_BUILD)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/config_coinutils.h.in ${CMAKE_BINARY_DIR}/CoinUtils/include/config_coinutils.h)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/config.h.in           ${CMAKE_BINARY_DIR}/CoinUtils/include/config.h)

set(CoinUtils_SRC_LIST ${CoinUtils_DIR}/src/CoinAlloc.cpp
                       ${CoinUtils_DIR}/src/CoinBuild.cpp
                       ${CoinUtils_DIR}/src/CoinDenseVector.cpp
                       ${CoinUtils_DIR}/src/CoinError.cpp
                       ${CoinUtils_DIR}/src/CoinFactorization1.cpp
                       ${CoinUtils_DIR}/src/CoinFactorization2.cpp
                       ${CoinUtils_DIR}/src/CoinFactorization3.cpp
                       ${CoinUtils_DIR}/src/CoinFactorization4.cpp
                       ${CoinUtils_DIR}/src/CoinSimpFactorization.cpp
                       ${CoinUtils_DIR}/src/CoinDenseFactorization.cpp
                       ${CoinUtils_DIR}/src/CoinOslFactorization.cpp
                       ${CoinUtils_DIR}/src/CoinOslFactorization2.cpp
                       ${CoinUtils_DIR}/src/CoinOslFactorization3.cpp
                       ${CoinUtils_DIR}/src/CoinFileIO.cpp
                       ${CoinUtils_DIR}/src/CoinFinite.cpp
                       ${CoinUtils_DIR}/src/CoinIndexedVector.cpp
                       ${CoinUtils_DIR}/src/CoinLpIO.cpp
                       ${CoinUtils_DIR}/src/CoinMessage.cpp
                       ${CoinUtils_DIR}/src/CoinMessageHandler.cpp
                       ${CoinUtils_DIR}/src/CoinModel.cpp
                       ${CoinUtils_DIR}/src/CoinStructuredModel.cpp
                       ${CoinUtils_DIR}/src/CoinModelUseful.cpp
                       ${CoinUtils_DIR}/src/CoinModelUseful2.cpp
                       ${CoinUtils_DIR}/src/CoinMpsIO.cpp
                       ${CoinUtils_DIR}/src/CoinPackedMatrix.cpp
                       ${CoinUtils_DIR}/src/CoinPackedVector.cpp
                       ${CoinUtils_DIR}/src/CoinPackedVectorBase.cpp
                       ${CoinUtils_DIR}/src/CoinParam.cpp
                       ${CoinUtils_DIR}/src/CoinParamUtils.cpp
                       ${CoinUtils_DIR}/src/CoinPostsolveMatrix.cpp
                       ${CoinUtils_DIR}/src/CoinPrePostsolveMatrix.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveDoubleton.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveDual.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveDupcol.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveEmpty.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveFixed.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveForcing.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveHelperFunctions.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveImpliedFree.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveIsolated.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveMatrix.cpp
                       ${CoinUtils_DIR}/src/CoinPresolvePsdebug.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveMonitor.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveSingleton.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveSubst.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveTighten.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveTripleton.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveUseless.cpp
                       ${CoinUtils_DIR}/src/CoinPresolveZeros.cpp
                       ${CoinUtils_DIR}/src/CoinSearchTree.cpp
                       ${CoinUtils_DIR}/src/CoinShallowPackedVector.cpp
                       ${CoinUtils_DIR}/src/CoinSnapshot.cpp
                       ${CoinUtils_DIR}/src/CoinWarmStartBasis.cpp
                       ${CoinUtils_DIR}/src/CoinWarmStartVector.cpp
                       ${CoinUtils_DIR}/src/CoinWarmStartDual.cpp
                       ${CoinUtils_DIR}/src/CoinWarmStartPrimalDual.cpp
                       ${CoinUtils_DIR}/src/CoinRational.cpp)

set(Coin_SRC_Headers ${CoinUtils_DIR}/src/Coin_C_defines.h
                     ${CoinUtils_DIR}/src/CoinAlloc.hpp
                     ${CoinUtils_DIR}/src/CoinBuild.hpp
                     ${CoinUtils_DIR}/src/CoinDenseVector.hpp
                     ${CoinUtils_DIR}/src/CoinDistance.hpp
                     ${CoinUtils_DIR}/src/CoinError.hpp
                     ${CoinUtils_DIR}/src/CoinFactorization.hpp
                     ${CoinUtils_DIR}/src/CoinSimpFactorization.hpp
                     ${CoinUtils_DIR}/src/CoinDenseFactorization.hpp
                     ${CoinUtils_DIR}/src/CoinOslFactorization.hpp
                     ${CoinUtils_DIR}/src/CoinFileIO.hpp
                     ${CoinUtils_DIR}/src/CoinFinite.hpp
                     ${CoinUtils_DIR}/src/CoinFloatEqual.hpp
                     ${CoinUtils_DIR}/src/CoinHelperFunctions.hpp
                     ${CoinUtils_DIR}/src/CoinIndexedVector.hpp
                     ${CoinUtils_DIR}/src/CoinLpIO.hpp
                     ${CoinUtils_DIR}/src/CoinMessage.hpp
                     ${CoinUtils_DIR}/src/CoinMessageHandler.hpp
                     ${CoinUtils_DIR}/src/CoinModel.hpp
                     ${CoinUtils_DIR}/src/CoinStructuredModel.hpp
                     ${CoinUtils_DIR}/src/CoinModelUseful.hpp
                     ${CoinUtils_DIR}/src/CoinMpsIO.hpp
                     ${CoinUtils_DIR}/src/CoinPackedMatrix.hpp
                     ${CoinUtils_DIR}/src/CoinPackedVector.hpp
                     ${CoinUtils_DIR}/src/CoinPackedVectorBase.hpp
                     ${CoinUtils_DIR}/src/CoinParam.hpp
                     ${CoinUtils_DIR}/src/CoinPragma.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveDoubleton.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveDual.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveDupcol.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveEmpty.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveFixed.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveForcing.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveImpliedFree.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveIsolated.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveMatrix.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveMonitor.hpp
                     ${CoinUtils_DIR}/src/CoinPresolvePsdebug.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveSingleton.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveSubst.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveTighten.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveTripleton.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveUseless.hpp
                     ${CoinUtils_DIR}/src/CoinPresolveZeros.hpp
                     ${CoinUtils_DIR}/src/CoinSearchTree.hpp
                     ${CoinUtils_DIR}/src/CoinShallowPackedVector.hpp
                     ${CoinUtils_DIR}/src/CoinSignal.hpp
                     ${CoinUtils_DIR}/src/CoinSmartPtr.hpp
                     ${CoinUtils_DIR}/src/CoinSnapshot.hpp
                     ${CoinUtils_DIR}/src/CoinSort.hpp
                     ${CoinUtils_DIR}/src/CoinTime.hpp
                     ${CoinUtils_DIR}/src/CoinTypes.hpp
                     ${CoinUtils_DIR}/src/CoinUtility.hpp
                     ${CoinUtils_DIR}/src/CoinWarmStart.hpp
                     ${CoinUtils_DIR}/src/CoinWarmStartBasis.hpp
                     ${CoinUtils_DIR}/src/CoinWarmStartVector.hpp
                     ${CoinUtils_DIR}/src/CoinWarmStartDual.hpp
                     ${CoinUtils_DIR}/src/CoinWarmStartPrimalDual.hpp
                     ${CoinUtils_DIR}/src/CoinRational.hpp)

set(CoinUtilsTest_SRC_LIST ${CoinUtilsTest_DIR}/CoinLpIOTest.cpp
                           ${CoinUtilsTest_DIR}/CoinDenseVectorTest.cpp
                           ${CoinUtilsTest_DIR}/CoinErrorTest.cpp
                           ${CoinUtilsTest_DIR}/CoinIndexedVectorTest.cpp
                           ${CoinUtilsTest_DIR}/CoinMessageHandlerTest.cpp
                           ${CoinUtilsTest_DIR}/CoinModelTest.cpp
                           ${CoinUtilsTest_DIR}/CoinMpsIOTest.cpp
                           ${CoinUtilsTest_DIR}/CoinPackedMatrixTest.cpp
                           ${CoinUtilsTest_DIR}/CoinPackedVectorTest.cpp
                           ${CoinUtilsTest_DIR}/CoinShallowPackedVectorTest.cpp
                           ${CoinUtilsTest_DIR}/unitTest.cpp)

include_directories(${CoinUtils_DIR}/src)

if (WIN32)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_DEPRECATE)
endif ()
if (COINUTILS_HAS_ZLIB)
  add_definitions(-DCOIN_HAS_ZLIB)
endif ()
if (COINUTILS_HAS_BZLIB)
  add_definitions(-DCOIN_HAS_BZLIB)
endif ()

coin_check_and_add_include_path(COINUTILS_HAS_ZLIB_INCLUDE_PATH)
coin_check_and_add_library_path(COINUTILS_HAS_ZLIB_LIBRARY_PATH)
coin_check_and_add_include_path(COINUTILS_HAS_BZLIB_INCLUDE_PATH)
coin_check_and_add_library_path(COINUTILS_HAS_BZLIB_LIBRARY_PATH)

if (ENABLE_SHARED_LIBRARIES)
  add_library(libCoinUtils SHARED ${CoinUtils_SRC_LIST})
else ()
  add_library(libCoinUtils STATIC ${CoinUtils_SRC_LIST})
endif ()
if (NOT WIN32 OR MINGW)
  set_target_properties(libCoinUtils PROPERTIES PREFIX "")
endif ()
target_include_directories(libCoinUtils BEFORE PRIVATE ${CMAKE_BINARY_DIR}/CoinUtils/include)

if (COINUTILS_ENABLE_UNITTEST)
  add_executable(unitTest-coinutils ${CoinUtilsTest_SRC_LIST})
  target_link_libraries(unitTest-coinutils libCoinUtils)
  target_include_directories(unitTest-coinutils BEFORE PRIVATE ${CMAKE_BINARY_DIR}/CoinUtils/include)
  if (COINUTILS_HAS_ZLIB)
    target_link_libraries(unitTest-coinutils ${ZLIB_NAME})
  endif ()
  if (COINUTILS_HAS_BZLIB)
    target_link_libraries(unitTest-coinutils ${BZLIB_NAME})
  endif ()
  if (COINUTILS_USE_THREADS)
    if (MINGW)
      target_link_libraries(unitTest-coinutils winpthread)
    elseif (NOT WIN32)
      target_link_libraries(unitTest-coinutils pthread)
    else ()
      target_link_libraries(unitTest-coinutils pthreadVC2)
    endif ()
  endif ()
  
  set(TEST_LIB_PATH "${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR}/lib")
  set(TEST_BIN_PATH "${CMAKE_BINARY_DIR}/Dependencies/${CMAKE_CFG_INTDIR}/bin")
  
  add_test(NAME unitTest_coinutils
           COMMAND $<TARGET_FILE:unitTest-coinutils> -mpsDir=${CMAKE_SOURCE_DIR}/Data/Sample)
  set_tests_properties(unitTest_coinutils PROPERTIES TIMEOUT 30)
  if (WIN32)
    set_tests_properties(unitTest_coinutils PROPERTIES ENVIRONMENT "PATH=${TEST_LIB_DIR}\\;${TEST_BIN_PATH}")
  endif ()
endif ()

install(TARGETS libCoinUtils 
        DESTINATION lib)

install(FILES ${Coin_SRC_Headers} 
        DESTINATION include/coin)

install(FILES ${CMAKE_BINARY_DIR}/CoinUtils/include/config_coinutils.h
        DESTINATION include/coin
        RENAME CoinUtilsConfig.h)

install(FILES ${CMAKE_BINARY_DIR}/CoinUtils/include/config_coinutils.h
        DESTINATION include/coin)
