cmake_minimum_required (VERSION 3.9.0)

project(CorgiTest VERSION 1.0.0)

set(CMAKE_DEBUG_POSTFIX d)

# We'll install the library in lib64 if we're in x64, in lib otherwise
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(arch_suffix "")
else()
    set(arch_suffix "64")
endif()

set(target_name ${PROJECT_NAME})

add_library(${target_name} INTERFACE)

target_include_directories(${target_name}
INTERFACE 
$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
$<INSTALL_INTERFACE:lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/include/>)

#####################################################
# Installing the library and exporting the targets
#####################################################

include(CMakePackageConfigHelpers)

write_basic_package_version_file(${target_name}ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion)

install(TARGETS ${target_name} EXPORT ${target_name}-target
    LIBRARY     DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/
    ARCHIVE     DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/
    RUNTIME     DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/
    INCLUDES    DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/include)

install(
    EXPORT  ${target_name}-target
    DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/cmake
)

configure_file(cmake/config.cmake.in ${target_name}Config.cmake @ONLY)

# Installs the config and config version file 
install(
    FILES 
    "${CMAKE_CURRENT_BINARY_DIR}/${target_name}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${target_name}ConfigVersion.cmake"  
    DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/cmake
)

# Installing header files
install(
    DIRECTORY include/
    DESTINATION lib${arch_suffix}/${target_name}-${PROJECT_VERSION}/include
    FILES_MATCHING PATTERN *.h)

#########################################
# Tests
#########################################

option(BUILD_TESTS "Build the corgi test tests" ON)

if(BUILD_TESTS)
enable_testing()
add_subdirectory(test)
endif()