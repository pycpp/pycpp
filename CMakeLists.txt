#  :copyright: (c) 2017-2018 Alex Huszagh.
#  :license: MIT, see licenses/mit.md for more details.

set(CMAKE_LEGACY_CYGWIN_WIN32 1)
cmake_minimum_required(VERSION 2.8)
project(pycpp CXX)
enable_testing()
enable_language(C ASM)

if(MSVC AND BUILD_SHARED_LIBS)
    cmake_minimum_required(VERSION 3.4)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

# FLAGS
# -----

# MODULES
option(BUILD_BLOOM "Build bloom filters library" ON)
option(BUILD_CIPHER "Build cipher library" ON)
option(BUILD_COLLECTIONS "Build collections library" ON)
option(BUILD_COMPRESSION "Build compression library" ON)
option(BUILD_CSV "Build CSV library" ON)
option(BUILD_CUCKOO "Build Cuckoo hash-table library" ON)
option(BUILD_DATETIME "Build datetime library" ON)
option(BUILD_FILESYSTEM "Build filesystem library" ON)
option(BUILD_HASHLIB "Build hash library" ON)
option(BUILD_JSON "Build JSON library" ON)
option(BUILD_KEYVALUE "Build key-value database library" ON)
option(BUILD_LATTICE "Build lattice (networking) library" ON)
option(BUILD_MATH "Build math library" ON)
option(BUILD_MYSQL "Build MySQL database library" ON)
option(BUILD_POSTGRES "Build PostgreSQL database library" ON)
option(BUILD_RE "Build regular expression library" ON)
option(BUILD_SQLITE "Build SQLite database library" ON)
option(BUILD_STREAM "Build stream library" ON)
option(BUILD_XML "Build XML library" ON)
option(USE_POLYMORPHIC_ALLOCATOR "Use polymorphic allocator as default allocator" ON)
option(USE_XXHASH "Use xxHash as default hash function" ON)

# DEPENDENCIES
option(USE_SYSTEM_BLOSC "Use system BLOSC installation" OFF)
option(USE_SYSTEM_BZIP2 "Use system BZIP2 installation" OFF)
option(USE_SYSTEM_LEVELDB "Use system LEVELDB installation" OFF)
option(USE_SYSTEM_LIBXML2 "Use system LIBXML2 installation" OFF)
option(USE_SYSTEM_LZMA "Use system LZMA2/XZZ installation" OFF)
option(USE_SYSTEM_MYSQL "Use system MySQL installation" OFF)
option(USE_SYSTEM_POSTGRES "Use system PostgreSQL installation" OFF)
option(USE_SYSTEM_RE2 "Use system RE2 installation" OFF)
option(USE_SYSTEM_SQLITE "Use system SQLite installation" OFF)
option(USE_SYSTEM_ZLIB "Use system ZLIB installation" OFF)

# WITH MODULE
option(WITH_CORE "Build (only) core library" OFF)
option(WITH_BLOOM "Build (only) bloom filters library" OFF)
option(WITH_CIPHER "Build (only) cipher library" OFF)
option(WITH_COLLECTIONS "Build (only) collections library" OFF)
option(WITH_COMPRESSION "Build (only) compression library" OFF)
option(WITH_CSV "Build (only) CSV library" OFF)
option(WITH_CUCKOO "Build (only) Cuckoo hash-table library" OFF)
option(WITH_DATETIME "Build (only) datetime library" OFF)
option(WITH_FILESYSTEM "Build (only) filesystem library" OFF)
option(WITH_HASHLIB "Build (only) hash library" OFF)
option(WITH_JSON "Build (only) JSON library" OFF)
option(WITH_KEYVALUE "Build (only) key-value database library" OFF)
option(WITH_LATTICE "Build (only) lattice library" OFF)
option(WITH_MATH "Build (only) math library" OFF)
option(WITH_MYSQL "Build (only) MySQL database library" OFF)
option(WITH_POSTGRES "Build (only) PostgreSQL database library" OFF)
option(WITH_RE "Build (only) regular expression library" OFF)
option(WITH_SQLITE "Build (only) SQLite database library" OFF)
option(WITH_STREAM "Build (only) stream library" OFF)
option(WITH_XML "Build (only) XML library" OFF)

