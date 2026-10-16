cmake_minimum_required(VERSION 3.19)

file(READ "${CMAKE_SOURCE_DIR}/VERSION" VER_RAW)
string(STRIP ${VER_RAW} VERSION)

project(
  abyss-scanner
  DESCRIPTION "An Abyss version of wayland-scanner in and for C++"
  VERSION ${VERSION})

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(PREFIX ${CMAKE_INSTALL_PREFIX})

set(CMAKE_MESSAGE_LOG_LEVEL "STATUS")

if(CMAKE_BUILD_TYPE MATCHES Debug OR CMAKE_BUILD_TYPE MATCHES DEBUG)
  message(STATUS "Configuring abyss-scanner in Debug with CMake")
  add_compile_definitions(ABYSSLAND_DEBUG)
else()
  add_compile_options(-O3)
  message(STATUS "Configuring abyss-scanner in Release with CMake")
endif()

# configure
set(CMAKE_CXX_STANDARD 23)
add_compile_options(-Wall -Wextra -Wno-unused-parameter -Wno-unused-value
                    -Wno-missing-field-initializers -Wno-narrowing)

add_compile_definitions(SCANNER_VERSION="${VERSION}")

configure_file(abyss-scanner.pc.in abyss-scanner.pc @ONLY)

# dependencies
message(STATUS "Checking deps...")

find_package(Threads REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(deps REQUIRED IMPORTED_TARGET pugixml)

file(GLOB_RECURSE SRCFILES CONFIGURE_DEPENDS "src/*.cpp")
add_executable(abyss-scanner ${SRCFILES})
target_link_libraries(abyss-scanner PRIVATE rt Threads::Threads
                                                  PkgConfig::deps)

configure_package_config_file(
  abyss-scanner-config.cmake.in abyss-scanner-config.cmake
  INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/abyss-scanner"
  PATH_VARS CMAKE_INSTALL_BINDIR)
write_basic_package_version_file(
  "abyss-scanner-config-version.cmake"
  VERSION "${VERSION}"
  ARCH_INDEPENDENT
  COMPATIBILITY AnyNewerVersion)

# Installation
install(TARGETS abyss-scanner)
install(FILES ${CMAKE_BINARY_DIR}/abyss-scanner.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
install(FILES ${CMAKE_BINARY_DIR}/abyss-scanner-config.cmake
              ${CMAKE_BINARY_DIR}/abyss-scanner-config-version.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/abyss-scanner)
