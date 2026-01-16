cmake_minimum_required(VERSION 3.16)
project("Lua Windows" LANGUAGES CXX ASM_MASM)

file(GLOB CPP_SOURCES "Src/*.cpp")
file(GLOB HEADER_SOURCES "Include/*.h")
set(PRECOMPILED_HEADER Include/lwindows.h)
add_library(lwindows SHARED ${HEADER_SOURCES} ${CPP_SOURCES})

target_compile_features(lwindows PRIVATE cxx_std_20)
target_precompile_headers(lwindows PRIVATE ${PRECOMPILED_HEADER})

target_link_libraries(lwindows PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/lua/lib/lua.lib")
target_include_directories(lwindows PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/lua/include")