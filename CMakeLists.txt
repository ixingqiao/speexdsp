cmake_minimum_required(VERSION 3.1.3)
project(speexdsp)

# ����Ĭ�ϵĹ�������ΪRelease
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

# ����CMakeģ����֧��GNU��׼�İ�װĿ¼
include(GNUInstallDirs)

# ���ӱ���ѡ��
option(BUILD_TESTS "Build test programs" ON)
option(USE_AVX "Use AVX instructions for performance optimization" OFF)
option(USE_SSE "Use SSE instructions for performance optimization" OFF)

# ��鲢����AVX��SSEָ�֧��
if(USE_AVX)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        add_compile_options(-mavx)
    elseif(MSVC)
        add_compile_options(/arch:AVX)
    endif()
elseif(USE_SSE)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        add_compile_options(-msse)
    elseif(MSVC)
        add_compile_options(/arch:SSE2)
    endif()
endif()

# ���ñ�����ѡ��
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra -pedantic)
elseif(MSVC)
    add_compile_options(/W4)
endif()

# ��鲢����int16_t���͵Ĵ�С
include(CheckTypeSize)
check_type_size("int16_t" SIZEOF_INT16)

# ����Դ�����ͷ�ļ�·��
set(SPEEXDSP_FOLDER ${CMAKE_CURRENT_SOURCE_DIR})
set(SPEEXDSP_SOURCE ${SPEEXDSP_FOLDER}/libspeexdsp)

# ������Ҫ�ĺ��ͷ�ļ�
set(SIZE16 "int16_t")
set(USIZE16 "uint16_t")
set(SIZE32 "int32_t")
set(USIZE32 "uint32_t")
set(EXPORT "__attribute__((visibility(\"default\")))")
set(FLOATING_POINT 1)
set(USE_SMALLFT 1)

# �����ļ�����������speexdsp_config_types.h��config.h
if(HAVE_SYS_TYPES_H)
    set(INCLUDE_STDINT "#include <sys/types.h>")
endif()
if(HAVE_STDINT_H)
    set(INCLUDE_STDINT "#include <stdint.h>")
endif()
if(HAVE_STDDEF_H)
    set(INCLUDE_STDINT "#include <inttypes.h>")
endif()

configure_file(
        ${SPEEXDSP_FOLDER}/include/speex/speexdsp_config_types.h.in
        ${SPEEXDSP_FOLDER}/include/speex/speexdsp_config_types.h
)

configure_file(config.h.cmake ${SPEEXDSP_FOLDER}/config.h)

# ����Ĭ�ϵ������ļ�·��Ϊ��ǰ·���µ�lib�ļ���
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)

# ���ӿ�
add_library(speexdsp STATIC ${SPEEXDSP_SOURCE}/resample.c
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

# ��Բ�ͬƽ̨ѡ�����ӵ���ѧ��
if(UNIX)
    find_library(M_LIBRARY m)
    target_link_libraries(speexdsp PRIVATE ${M_LIBRARY})
endif()

# ���ÿ��ͷ�ļ�·��
target_include_directories(speexdsp PUBLIC ${SPEEXDSP_FOLDER}/include PRIVATE ${SPEEXDSP_FOLDER} ${SPEEXDSP_SOURCE} ${SPEEXDSP_FOLDER}/include ${SPEEXDSP_FOLDER}/include/speex)

# ���ñ���ѡ��
target_compile_options(speexdsp PRIVATE "-DHAVE_CONFIG_H")

# ���Ӳ��Գ���
if(BUILD_TESTS)
    file(GLOB TEST_SOURCES "${SPEEXDSP_SOURCE}/test*.c")
    foreach(TEST_SOURCE ${TEST_SOURCES})
        # ��ȡ���Գ��������
        get_filename_component(TEST_NAME ${TEST_SOURCE} NAME_WE)
        # ���Ӳ��Գ���Ĺ�������
        add_executable(${TEST_NAME} ${TEST_SOURCE})
        target_link_libraries(${TEST_NAME} speexdsp)
    endforeach()
endif()

# ��װ����
install(TARGETS speexdsp
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY ${SPEEXDSP_FOLDER}/include/speex DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} FILES_MATCHING PATTERN "*.h")

# ��ӡ������Ϣ
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Build tests: ${BUILD_TESTS}")
message(STATUS "Use AVX: ${USE_AVX}")
message(STATUS "Use SSE: ${USE_SSE}")