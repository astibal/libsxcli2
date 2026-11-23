cmake_minimum_required(VERSION 3.16)
project(libsxcli2 VERSION 0.1.0 LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(sxcli2 STATIC
    src/service/libcli/cli.cpp
    src/service/libcli/fd_transport.cpp
    src/service/libcli/line_editor.cpp)
add_library(sxcli2::sxcli2 ALIAS sxcli2)
add_library(smithproxy_libcli ALIAS sxcli2)
target_compile_features(sxcli2 PUBLIC cxx_std_17)
target_include_directories(sxcli2 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/service/libcli>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/service/libcli>)

install(TARGETS sxcli2 EXPORT sxcli2Targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(DIRECTORY src/service/libcli/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/service/libcli
    FILES_MATCHING PATTERN "*.hpp" PATTERN "examples" EXCLUDE)
set(package_dir ${CMAKE_INSTALL_LIBDIR}/cmake/sxcli2)
install(EXPORT sxcli2Targets NAMESPACE sxcli2:: DESTINATION ${package_dir})
configure_package_config_file(cmake/sxcli2Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/sxcli2Config.cmake INSTALL_DESTINATION ${package_dir})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/sxcli2ConfigVersion.cmake
    VERSION ${PROJECT_VERSION} COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/sxcli2Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/sxcli2ConfigVersion.cmake DESTINATION ${package_dir})

set(sxcli2_standalone OFF)
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(sxcli2_standalone ON)
endif()
option(SXCLI2_BUILD_TESTS "Build CLI tests" ${sxcli2_standalone})
option(SXCLI2_BUILD_EXAMPLES "Build the interactive CLI example" ${sxcli2_standalone})
if(SXCLI2_BUILD_TESTS)
    enable_testing()
    find_package(GTest REQUIRED)
    find_package(Threads REQUIRED)
    add_executable(sxcli2_tests
        tests/unit/service/libcli/cli_tests.cpp
        tests/unit/service/libcli/fd_transport_tests.cpp)
    target_link_libraries(sxcli2_tests PRIVATE sxcli2 GTest::gtest_main Threads::Threads)
    add_test(NAME sxcli2.unit COMMAND sxcli2_tests)
endif()
if(SXCLI2_BUILD_EXAMPLES)
    add_executable(sxcli2-demo src/service/libcli/examples/example.cpp)
    target_link_libraries(sxcli2-demo PRIVATE sxcli2)
endif()
