set(FOLDER_NAME "thirdparty")

# Only show moderate warnings for external library code.
if(IS_MSVC)
    if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    elseif(CMAKE_CXX_FLAGS MATCHES "/Wall")
        string(REGEX REPLACE "/Wall" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    endif()
    if(CMAKE_C_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    elseif(CMAKE_C_FLAGS MATCHES "/Wall")
        string(REGEX REPLACE "/Wall" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    endif()
elseif(IS_GNU OR IS_CLANG)
    if(CMAKE_CXX_FLAGS MATCHES "-Wall")
        string(REGEX REPLACE "-Wall" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    endif()
    if(CMAKE_C_FLAGS MATCHES "-Wall")
        string(REGEX REPLACE "-Wall" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
        set(REMOVED_WARNING_LEVEL TRUE)
    endif()
endif()

if(IS_MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W0")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W0")
elseif(IS_GNU OR IS_CLANG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")
endif()

set(CLANG_TIDY_EXE_TEMP ${CLANG_TIDY_EXE})
set(CLANG_TIDY_EXE "")
if(LSD_ENABLED)
    add_subdirectory(LSD)
endif()
add_subdirectory(PoissonRecon)
if(GPU_ENABLED)
    add_subdirectory(SiftGPU)
endif()
add_subdirectory(VLFeat)
if(TORCH_ENABLED)
    add_subdirectory(ALIKED)
    add_subdirectory(LightGlue)
endif()

if(FETCH_POSELIB)
    include(FetchContent)
    # Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP in CMake 3.24:
    if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
        cmake_policy(SET CMP0135 NEW)
    endif()
    if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.28")
        set(_fetch_content_declare_args "SYSTEM" "EXCLUDE_FROM_ALL")
    endif()
    FetchContent_Declare(poselib
        URL https://github.com/PoseLib/PoseLib/archive/8028473d92c9347794a0e3d3541863b5cbb15743.zip
        URL_HASH SHA256=20ff75d14c839dcab3d38fb51503c47088898f394b812472a758ec17720a54c2
        ${_fetch_content_declare_args}
    )
    message(STATUS "Configuring PoseLib...")
    set(MARCH_NATIVE OFF CACHE BOOL "")
    FetchContent_MakeAvailable(poselib)
    message(STATUS "Configuring PoseLib... done")
endif()

set(CLANG_TIDY_EXE ${CLANG_TIDY_EXE_TEMP})
