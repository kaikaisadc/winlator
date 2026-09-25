cmake_minimum_required(VERSION 3.22.1)

project(Winlator)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -Wno-unused-function -Wimplicit-function-declaration")

add_subdirectory(OpenXR-SDK)
add_subdirectory(patchelf)

# Add Winlator shared library
add_library(winlator SHARED
        xr/engine.c
        xr/framebuffer.c
        xr/input.c
        xr/main.c
        xr/math.c
        xr/renderer.c
        winlator/drawable.c
        winlator/gpu_image.c
        winlator/sysvshared_memory.c
        winlator/xconnector_epoll.c
        winlator/alsa_client.c
        winlator/patchelf_wrapper.cpp
        winlator/vulkan.cpp
)

target_link_libraries(winlator
        log
        android
        jnigraphics
        openxr_loader
        aaudio
        EGL
        GLESv2
        GLESv3
)


# ---- EVSHIM (not built here) ------------------------------------------------
# winlator/evshim.c is a multi-controller virtual-joystick shim. Its SDL symbols
# are resolved at runtime via dlopen("libSDL2-2.0.so.0") + dlsym, so it only
# needs SDL2 headers to compile and must NOT link against SDL2.
#
# The app does not build or bundle a usable evshim: the live copy ships inside
# imagefs.txz and is LD_PRELOADed from the imagefs lib dir by
# BionicProgramLauncherComponent. The old "add_subdirectory(sdl2)" step relied
# on a submodule whose URL was never recorded in .gitmodules (the gitlink is
# orphaned and its commit is unrecoverable), so a clean clone could not build.
# evshim.c is kept as the reference implementation for the imagefs artifact.
