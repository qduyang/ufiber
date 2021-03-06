cmake_minimum_required(VERSION 3.8)
project(ufiber VERSION 0.1 LANGUAGES CXX)

option(UFIBER_SANITIZE "Build ufiber tests and examples with address & undefined sanitization enabled" OFF)
if (UFIBER_SANITIZE)
    message(STATUS "ufiber: address & undefined sanitizers enabled")
    add_compile_options(-fsanitize=address,undefined)
    link_libraries(-fsanitize=address,undefined)
endif()

list(INSERT CMAKE_MODULE_PATH 0 ${PROJECT_SOURCE_DIR}/CMakeModules)

if (CMAKE_VERSION VERSION_LESS 3.11)
    # Latest FindBoost.cmake has likely been updated to detect Boost version not yet released
    if (NOT EXISTS "${CMAKE_BINARY_DIR}/cmake/FindBoost.cmake")
        message(STATUS "Downloading FindBoost.cmake from https://gitlab.kitware.com/cmake/ release branch")
        file(
            DOWNLOAD
            "https://gitlab.kitware.com/cmake/cmake/raw/release/Modules/FindBoost.cmake"
            "${CMAKE_BINARY_DIR}/cmake/FindBoost.cmake")
    endif()
    list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_BINARY_DIR}/cmake)
endif()

find_package(Boost 1.67
             COMPONENTS
                context
                system
             REQUIRED)

find_package (Threads)


add_library(ufiber INTERFACE)
add_library(ufiber::ufiber ALIAS ufiber)

target_include_directories(ufiber INTERFACE
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(
    ufiber
    INTERFACE
        Boost::boost
        Boost::system
        Boost::context
        Threads::Threads)

target_compile_features(ufiber INTERFACE cxx_std_11)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "ufiberConfigVersion.cmake"
    COMPATIBILITY AnyNewerVersion)

install(FILES
            "ufiberConfig.cmake"
            "${CMAKE_BINARY_DIR}/ufiberConfigVersion.cmake"
        DESTINATION lib/cmake/ufiber)

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/
        DESTINATION include
        FILES_MATCHING PATTERN "*.hpp")

install(TARGETS ufiber
        EXPORT ufiberTargets
        INCLUDES DESTINATION include)

install(EXPORT ufiberTargets
        FILE ufiberTargets.cmake
        NAMESPACE ufiber::
        DESTINATION lib/cmake/ufiber)

include(CTest)
if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()

add_subdirectory(examples)
