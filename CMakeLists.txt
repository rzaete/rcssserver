cmake_minimum_required(VERSION 3.5.1)

project(RCSSServer VERSION 19.0.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
endif()

# High-performance compiler flags for Release builds (GCC/Clang)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(
        "$<$<CONFIG:Release>:-march=native>"
        "$<$<CONFIG:Release>:-mtune=native>"
        "$<$<CONFIG:Release>:-ffast-math>"
    )
endif()

cmake_policy(SET CMP0069 NEW)

include(CheckIPOSupported)
check_ipo_supported(RESULT supported OUTPUT error)
if(supported)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
    message(STATUS "IPO/LTO enabled")
else()
    message(WARNING "IPO/LTO not supported: ${error}")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON)

find_package(ZLIB REQUIRED)
if(ZLIB_FOUND)
  set(HAVE_LIBZ TRUE)
endif()
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)
#find_package(Boost COMPONENTS system filesystem REQUIRED)
find_package(Boost 1.44.0 REQUIRED COMPONENTS)

include(GNUInstallDirs)
include(CheckIncludeFileCXX)
include(FetchContent)
FetchContent_Declare(
  cpptrace
  GIT_REPOSITORY https://github.com/jeremy-rifkin/cpptrace.git
  GIT_TAG        v1.0.4 # <HASH or TAG>
)
FetchContent_MakeAvailable(cpptrace)

check_include_file_cxx("sys/socket.h" HAVE_SYS_SOCKET_H)
check_include_file_cxx("sys/param.h" HAVE_SYS_PARAM_H)
check_include_file_cxx("sys/time.h" HAVE_SYS_TIME_H)
check_include_file_cxx("netinet/in.h" HAVE_NETINET_IN_H)
check_include_file_cxx("arpa/inet.h" HAVE_ARPA_INET_H)
check_include_file_cxx("netdb.h" HAVE_NETDB_H)
check_include_file_cxx("unistd.h" HAVE_UNISTD_H)
check_include_file_cxx("poll.h" HAVE_POLL_H)
check_include_file_cxx("pwd.h" HAVE_PWD_H)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake config.h)

add_subdirectory(rcss)
add_subdirectory(src)