# OPTIONS
option(BUILD_BENCHMARKS "Build pycpp library benchmarks" OFF)
option(BUILD_TESTS "Build pycpp library tests" OFF)
SET(CXX_STANDARD "11" CACHE STRING "Minimum C++ standard to support.")
SET(PYCPP_NAMESPACE "pycpp" CACHE STRING "Name for PyCPP namespace (empty for no namespace).")

# The following options are only for program developers.
option(USE_LIBCXX "Force the use of libcxx" OFF)

if(WITH_CORE OR WITH_BLOOM OR WITH_CIPHER OR WITH_COLLECTIONS OR WITH_COMPRESSION OR WITH_CSV OR WITH_CUCKOO OR WITH_DATETIME OR WITH_FILESYSTEM OR WITH_HASHLIB OR WITH_JSON OR WITH_KEYVALUE OR WITH_LATTICE OR WITH_MATH OR WITH_MYSQL OR WITH_RE OR WITH_SQLITE OR WITH_STREAM OR WITH_XML)
    set(BUILD_BLOOM ${WITH_BLOOM})
    set(BUILD_CIPHER ${WITH_CIPHER})
    set(BUILD_COLLECTIONS ${WITH_COLLECTIONS})
    set(BUILD_COMPRESSION ${WITH_COMPRESSION})
    set(BUILD_CSV ${WITH_CSV})
    set(BUILD_CUCKOO ${WITH_CUCKOO})
    set(BUILD_DATETIME ${WITH_DATETIME})
    set(BUILD_FILESYSTEM ${WITH_FILESYSTEM})
    set(BUILD_HASHLIB ${WITH_HASHLIB})
    set(BUILD_JSON ${WITH_JSON})
    set(BUILD_KEYVALUE ${WITH_KEYVALUE})
    set(BUILD_LATTICE ${WITH_LATTICE})
    set(BUILD_MATH ${WITH_MATH})
    set(BUILD_MYSQL ${WITH_MYSQL})
    set(BUILD_POSTGRES ${WITH_POSTGRES})
    set(BUILD_RE ${WITH_RE})
    set(BUILD_SQLITE ${WITH_SQLITE})
    set(BUILD_STREAM ${WITH_STREAM})
    set(BUILD_XML ${WITH_XML})
endif()

set(BUILD_CACHE ${BUILD_KEYVALUE})
set(BUILD_SQL BUILD_MYSQL OR BUILD_POSTGRES OR BUILD_SQLITE)

# C++ standards.
if (MSVC)
    if (CXX_STANDARD LESS "14")
        set(CXX_STANDARD 14)
    endif()
endif()
set(CMAKE_CXX_STANDARD ${CXX_STANDARD})

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -std=c++${CXX_STANDARD}")
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -std=c++${CXX_STANDARD}")
    if(USE_LIBCXX)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++ -lc++abi")
    endif()
elseif(MSVC)
    list(APPEND PYCPP_COMPILE_DEFINITIONS _CRT_SECURE_NO_WARNINGS _SCL_SECURE_NO_WARNINGS)
    # Don't add "-WX" since external projects should be free to have
    # warnings: only add after.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -wd4018 -wd4503 /std:c++${CXX_STANDARD}")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd /Od /Fd${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pdb")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT /Ox /Zi /Fd${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pdb")
    set(CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_MINSIZEREL} /MT /Zi /Fd${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pdb")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /MT /Fd${CMAKE_BINARY_DIR}/${PROJECT_NAME}.pdb")
endif()

set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -x assembler-with-cpp")

# STATIC/SHARED
if(BUILD_SHARED_LIBS)
    set(BUILD_STATIC_LIBS OFF)
else()
    set(BUILD_STATIC_LIBS ON)
    if(MINGW OR MSYS OR CYGWIN)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
    elseif(UNIX)
        set(CMAKE_POSITION_INDEPENDENT_CODE ON)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
    endif()
endif()

# Force -DUNICODE and -D_UNICODE for WIN32, use wide paths.
if(WIN32)
    list(APPEND PYCPP_COMPILE_DEFINITIONS UNICODE _UNICODE)
endif()

# INCLUDES
# --------

include(CheckIncludeFile)
CHECK_INCLUDE_FILE(sys/mman.h HAVE_SYS_MMAN_H)

# FUNCTIONS
# ---------

