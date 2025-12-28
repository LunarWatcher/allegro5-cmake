# Allegro FetchContent patch

This repo contains a patch for [liballegro](https://github.com/liballeg/allegro5) so it's compatible with FetchContent and conan or other FetchContent dependencies. Upstream allegro _can_ be used with FetchContent out of the box, but it struggles when the other third party dependencies are also sourced from FetchContent or other sources. At the time of this repo being separated from the game I use allegro for, allegro did not work directly without a patch at all, for reasons I didn't bother investigating.

Prior to this, the dependency-related issues are purely due to minor differences in how the libraries are sourced, and it's therefore patchable.

The patch rips out many bundled checks in allegro's CMake file to get around case differences between FetchContent/Conan and the built-in `find*` for some of the dependencies allegro uses. The patch _could_ change the names to what Conan/FetchContent creates, but this approach makes it slightly easier to assume specific defaults.

This patch has been verified with the following conan settings:
```python
def requirements(self):
    self.requires("freetype/2.13.3")
    self.requires("libpng/1.6.53")
    # This is required for ZLIB_INCLUDE_DIRS to be populated
    self.requires("zlib/1.3.1")

def configure(self):
    pass
```

And the following CMake settings:
```cmake
set(WANT_SHADERS_GL ON CACHE STRING "" FORCE)
set(WANT_MEMFILE OFF CACHE STRING "" FORCE)
set(WANT_PHYSFS OFF CACHE STRING "" FORCE)
set(WANT_NATIVE_DIALOG OFF CACHE STRING "" FORCE)
set(WANT_VIDEO OFF CACHE STRING "" FORCE)
set(WANT_IMAGE_WEBP OFF CACHE STRING "" FORCE)
# This line is important; do not turn FreeImage on
# See also https://archlinux.org/todo/drop-freeimage/
# TL;DR: freeimage is unmaintained and full of vulnerabilities. Use the other image libraries directly instead.
set(WANT_IMAGE_FREEIMAGE OFF CACHE STRING "" FORCE)
# Unnecessary stuff 
set(WANT_DOCS OFF)
set(WANT_DEMO OFF)
set(WANT_EXAMPLES OFF)
set(WANT_TESTS OFF)
set(WANT_POPUP_EXAMPLES OFF)
# This works around a bug where WANT_ALLOW_SSE defaulted to "ON", resulting in the invalid compiler flag -mON
# This may be fixed upstream depending on the source of the bug
if(CMAKE_SYSTEM_PROCESSOR MATCHES "i.86" OR CMAKE_SYSTEM_PROCESSOR MATCHES "x86|AMD64")
    # I want avx, so I turned it on here. you can just force this off
    set (WANT_ALLOW_SSE "avx2")
else()
    # This clause is unverified. It may need to be be "off" (a string) instead.
    # If you have hardware triggering this, please consider opening an issue that either confirms or denies that OFF works
    # so this can be properly confirmed.
    set (WANT_ALLOW_SSE OFF)
endif()

# My game does not include audio, because audio is hard, so I disabled it. If you need audio, further patching will likely be required
set(WANT_AUDIO OFF)
```

Note that most of the extra FreeImage formats have been disabled as I have no use for them. Improvements to the patch are welcome.

## Usage
```cmake
FetchContent_Declare(allegro5-patch
    GIT_REPOSITORY https://github.com/LunarWatcher/allegro5-cmake
    # Optional but recommended:
    # GIT_TAG <commit>
)
FetchContent_MakeAvailable(allegro5-patch)

...
target_link_libraries(
    your_thing
PUBLIC
    # This is a meta target added by the patch. It's equivalent to:
    # allegro
    # allegro_primitives
    # allegro_font
    # allegro_color
    # allegro_image
    # allegro_ttf
    #
    # You can still use these libraries directly if you need a different set of
    # packages than what the meta target provides. Note that because allegro
    # is jank, you'll still need to declare the headers yourself. Copy 
    # and modify ALLEGRO_INCLUDE_DIRS as needed.
    # Making your own interface target is strongly recommended in this scenario,
    # because there's so much patching to do
    allegro-meta
)
```

This has been tested on a grand total of one project, so it likely has many severe problems. PRs are welcome. Preferably, some of these changes should be improved and upstreamed directly into allegro so these patches become redundant, but I don't know how to do that without breaking current builds.

The patch could also be upstreamed to Conan along with additional patches so allegro works directly from conan, rather than splitting dependencies up.

For a demonstration of this patch being used, see https://github.com/LunarWatcher/nyaa

## Exports

### Targets

* `allegro-meta`: as described, INTERFACE target that links allegro with primtives, font, color, image, and ttf

### Variables

* `ALLEGRO_SOURCE_DIR`: `allegro5_SOURCE_DIR` from the embedded FetchContent call to allegro
* `ALLEGRO_BINARY_DIR`: `allegro5_BINARY_DIR` from the embedded FetchContent call to allegro
* `ALLEGRO_INCLUDE_DIRS`: All the include dirs required for `allegro-meta`. Can also be used standalone should you need it

## Versioning

This patch repo is versionless because it's too much effort to bother. If you want to pin allegro5, pin this repo to whatever commit you want. Old versions are not actively maintained and may break; if you need to repatch an older version, fork the repo.

