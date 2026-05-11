cmake_minimum_required(VERSION 3.16)
project(pokesilver C CXX)

include(CheckIPOSupported)
include(FetchContent)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE MinSizeRel)
endif()
# Default to a small-binary profile: MinSizeRel + section-based dead
# stripping + post-link symbol stripping. Flip GBRECOMP_SHRINK off for
# debugging; bump GBRECOMP_GENERATED_OPT_LEVEL to 3 for max perf.
option(GBRECOMP_SHRINK "Aggressive size shrink (dead-strip + symbol strip)" ON)
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-O0 -g)
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
        add_compile_options(-O3)
    endif()
    if(GBRECOMP_SHRINK)
        add_compile_options(-ffunction-sections -fdata-sections)
        add_link_options(-Wl,--gc-sections)
        if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
            add_link_options(-s)
        endif()
    endif()
endif()

# ---------------------------------------------------------------------------
# gb-recompiled runtime
# When consumed by an umbrella compilation (e.g. GB-Recomp/pgbcomp), the
# parent is expected to provide `gbrt`. We detect that and skip the fetch.
# ---------------------------------------------------------------------------
if(NOT TARGET gbrt)
    set(GBRT_REF "main" CACHE STRING "git ref of GB-Recomp/gb-recompiled to use")
    FetchContent_Declare(gb_recompiled
        GIT_REPOSITORY https://github.com/GB-Recomp/gb-recompiled.git
        GIT_TAG ${GBRT_REF}
        GIT_SHALLOW TRUE
    )
    FetchContent_GetProperties(gb_recompiled)
    if(NOT gb_recompiled_POPULATED)
        FetchContent_Populate(gb_recompiled)
    endif()
    add_subdirectory(${gb_recompiled_SOURCE_DIR}/runtime ${CMAKE_BINARY_DIR}/_gbrt_build)
endif()

# ---------------------------------------------------------------------------
# Cart sources
# ---------------------------------------------------------------------------
set(POKESILVER_SOURCES
    pokesilver_main.c
    pokesilver.c
    pokesilver_rom.c
    pokesilver_dispatch_chunk_0.c
    pokesilver_dispatch_chunk_1.c
    pokesilver_dispatch_chunk_2.c
    pokesilver_dispatch_chunk_3.c
    pokesilver_funcs_0.c
    pokesilver_funcs_1.c
    pokesilver_funcs_2.c
    pokesilver_funcs_3.c
    pokesilver_funcs_4.c
    pokesilver_funcs_5.c
    pokesilver_funcs_6.c
    pokesilver_funcs_7.c
    pokesilver_funcs_8.c
    pokesilver_funcs_9.c
    pokesilver_funcs_10.c
    pokesilver_funcs_11.c
    pokesilver_funcs_12.c
    pokesilver_funcs_13.c
    pokesilver_funcs_14.c
    pokesilver_funcs_15.c
    pokesilver_funcs_16.c
    pokesilver_funcs_17.c
    pokesilver_funcs_18.c
    pokesilver_funcs_19.c
    pokesilver_funcs_20.c
    pokesilver_funcs_21.c
    pokesilver_funcs_22.c
    pokesilver_funcs_23.c
    pokesilver_funcs_24.c
    pokesilver_funcs_25.c
    pokesilver_funcs_26.c
    pokesilver_funcs_27.c
    pokesilver_funcs_28.c
    pokesilver_funcs_29.c
    pokesilver_funcs_30.c
    pokesilver_funcs_31.c
    pokesilver_funcs_32.c
    pokesilver_funcs_33.c
    pokesilver_funcs_34.c
    pokesilver_funcs_35.c
    pokesilver_funcs_36.c
    pokesilver_funcs_37.c
    pokesilver_funcs_38.c
    pokesilver_funcs_39.c
    pokesilver_funcs_40.c
    pokesilver_funcs_41.c
    pokesilver_funcs_42.c
    pokesilver_funcs_43.c
    pokesilver_funcs_44.c
    pokesilver_funcs_45.c
    pokesilver_funcs_46.c
    pokesilver_funcs_47.c
    pokesilver_funcs_48.c
    pokesilver_funcs_49.c
    pokesilver_funcs_50.c
    pokesilver_funcs_51.c
)

set(GBRECOMP_GENERATED_OPT_LEVEL "1" CACHE STRING "Optimization level for generated ROM source files")
set_property(CACHE GBRECOMP_GENERATED_OPT_LEVEL PROPERTY STRINGS 0 1 2 3)
set_source_files_properties(${POKESILVER_SOURCES} PROPERTIES COMPILE_OPTIONS "-O${GBRECOMP_GENERATED_OPT_LEVEL}")

# ---------------------------------------------------------------------------
# pokesilver_cart: static library so umbrella compilations can link us in.
# Exposes `pokesilver_main(argc, argv)` for the launcher's g_games[].
# ---------------------------------------------------------------------------
add_library(pokesilver_cart STATIC ${POKESILVER_SOURCES})
target_include_directories(pokesilver_cart PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(pokesilver_cart PUBLIC gbrt)

# ---------------------------------------------------------------------------
# Standalone launcher executable. Only built when this is the top-level
# CMake project.
# ---------------------------------------------------------------------------
if(CMAKE_PROJECT_NAME STREQUAL "pokesilver")
    add_executable(pokesilver_launcher launcher_main.cpp)
    target_link_libraries(pokesilver_launcher PRIVATE pokesilver_cart gbrt)
    set_target_properties(pokesilver_launcher PROPERTIES OUTPUT_NAME pokesilver)
endif()