include(CheckFunctionExists)

CHECK_FUNCTION_EXISTS(explicit_bzero HAVE_EXPLICIT_BZERO)
CHECK_FUNCTION_EXISTS(memset_s HAVE_MEMSET_S)
CHECK_FUNCTION_EXISTS(memcpy_s HAVE_MEMCPY_S)
CHECK_FUNCTION_EXISTS(memmove_s HAVE_MEMMOVE_S)
CHECK_FUNCTION_EXISTS(posix_memalign HAVE_POSIX_MEMALIGN)
CHECK_FUNCTION_EXISTS(posix_fallocate HAVE_POSIX_FALLOCATE)
CHECK_FUNCTION_EXISTS(posix_fadvise HAVE_POSIX_FADVISE)
CHECK_FUNCTION_EXISTS(madvise HAVE_MADVISE)
CHECK_FUNCTION_EXISTS(mlock HAVE_MLOCK)
CHECK_FUNCTION_EXISTS(mmap HAVE_MMAP)
CHECK_FUNCTION_EXISTS(mmap HAVE_MPROTECT)
CHECK_FUNCTION_EXISTS(_wfopen HAVE_WFOPEN)
CHECK_FUNCTION_EXISTS(aligned_alloc HAVE_ALIGNED_ALLOC)

# CONFIGURATION
# -------------

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/pycpp/config.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/include/pycpp/config.h"
)

# EXTERNAL
# --------

include(ExternalProject)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

#
#   Add external project.
#
#   Takes optional N arguments after the name and path
#   to specify additional flags for the build sequence.
#
#   \param name             Name of external project
#   \param path             Path to source directory
#
macro(add_external_project name path)
    set(extra_args ${ARGN})
    # Create external project
    set(${name}_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${path})
    set(${name}_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/${path})
    ExternalProject_Add(${name}
        SOURCE_DIR "${${name}_SOURCE_DIR}"
        BINARY_DIR "${${name}_BINARY_DIR}"
        CMAKE_ARGS "-DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}"
        CMAKE_ARGS "-DCMAKE_C_FLAGS_DEBUG=${CMAKE_C_FLAGS_DEBUG}"
        CMAKE_ARGS "-DCMAKE_C_FLAGS_RELEASE=${CMAKE_C_FLAGS_RELEASE}"
                   "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}"
                   "-DCMAKE_CXX_FLAGS_DEBUG=${CMAKE_CXX_FLAGS_DEBUG}"
                   "-DCMAKE_CXX_FLAGS_RELEASE=${CMAKE_CXX_FLAGS_RELEASE}"
                   "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
                   "-DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}"
                   "-DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}"
                   "-DCMAKE_SYSTEM_PROCESSOR=${CMAKE_SYSTEM_PROCESSOR}"
                   "-DCMAKE_AR=${CMAKE_AR}"
                   "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}"
                   "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}"
                   "-DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER}"
                   "-DCMAKE_COMPILER_PREFIX=${CMAKE_COMPILER_PREFIX}"
                   "-DCMAKE_FIND_ROOT_PATH=${CMAKE_FIND_ROOT_PATH}"
                   ${extra_args}
       INSTALL_COMMAND ""
    )

endmacro(add_external_project)

