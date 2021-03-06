# stb is a header-only library
# (see https://dominikberner.ch/cmake-interface-lib/ for the basics on 
# configuring a header-only library)

cmake_minimum_required(VERSION 3.12)

include(GNUInstallDirs)

project("stb"
    VERSION      0.0.0
    DESCRIPTION  "stb single-file public domain libraries for C/C++"
    HOMEPAGE_URL "https://github.com/nothings/stb")

add_library(${PROJECT_NAME} INTERFACE)

# Name alias so targets can always be used with namespace
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# BUILD_INTERFACE:   include path if being used to build
# INSTALL_INTERFACE: include path if being used as an installation (relative to install prefix)
target_include_directories(${PROJECT_NAME} INTERFACE 
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_17)

# Specifying rules for installing targets from a project
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}_exported_targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

# find_package() command will search for a file called "<name>Config.cmake" 
# or "<lower-case-name>-config.cmake" for each name specified. We use the 
# former convention as it includes case information.

# Note that using "Config"/"-config" and "ConfigVersion"/"-config-version" 
# suffixes are mandatory for Cmake to find them automatically (target file 
# is included manually by the config file and is not required to have the 
# "Targets"/"-targets" suffix, however it's good practice to follow the 
# same convention)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake
    VERSION       ${PROJECT_VERSION}
    COMPATIBILITY ExactVersion)

configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/)

# Create/Install the export file
install(EXPORT ${PROJECT_NAME}_exported_targets
    FILE        ${PROJECT_NAME}Targets.cmake
    NAMESPACE   ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/)

# Basically copying cmake files to the desired location
install(FILES
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/)

# Basically copying headers to the desired location
install(DIRECTORY ${PROJECT_SOURCE_DIR}/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h"
    PATTERN ".git" EXCLUDE
    PATTERN ".github" EXCLUDE
    PATTERN "cmake" EXCLUDE
    PATTERN "data" EXCLUDE
    PATTERN "deprecated" EXCLUDE
    PATTERN "docs" EXCLUDE
    PATTERN "tests" EXCLUDE
    PATTERN "tools" EXCLUDE)
