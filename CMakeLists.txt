cmake_minimum_required(VERSION 3.28)
project(allegro5-cmake)

# Png and zlib already have REQUIRED, but there's no real reason for them to
find_package(PNG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(Freetype REQUIRED)

include(FetchContent)
FetchContent_Declare(allegro5
    GIT_REPOSITORY https://github.com/liballeg/allegro5
    GIT_TAG 5.2.11.1
    OVERRIDE_FIND_PACKAGE
    # git reset --hard is required for CMake not to have a stroke
    PATCH_COMMAND git reset --hard && git apply ${CMAKE_CURRENT_SOURCE_DIR}/patches/allegro.patch
    EXCLUDE_FROM_ALL
)
FetchContent_MakeAvailable(
    allegro5
)

set (ALLEGRO_INCLUDE_DIRS
    ${allegro5_SOURCE_DIR}/include/
    ${allegro5_BINARY_DIR}/include/
    ${allegro5_SOURCE_DIR}/addons/image/
    ${allegro5_SOURCE_DIR}/addons/ttf/
    ${allegro5_SOURCE_DIR}/addons/font/
    ${allegro5_SOURCE_DIR}/addons/color/
    ${allegro5_SOURCE_DIR}/addons/primitives/
)

# Necessary to export the dirs (?)
set (ALLEGRO_SOURCE_DIR ${allegro5_SOURCE_DIR})
set (ALLEGRO_BINARY_DIR ${allegro5_BINARY_DIR})

add_library(allegro-meta INTERFACE)
target_include_directories(allegro-meta INTERFACE ${ALLEGRO_INCLUDE_DIRS})
target_link_libraries(
    allegro-meta
INTERFACE
    allegro
    allegro_primitives
    allegro_font
    allegro_color
    allegro_image
    allegro_ttf
)
