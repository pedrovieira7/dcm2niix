project(console)
set(PROGRAMS dcm2niix)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    # using Clang
    add_definitions(-dead_strip)
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # using GCC
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
    # using Intel C++
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    # using Visual Studio C++
endif ()


#We now use miniz, removing zlib dependency
#if(ZLIB_FOUND)
#  TARGET_LINK_LIBRARIES(dcm2niix z)
#else(ZLIB_FOUND)
#  ADD_DEFINITIONS(-DmyDisableZlib)
#endif(ZLIB_FOUND)
add_definitions(-DmyDisableJasper)
add_definitions(-DmyDisableOpenJPEG)

add_executable(dcm2niix
        main_console.cpp
        nii_dicom.cpp
        jpg_0XC3.cpp
        ujpeg.cpp
        nifti1_io_core.cpp
        nii_ortho.cpp
        nii_dicom_batch.cpp)


if (BATCH_VERSION)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

    message("Creating batch version")

    # Resolve system dependency on yaml-cpp, which apparently does not
    # provide a CMake find_package() module.

    # Step 1: Search for library using pkg-config
    find_package(PkgConfig)
    if (PKG_CONFIG_FOUND)
        pkg_check_modules(YAML_CPP yaml-cpp)
        find_path(YAML_CPP_INCLUDE_DIR
                NAMES yaml-cpp
                PATHS ${YAML_CPP_INCLUDE_DIRS})
        find_library(YAML_CPP_LIBRARY
                NAMES yaml-cpp
                PATHS ${YAML_CPP_LIBRARY_DIRS})
    endif ()

    # Step 2: Search for library locally. Only searches for a folder called yaml-cpp
    if (NOT YAML_CPP_LIBRARY)
        message("Checking for yaml-cpp local folder: yaml-cpp")
        if (EXISTS ${CMAKE_SOURCE_DIR}/yaml-cpp)
            message("local version found")
            SET(YAML_CPP_BUILD_TOOLS OFF CACHE BOOL "Build test packages off")
            add_subdirectory(${CMAKE_SOURCE_DIR}/yaml-cpp ${CMAKE_SOURCE_DIR}/yaml-cpp)
            set(YAML_CPP_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/yaml-cpp/include)
        else()
            message("Local version not found")
        endif()
    endif ()

    add_executable(dcm2niibatch
            main_console_batch.cpp
            nii_dicom.cpp
            jpg_0XC3.cpp
            ujpeg.cpp
            nifti1_io_core.cpp
            nii_ortho.cpp
            nii_dicom_batch.cpp)

    target_link_libraries(dcm2niibatch yaml-cpp)

    if (YAML_CPP_INCLUDE_DIR)
        include_directories(${YAML_CPP_INCLUDE_DIR})
    endif ()
endif ()

install(TARGETS ${PROGRAMS} DESTINATION bin)
