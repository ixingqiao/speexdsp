cmake_minimum_required(VERSION 3.1.3)
project(speexdsp)

# 设置默认的构建类型为Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

# 添加CMake模块以支持GNU标准的安装目录
include(GNUInstallDirs)

# 添加编译选项
option(BUILD_TESTS "Build test programs" OFF)
option(USE_AVX "Use AVX instructions for performance optimization" OFF)
option(USE_SSE "Use SSE instructions for performance optimization" OFF)

# 检查并启用AVX和SSE指令集支持
if(USE_AVX)
    add_compile_options(-mavx)
elseif(USE_SSE)
    add_compile_options(-msse)
endif()

# 设置编译器选项
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra -pedantic)
elseif(MSVC)
    add_compile_options(/W4)
endif()

# 检查并设置int16_t类型的大小
include(CheckTypeSize)
check_type_size("int16_t" SIZEOF_INT16)

# 定义源代码和头文件路径
set(SPEEXDSP_FOLDER ${CMAKE_CURRENT_SOURCE_DIR})
set(SPEEXDSP_SOURCE ${SPEEXDSP_FOLDER}/libspeexdsp)

# 定义需要的宏和头文件
set(SIZE16 "int16_t")
set(USIZE16 "uint16_t")
set(SIZE32 "int32_t")
set(USIZE32 "uint32_t")
set(EXPORT "__attribute__((visibility(\"default\")))")
set(FLOATING_POINT 1)
set(USE_SMALLFT 1)

# 配置文件，用于生成speexdsp_config_types.h和config.h
if(HAVE_SYS_TYPES_H)
    set(INCLUDE_STDINT "#include <sys/types.h>")
elseif(HAVE_STDINT_H)
    set(INCLUDE_STDINT "#include <stdint.h>")
elseif(HAVE_STDDEF_H)
    set(INCLUDE_STDINT "#include <inttypes.h>")
endif()

configure_file(
        ${SPEEXDSP_FOLDER}/include/speex/speexdsp_config_types.h.in
        ${SPEEXDSP_FOLDER}/include/speex/speexdsp_config_types.h
)

configure_file(config.h.cmake ${SPEEXDSP_FOLDER}/config.h)

# 设置默认的生成文件路径为当前路径下的lib文件夹
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)

# 添加库
add_library(libspeexdsp STATIC ${SPEEXDSP_SOURCE}/resample.c
        ${SPEEXDSP_SOURCE}/buffer.c
        ${SPEEXDSP_SOURCE}/fftwrap.c
        ${SPEEXDSP_SOURCE}/filterbank.c
        ${SPEEXDSP_SOURCE}/jitter.c
        ${SPEEXDSP_SOURCE}/kiss_fft.c
        ${SPEEXDSP_SOURCE}/kiss_fftr.c
        ${SPEEXDSP_SOURCE}/mdf.c
        ${SPEEXDSP_SOURCE}/preprocess.c
        ${SPEEXDSP_SOURCE}/scal.c
        ${SPEEXDSP_SOURCE}/smallft.c)

# 设置库的头文件路径
target_include_directories(libspeexdsp PUBLIC ${SPEEXDSP_FOLDER}/include PRIVATE ${SPEEXDSP_FOLDER} ${SPEEXDSP_SOURCE} ${SPEEXDSP_FOLDER}/include ${SPEEXDSP_FOLDER}/include/speex)

# 设置编译选项
target_compile_options(libspeexdsp PRIVATE "-DHAVE_CONFIG_H")

# 添加测试程序
if(BUILD_TESTS)
    file(GLOB TEST_SOURCES "${SPEEXDSP_SOURCE}/test*.c")
    foreach(TEST_SOURCE ${TEST_SOURCES})
        # 提取测试程序的名称
        get_filename_component(TEST_NAME ${TEST_SOURCE} NAME_WE)
        # 添加测试程序的构建规则
        add_executable(${TEST_NAME} ${TEST_SOURCE})
        target_link_libraries(${TEST_NAME} libspeexdsp)
    endforeach()
endif()

# 安装规则
install(TARGETS libspeexdsp
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${SPEEXDSP_FOLDER}/include/speex DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# 打印调试信息
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Build tests: ${BUILD_TESTS}")
message(STATUS "Use AVX: ${USE_AVX}")
message(STATUS "Use SSE: ${USE_SSE}")