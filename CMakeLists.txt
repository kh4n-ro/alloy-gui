cmake_minimum_required(VERSION 2.8.6)
include(CryptoNoteWallet.cmake)

project(${CN_PROJECT_NAME})

execute_process(COMMAND git log -1 --pretty=format:%h
WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
OUTPUT_VARIABLE GIT_REVISION)

set(CRYPTONOTE_LIB cryptonote)

include_directories(${CMAKE_BINARY_DIR}
  src
  include
  alloy/external
  alloy/external/rocksdb/include/
  alloy/include
  alloy/src
)

find_package(Qt5Gui REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Network REQUIRED)
find_package(Qt5DBus REQUIRED)

include_directories(${Qt5DBus_INCLUDE_DIRS} alloy/external/rocksdb/include/ include src external "${CMAKE_CURRENT_BINARY_DIR}/version")

if(STATIC)
  set(Boost_USE_STATIC_LIBS ON)
  set(Boost_USE_STATIC_RUNTIME ON)
endif()

find_package(Boost 1.58 REQUIRED COMPONENTS date_time filesystem program_options regex serialization system thread chrono)
if ((${Boost_MAJOR_VERSION} EQUAL 1) AND (${Boost_MINOR_VERSION} EQUAL 54))
  message(SEND_ERROR "Boost version 1.58 is unsupported, more details are available here http://goo.gl/RrCFmA")
endif ()

include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

set(VERSION "")
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/miniupnpcstrings.h.in "miniupnpcstrings.h")
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/alloy/src/version.h.in "version.h")
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/CryptoNoteWalletConfig.h.in "CryptoNoteWalletConfig.h")

add_definitions(-DGIT_REVISION="${GIT_REVISION}")

set(CMAKE_AUTOMOC ON)
set_property(SOURCE qrc_resources.cpp PROPERTY SKIP_AUTOMOC ON)

