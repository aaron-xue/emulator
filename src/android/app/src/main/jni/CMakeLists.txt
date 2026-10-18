# SPDX-FileCopyrightText: 2023 yuzu Emulator Project
# SPDX-License-Identifier: GPL-3.0-or-later

add_library(citron-android SHARED
    emu_window/emu_window.cpp
    emu_window/emu_window.h
    native.cpp
    native.h
    native_config.cpp
    android_settings.cpp
    game_metadata.cpp
    native_log.cpp
    android_config.cpp
    android_config.h
    native_input.cpp
)

set_property(TARGET citron-android PROPERTY IMPORTED_LOCATION ${FFmpeg_LIBRARY_DIR})

target_link_libraries(citron-android PRIVATE audio_core common core input_common frontend_common network Vulkan::Headers)
target_link_libraries(citron-android PRIVATE android camera2ndk EGL glad jnigraphics log)
if (ARCHITECTURE_arm64)
    target_link_libraries(citron-android PRIVATE adrenotools)
endif()

# The global -fvisibility=hidden set for Android (see root CMakeLists.txt) hides
# the bundled static archives' symbols, but it also hides our JNI entry points
# (the Java_* functions in native.cpp / native_input.cpp / native_log.cpp, which
# are not marked JNIEXPORT). Hidden visibility drops them from the dynamic symbol
# table, so dlsym() can't find them and the app dies with UnsatisfiedLinkError on
# the first native call. Restore default visibility for this target's own sources
# so the JNI functions are exported. The static archives keep their hidden
# visibility because they are compiled as separate translation units.
target_compile_options(citron-android PRIVATE -fvisibility=default)

set(CPACK_PACKAGE_EXECUTABLES ${CPACK_PACKAGE_EXECUTABLES} citron-android)