#
#   Add external target to external project.
#
#   \param name             Name of external project
#   \param includedir       Path to include directory
#   \param libdir           Path to library directory
#   \param build_type       Build type {STATIC, SHARED}
#   \param external         Name of the external target
#   \param debug_suffix     Suffix for the library if using a debug build
#
macro(add_external_target name includedir libdir build_type external debug_suffix)
    # Configurations
    set(${name}_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/${libdir})

    # Create external library
    add_library(${name} ${build_type} IMPORTED)

    set(${name}_prefix "${${name}_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${CMAKE_${build_type}_LIBRARY_PREFIX}${name}")
    if(CMAKE_BUILD_TYPE MATCHES Debug)
        set(${name}_suffix "${debug_suffix}${CMAKE_${build_type}_LIBRARY_SUFFIX}")
    else()
        set(${name}_suffix "${CMAKE_${build_type}_LIBRARY_SUFFIX}")
    endif()
    set(${name}_LIBRARY "${${name}_prefix}${${name}_suffix}")

    # Find paths and set dependencies
    add_dependencies(${name} ${external})
    set(${name}_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/${includedir}")

    # Set interface properties
    set_target_properties(${name} PROPERTIES IMPORTED_LOCATION ${${name}_LIBRARY})
    set_target_properties(${name} PROPERTIES INCLUDE_DIRECTORIES ${${name}_INCLUDE_DIR})
endmacro(add_external_target)

# WINDOWS
# -------

# Need to specify proper exception handling for Windows.
if(MSVC)
    list(APPEND PYCPP_COMPILE_OPTIONS "/EHsc")
endif()

if(MSVC)
    list(APPEND PYCPP_LIBRARIES ws2_32.lib crypt32.lib kernel32.lib)
elseif(MINGW OR MSYS OR CYGWIN)
    list(APPEND PYCPP_LIBRARIES -lws2_32 -lcrypt32 -lkernel32)
endif()

# EXTERNAL PROJECTS
# -----------------

set(PYCPP_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}/include")
set(WARNINGS_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/third_party/warnings/src")
#set(MULTI_INDEX_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/third_party/multi_index/include")
#set(BRIGAND_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/third_party/multi_index/third_party/brigand/include")

list(APPEND PYCPP_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${WARNINGS_INCLUDE_DIRS}
#    ${MULTI_INDEX_INCLUDE_DIRS}
#    ${BRIGAND_INCLUDE_DIRS}
)

# TODO: restore
#if(BUILD_CACHE)
#    if(BUILD_KEYVALUE)
#        if(USE_SYSTEM_LEVELDB)
#            find_package(LEVELDB "1.0")
#        else()
#            find_package(LEVELDB "1.0")     # TODO: remove
## TODO: restore, change to leveldb
##            add_external_project(zlib_external third_party/zlib)
##            add_external_target(z third_party/zlib third_party/zlib STATIC zlib_external "")
##            list(APPEND PYCPP_DEPENDENCIES z)
##            set(Z_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/zlib" "${CMAKE_CURRENT_BINARY_DIR}/third_party/zlib")
##            set(LEVELDB_INCLUDE_DIRS ${Z_INCLUDE_DIR})
##            set(LEVELDB_LIBRARIES z)
##            set(LEVELDB_FOUND 1)
#        endif()
#
#        if(LEVELDB_FOUND)
#            list(APPEND PYCPP_LIBRARIES ${LEVELDB_LIBRARIES})
#            list(APPEND PYCPP_INCLUDE_DIRS ${LEVELDB_INCLUDE_DIRS})
#        else()
#            message(WARNING "Cannot find back-end for key-value database. Disabling...")
#            set(BUILD_KEYVALUE OFF)
#        endif()
#    endif()
#endif()
#
#if(BUILD_COMPRESSION)
#    # ZLIB
#    if(USE_SYSTEM_ZLIB)
#        find_package(ZLIB "1.0")
#    else()
#        add_external_project(zlib_external third_party/zlib)
#        add_external_target(z third_party/zlib third_party/zlib STATIC zlib_external "d")
#        list(APPEND PYCPP_DEPENDENCIES z)
#        set(Z_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/zlib" "${CMAKE_CURRENT_BINARY_DIR}/third_party/zlib")
#        set(ZLIB_INCLUDE_DIRS ${Z_INCLUDE_DIR})
#        set(ZLIB_LIBRARIES z)
#        set(ZLIB_FOUND 1)
#    endif()
#
#    if(ZLIB_FOUND)
#        list(APPEND PYCPP_COMPILE_DEFINITIONS HAVE_ZLIB=1)
#        list(APPEND PYCPP_LIBRARIES ${ZLIB_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${ZLIB_INCLUDE_DIRS})
#    endif()
#
#    # BZIP2
#    if(USE_SYSTEM_BZIP2)
#        find_package(BZIP2 "1.0")
#    else()
#        add_external_project(bzip2_external third_party/bzip2)
#        add_external_target(bz2 third_party/bzip2 third_party/bzip2 STATIC bzip2_external "")
#        list(APPEND PYCPP_DEPENDENCIES bz2)
#        set(BZIP2_INCLUDE_DIRS ${bz2_INCLUDE_DIR})
#        set(BZIP2_LIBRARIES bz2)
#        set(BZIP2_FOUND 1)
#    endif()
#
#    if(BZIP2_FOUND)
#        list(APPEND PYCPP_COMPILE_DEFINITIONS HAVE_BZIP2=1)
#        list(APPEND PYCPP_LIBRARIES ${BZIP2_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${BZIP2_INCLUDE_DIRS})
#    endif()
#
#    # XZ
#    if(USE_SYSTEM_LZMA)
#        find_package(LZMA "5.0")
#    else()
#        add_external_project(xz_external third_party/xz)
#        add_external_target(lzma third_party/xz/src/liblzma/api third_party/xz/src/liblzma/ STATIC xz_external "")
#        list(APPEND PYCPP_DEPENDENCIES lzma)
#        set(LZMA_INCLUDE_DIRS ${lzma_INCLUDE_DIR})
#        set(LZMA_LIBRARIES lzma)
#        set(LZMA_FOUND 1)
#    endif()
#
#    if(LZMA_FOUND)
#        list(APPEND PYCPP_COMPILE_DEFINITIONS HAVE_LZMA=1 LZMA_API_STATIC=1)
#        list(APPEND PYCPP_LIBRARIES ${LZMA_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${LZMA_INCLUDE_DIRS})
#    endif()
#
#    # BLOSC
#    list(APPEND BLOSC_FLAGS
#        "-DBUILD_SHARED=OFF"
#        "-DBUILD_TESTS=OFF"
#        "-DBUILD_BENCHMARKS=OFF"
#        "-DBLOSC_INSTALL=OFF"
#    )
#    if(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
#        if (WIN32)
#            list(APPEND BLOSC_FLAGS "-DCMAKE_C_FLAGS=-Wno-implicit-function-declaration -Wno-attributes")
#        else()
#            list(APPEND BLOSC_FLAGS "-DCMAKE_C_FLAGS=-Wno-implicit-function-declaration")
#        endif()
#    elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
#        list(APPEND BLOSC_FLAGS "-DCMAKE_C_FLAGS=-Wno-implicit-function-declaration -Wno-shift-negative-value")
#    elseif(MSVC)
#        list(APPEND BLOSC_FLAGS "-DCMAKE_CXX_FLAGS=-wd4101 -wd4146 -wd4244 -wd4530 -wd4577 -wd4996")
#    endif()
#    list(APPEND BLOSC_FLAGS "-DBUILD_SHARED=${BUILD_SHARED_LIBS} -DBUILD_STATIC=${BUILD_STATIC_LIBS}")
#
#    if(USE_SYSTEM_BLOSC)
#        find_package(BLOSC "1.0")
#    else()
#        add_external_project(blosc_external third_party/c-blosc ${BLOSC_FLAGS})
#        add_external_target(blosc third_party/c-blosc/blosc third_party/c-blosc/blosc STATIC blosc_external "")
#        list(APPEND PYCPP_DEPENDENCIES blosc)
#        set(BLOSC_INCLUDE_DIRS ${blosc_INCLUDE_DIR})
#        set(BLOSC_LIBRARIES blosc)
#        set(BLOSC_FOUND 1)
#    endif()
#
#    if(BLOSC_FOUND)
#        list(APPEND PYCPP_COMPILE_DEFINITIONS HAVE_BLOSC=1)
#        list(APPEND PYCPP_LIBRARIES ${BLOSC_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${BLOSC_INCLUDE_DIRS})
#    endif()
#endif()
#
#if(BUILD_JSON)
#    set(RAPIDJSON_INCLUDE_DIRS third_party/rapidjson/include)
#    list(APPEND PYCPP_INCLUDE_DIRS ${RAPIDJSON_INCLUDE_DIRS})
#endif()
#
#if(BUILD_RE)
#    if(USE_SYSTEM_RE2)
#        find_package(RE2)
#    else()
#        add_external_project(re2_external third_party/re2)
#        add_external_target(re2 third_party/re2 third_party/re2 STATIC re2_external "")
#        list(APPEND PYCPP_DEPENDENCIES re2)
#        set(RE2_INCLUDE_DIRS ${re2_INCLUDE_DIR})
#        set(RE2_LIBRARIES re2)
#        set(RE2_FOUND 1)
#    endif()
#
#    if(RE2_FOUND)
#        list(APPEND PYCPP_LIBRARIES ${RE2_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${RE2_INCLUDE_DIRS})
#    endif()
#endif()
#
#if(BUILD_XML)
#    if(USE_SYSTEM_LIBXML2)
#        find_package(LIBXML2)
#    else()
#        add_external_project(xml2_external third_party/libxml2)
#        add_external_target(xml2 third_party/libxml2/include third_party/libxml2 STATIC xml2_external "")
#        list(APPEND PYCPP_DEPENDENCIES xml2)
#        set(LIBXML2_INCLUDE_DIRS ${xml2_INCLUDE_DIR})
#        set(LIBXML2_LIBRARIES xml2)
#        set(LIBXML2_FOUND 1)
#    endif()
#
#    if(LIBXML2_FOUND)
#        list(APPEND PYCPP_COMPILE_DEFINITIONS LIBXML_STATIC=1)
#        list(APPEND PYCPP_LIBRARIES ${LIBXML2_LIBRARIES})
#        list(APPEND PYCPP_INCLUDE_DIRS ${LIBXML2_INCLUDE_DIRS})
#    endif()
#endif()

if (BUILD_TESTS)
    if (NOT MINGW)
        find_package(Threads)
    endif()

    if(NOT TARGET gtest)
        add_external_project(googletest_external third_party/googletest)
        add_external_target(gtest third_party/googletest/googletest/include third_party/googletest/googlemock/gtest STATIC googletest_external "")
        add_external_target(gtest_main third_party/googletest/googletest/include third_party/googletest/googlemock/gtest STATIC googletest_external "")
    endif()
endif()

if (BUILD_BENCHMARKS)
    if (NOT TARGET benchmark)
        add_external_project(benchmark_external third_party/benchmark)
        add_external_target(benchmark third_party/benchmark/include third_party/benchmark/src STATIC benchmark_external "")
        include_directories(${benchmark_INCLUDE_DIR})
    endif()
endif()

# SOURCES
# -------

set(PYCPP_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")
set(PYCPP_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

#
#   Append files to a list, prefixing a path.
#
#   Takes optional N arguments after the local path
#   to specify headers to add.
#
#   \param path         Directory of file list.
#   \param target       Output variable name.
#
macro(prefix_path path target)
    # Store the extra args
    set(filelist ${ARGN})

    # Prefix the paths
    foreach(filename ${filelist})
        list(APPEND ${target} "${path}/${filename}")
    endforeach(filename)

    # Add them to the parent scope.
    set(${target} "${${target}}" PARENT_SCOPE)

endmacro(prefix_path)

#
#   Add headers from local path.
#
#   Takes optional N arguments after the local path
#   to specify headers to add.
#
#   \param local_path   Path of directory relative to "${PYCPP_SOURCE_DIR}".
#
macro(add_headers)
    set(headers ${ARGN})
    prefix_path("${PYCPP_SOURCE_DIR}/pycpp/${module}" PYCPP_HEADER_FILES ${headers})
endmacro(add_headers)

#
#   Add sources from local path.
#
#   Takes optional N arguments after the local path
#   to specify sources to add.
#
macro(add_sources)
    set(sources ${ARGN})
    prefix_path("${PYCPP_SOURCE_DIR}/pycpp/${module}" PYCPP_SOURCE_FILES ${sources})
endmacro(add_sources)

#
#   Add tests from local path.
#
#   Takes optional N arguments after the local path
#   to specify tests to add.
#
macro(add_tests)
    set(tests ${ARGN})
    prefix_path("${PYCPP_SOURCE_DIR}/test/${module}" PYCPP_TEST_FILES ${tests})
endmacro(add_tests)

set(PYCPP_HEADER_FILES "${PYCPP_BINARY_DIR}/include/pycpp/config.h")
set(PYCPP_TEST_FILES "${PYCPP_SOURCE_DIR}/test/main.cc")

set(PYCPP_MODULES
    "preprocessor"
    "system"
    "stl"
    "sfinae"
    "adaptor"
    "intrusive"
    "allocator"
    "context"
    "utility"
    "lexical"
)

# TODO: append to modules based on optional modules...

foreach(module ${PYCPP_MODULES})
    # Configure and include the module header.
    configure_file(
      "${CMAKE_CURRENT_SOURCE_DIR}/pycpp/${module}/module.h.in"
      "${PYCPP_BINARY_DIR}/include/pycpp/${module}.h"
    )
    list(APPEND PYCPP_HEADER_FILES "${PYCPP_BINARY_DIR}/include/pycpp/${module}.h")

    # Get rest of source and test files.
    add_subdirectory(pycpp/${module})
    add_subdirectory(test/${module})
endforeach()

# LIBRARY
# -------

# Only suppress all warnings for our project.
if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -WX")
endif()

if(${CMAKE_VERSION} VERSION_LESS "3.1.0")
    # No `target_sources`, improvise
    add_library(pycpp ${PYCPP_SOURCE_FILES})
else()
    add_library(pycpp "")
    target_sources(pycpp
        PRIVATE ${PYCPP_SOURCE_FILES}
        PUBLIC ${PYCPP_HEADER_FILES}
    )
endif()

# INTERFACE
# ---------

if(PYCPP_COMPILE_DEFINITIONS)
    target_compile_definitions(pycpp PUBLIC ${PYCPP_COMPILE_DEFINITIONS})
endif()

if(PYCPP_COMPILE_OPTIONS)
    target_compile_options(pycpp PUBLIC ${PYCPP_COMPILE_OPTIONS})
endif()

if(PYCPP_DEPENDENCIES)
    add_dependencies(pycpp ${PYCPP_DEPENDENCIES})
endif()

if(PYCPP_INCLUDE_DIRS)
    target_include_directories(pycpp PUBLIC ${PYCPP_INCLUDE_DIRS})
endif()

if(PYCPP_LIBRARIES)
    target_link_libraries(pycpp PUBLIC ${PYCPP_LIBRARIES})
endif()

list(APPEND PYCPP_LIBRARIES pycpp)

# TESTS
# -----

if(BUILD_TESTS)
    include_directories(${gtest_INCLUDE_DIR})

    add_executable(pycpp_tests ${PYCPP_TEST_FILES})
    target_link_libraries(pycpp_tests
        gtest
        gtest_main
        ${CMAKE_THREAD_LIBS_INIT}
        ${PYCPP_LIBRARIES}
    )
    # Need to specify proper exception handling for Windows.
    if(MSVC)
        set_target_properties(pycpp_tests PROPERTIES
            COMPILE_OPTIONS "/EHsc"
        )
    endif()

    add_test(NAME pycpp_tests
        COMMAND pycpp_tests
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )

    add_dependencies(pycpp_tests ${PYCPP_DEPENDENCIES} gtest gtest_main)
endif()

# BENCHMARKS
# ----------

# TODO: restore....
#set(BENCHMARK_FILES
#    bench/lexical.cc
#)
#
#if(BUILD_BENCHMARKS)
#    set(BENCHMARK_LIBRARIES benchmark ${CMAKE_THREAD_LIBS_INIT})
#    if(MSVC)
#        list(APPEND BENCHMARK_LIBRARIES shlwapi.lib)
#    elseif(WIN32)
#        list(APPEND BENCHMARK_LIBRARIES -lshlwapi)
#    endif()
#
#    foreach(bench ${BENCHMARK_FILES})
#        get_filename_component(basename ${bench} NAME_WE)
#        add_executable("${basename}_bench" ${bench})
#        target_link_libraries("${basename}_bench"
#            ${BENCHMARK_LIBRARIES}
#            ${PYCPP_LIBRARIES}
#        )
#    endforeach()
#endif()

# INSTALL
# -------

if(MSVC)
    if(CMAKE_CL_64)
        set(MSVC_FOLDER_PREFIX x64)
    else()
        set(MSVC_FOLDER_PREFIX Win32)
    endif()

    install(TARGETS pycpp
        LIBRARY DESTINATION "lib/${MSVC_FOLDER_PREFIX}/\${CMAKE_INSTALL_CONFIG_NAME}"
        ARCHIVE DESTINATION "lib/${MSVC_FOLDER_PREFIX}/\${CMAKE_INSTALL_CONFIG_NAME}"
        RUNTIME DESTINATION "bin/${MSVC_FOLDER_PREFIX}/\${CMAKE_INSTALL_CONFIG_NAME}"
    )
else()
    install(TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
    )
endif()

install(DIRECTORY PYCPP_NAMESPACE
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)