file(GLOB_RECURSE CRYPTONOTE_SOURCES
  alloy/external/miniupnpc/connecthostport.c
  alloy/external/miniupnpc/igd_desc_parse.c
  alloy/external/miniupnpc/minisoap.c
  alloy/external/miniupnpc/miniupnpc.c
  alloy/external/miniupnpc/miniwget.c
  alloy/external/miniupnpc/minixml.c
  alloy/external/miniupnpc/portlistingparse.c
  alloy/external/miniupnpc/receivedata.c
  alloy/external/miniupnpc/upnpcommands.c
  alloy/external/miniupnpc/upnpreplyparse.c
  alloy/src/BlockchainExplorer/BlockchainExplorer.cpp
  alloy/src/BlockchainExplorer/BlockchainExplorerErrors.cpp
  alloy/src/Common/Base58.cpp
  alloy/src/Common/CommandLine.cpp
  alloy/src/Common/Util.cpp
  alloy/src/Common/StringTools.cpp
  alloy/src/Common/JsonValue.cpp
  alloy/src/Common/ConsoleTools.cpp
  alloy/src/Common/MemoryInputStream.cpp
  alloy/src/Common/PathTools.cpp
  alloy/src/Common/StdInputStream.cpp
  alloy/src/Common/StdOutputStream.cpp
  alloy/src/Common/StreamTools.cpp
  alloy/src/Common/StringOutputStream.cpp
  alloy/src/Common/StringView.cpp
  alloy/src/Common/VectorOutputStream.cpp
  alloy/src/Common/ScopeExit.cpp
  alloy/src/ConnectivityTool/ConnectivityTool.cpp
  alloy/src/crypto/blake256.c
  alloy/src/crypto/chacha8.c
  alloy/src/crypto/crypto-ops-data.c
  alloy/src/crypto/crypto-ops.c
  alloy/src/crypto/crypto.cpp
  alloy/src/crypto/groestl.c
  alloy/src/crypto/hash-extra-blake.c
  alloy/src/crypto/hash-extra-groestl.c
  alloy/src/crypto/hash-extra-jh.c
  alloy/src/crypto/hash-extra-skein.c
  alloy/src/crypto/hash.c
  alloy/src/crypto/jh.c
  alloy/src/crypto/keccak.c
  alloy/src/crypto/oaes_lib.c
  alloy/src/crypto/random.c
  alloy/src/crypto/skein.c
  alloy/src/crypto/slow-hash.c
  alloy/src/crypto/slow-hash.cpp
  alloy/src/crypto/tree-hash.c
  alloy/src/CryptoNoteCore/*.cpp
  alloy/src/CryptoNoteProtocol/*.cpp
  alloy/src/Daemon/*.cpp
  alloy/src/HTTP/*.cpp
  alloy/src/InProcessNode/InProcessNode.cpp
  alloy/src/InProcessNode/InProcessNodeErrors.cpp
  alloy/src/Logging/*.cpp
  alloy/src/NodeRpcProxy/NodeErrors.cpp
  alloy/src/NodeRpcProxy/NodeRpcProxy.cpp
  alloy/src/P2p/*.cpp
  alloy/src/Rpc/*.cpp
  alloy/src/Serialization/*.cpp
  alloy/src/Transfers/BlockchainSynchronizer.cpp
  alloy/src/Transfers/SynchronizationState.cpp
  alloy/src/Transfers/TransfersConsumer.cpp
  alloy/src/Transfers/TransfersContainer.cpp
  alloy/src/Transfers/TransfersSubscription.cpp
  alloy/src/Transfers/TransfersSynchronizer.cpp
  alloy/src/Wallet/*.cpp
  alloy/src/WalletLegacy/KeysStorage.cpp
  alloy/src/WalletLegacy/WalletLegacy.cpp
  alloy/src/WalletLegacy/WalletHelper.cpp
  alloy/src/WalletLegacy/WalletLegacySerializer.cpp
  alloy/src/WalletLegacy/WalletLegacySerialization.cpp
  alloy/src/WalletLegacy/WalletTransactionSender.cpp
  alloy/src/WalletLegacy/WalletUnconfirmedTransactions.cpp
  alloy/src/WalletLegacy/WalletUserTransactionsCache.cpp
  alloy/src/System/ContextGroup.cpp
  alloy/src/System/Event.cpp
  alloy/src/System/EventLock.cpp
  alloy/src/System/InterruptedException.cpp
  alloy/src/System/Ipv4Address.cpp
  alloy/src/System/TcpStream.cpp
  alloy/src/JsonRpcServer/*.cpp
  alloy/src/PaymentGate/*.cpp
  alloy/src/PaymentGateService/*.cpp
  alloy/src/Miner/*.cpp
)

file(GLOB_RECURSE SOURCES src/*.cpp)
file(GLOB_RECURSE HEADERS src/*.h)
file(GLOB_RECURSE FORMS src/gui/ui/*.ui)

if(MSVC)
  include_directories(Platform/Windows)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Windows/ApplicationEventHandler.cpp)
elseif(APPLE)
  include_directories(Platform/OSX)
  include_directories(Platform/Posix)
  #set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/*)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/ApplicationEventHandler.mm)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
else()
  include_directories(Platform/Linux)
  include_directories(Platform/Posix)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Linux/ApplicationEventHandler.cpp)
  set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
endif()


set(QRC src/resources.qrc)

qt5_wrap_ui(UIS ${FORMS})
qt5_add_resources(RCC ${QRC})

if (WIN32)
  if (NOT MSVC)
    message(FATAL_ERROR "Only MSVC is supported on this platform")
  endif ()

  set(STATIC ${MSVC} CACHE BOOL "Link libraries statically")

  if(STATIC)
    foreach(VAR CMAKE_C_FLAGS_DEBUG CMAKE_CXX_FLAGS_DEBUG CMAKE_C_FLAGS_RELWITHDEBINFO CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_C_FLAGS_RELEASE CMAKE_CXX_FLAGS_RELEASE)
      string(REPLACE "/MD" "/MT" ${VAR} "${${VAR}}")
    endforeach()
  endif()

  add_definitions(/D_CRT_SECURE_NO_WARNINGS /D_WIN32_WINNT=0x0600 /DSTATICLIB)
  include_directories(alloy/src/Platform/msc)
  link_directories(alloy/external/rocksdb)

  set(PLATFORM_DIR Windows)
  set(BUILD_PLATFORM WIN32)
  set(BUILD_RESOURCES src/alloywallet.rc)
  set(QTMAIN Qt5::WinMain)

elseif (UNIX)

  add_custom_target(
     rocksdb
     COMMAND make static_lib -Wno-undef -Wno-unused-result -Wno-unused-value
     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/alloy/external/rocksdb
  )
  add_library(rocksdblib STATIC IMPORTED GLOBAL)
  set_target_properties(rocksdblib PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/alloy/external/rocksdb/librocksdb.a)
  add_dependencies(rocksdblib rocksdb)

  set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} alloy/external/miniupnpc/minissdpc.c)

  if (APPLE)
    enable_language(ASM)
    file(GLOB_RECURSE OBJC_SOURCES src/*.mm)
    set(SOURCES ${SOURCES} ${OBJC_SOURCES})
    set(PLATFORM_DIR OSX)
    set(MACOSX_BUNDLE_INFO_STRING "Cryptonote GUI wallet")
    set(MACOSX_BUNDLE_LONG_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
    set(MACOSX_BUNDLE_BUNDLE_NAME AlloyWallet)
    set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
    set(MACOSX_BUNDLE_BUNDLE_VERSION "$${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")

    find_package(Qt5PrintSupport REQUIRED)

    include_directories(/usr/include/malloc)

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++14 -stdlib=libc++")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -maes -D_DARWIN_C_SOURCE")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework Cocoa -framework OpenGL -framework CoreFoundation -framework Carbon -framework IOKit -L/usr/lib")

    set(MACOSX_BUNDLE_ICON_FILE alloy.icns)
    set(APPLICATION_ICON src/images/alloy.icns)
    set_source_files_properties(${APPLICATION_ICON} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")

    set(BUILD_PLATFORM MACOSX_BUNDLE)
    set(BUILD_RESOURCES ${APPLICATION_ICON})

    GET_TARGET_PROPERTY(QT_LIB_DIR "${Qt5Widgets_LIBRARIES}" LOCATION)
    GET_FILENAME_COMPONENT(QT_LIB_DIR "${QT_LIB_DIR}" PATH)

  else(APPLE)

    set(PLATFORM_DIR Linux)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++14")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maes -std=c14")
  endif (APPLE)
endif ()

include_directories(alloy/src/Platform/${PLATFORM_DIR})
file(GLOB PLATFORM_SOURCES alloy/src/Platform/${PLATFORM_DIR}/System/*)
set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} ${PLATFORM_SOURCES} ${CRYPTONOTE_PLATFORM})

add_library(${CRYPTONOTE_LIB} STATIC ${CRYPTONOTE_SOURCES})
set_target_properties(${CRYPTONOTE_LIB} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
target_link_libraries(${CRYPTONOTE_LIB} ${Boost_LIBRARIES} ${Qt5DBus_LIBRARIES} rocksdblib)

add_executable(${PROJECT_NAME} ${BUILD_PLATFORM} ${BUILD_RESOURCES} ${SOURCES} ${HEADERS} ${UIS} ${RCC})

set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} ${QTMAIN} ${CRYPTONOTE_LIB} ${Qt5DBus_LIBRARIES} rocksdblib)

if (APPLE)
  qt5_use_modules(${PROJECT_NAME} PrintSupport)
elseif (UNIX)
  target_link_libraries(${PROJECT_NAME} -lpthread)
elseif (WIN32)
  target_link_libraries(${PROJECT_NAME} Imm32 Iphlpapi Winmm UxTheme d3d9 dxguid Rpcrt4 Wtsapi32 Dwmapi "${_qt5Widgets_install_prefix}/lib/qtharfbuzz.lib" "${_qt5Widgets_install_prefix}/lib/qtlibpng.lib" "${_qt5Widgets_install_prefix}/lib/qtpcre.lib" "${_qt5Widgets_install_prefix}/lib/Qt5EventDispatcherSupport.lib" "${_qt5Widgets_install_prefix}/lib/Qt5OpenGL.lib" "${_qt5Widgets_install_prefix}/lib/Qt5AccessibilitySupport.lib" "${_qt5Widgets_install_prefix}/lib/Qt5ThemeSupport.lib" "${_qt5Widgets_install_prefix}/lib/libGLESv2.lib" "${_qt5Widgets_install_prefix}/lib/libEGL.lib" "${_qt5Widgets_install_prefix}/lib/qtfreetype.lib" "${_qt5Widgets_install_prefix}/lib/Qt5FontDatabaseSupport.lib" "${_qt5Widgets_install_prefix}/lib/preprocessor.lib" "${_qt5Widgets_install_prefix}/lib/translator.lib" "${_qt5Widgets_install_prefix}/plugins/platforms/qwindows.lib")
endif (APPLE)

qt5_use_modules(${PROJECT_NAME} Widgets Gui Network)
