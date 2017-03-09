project(darling-bzip2)

cmake_minimum_required(VERSION 2.4.0)
enable_language(C ASM)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse -msse2 -msse3 -w -nostdinc")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D__DARWIN_UNIX03 -fPIC -w")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -nostdlib")
#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/)
include_directories(${DARLING_TOP_DIRECTORY}/platform-include)

add_definitions(-D_FILE_OFFSET_BITS=64)

set(bzip2_sources
	blocksort.c
	bzlib.c
	compress.c
	crctable.c
	decompress.c
	huffman.c
	randtable.c
)

set(DYLIB_INSTALL_NAME "/usr/lib/libbz2.1.0.dylib")
set(DYLIB_COMPAT_VERSION "1.0.0")
set(DYLIB_CURRENT_VERSION "1.0.5")

add_darling_library(bz2 SHARED ${bzip2_sources})
make_fat(bz2)
target_link_libraries(bz2 system)
set_target_properties(bz2 PROPERTIES OUTPUT_NAME "bz2.1.0")
install(TARGETS bz2 DESTINATION libexec/darling/usr/lib)
InstallSymlink("libbz2.1.0.dylib" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/libbz2.dylib")

if (NOT DARLING_NO_EXECUTABLES)
	include(darling_exe)
	include(InstallSymlink)

	add_darling_executable(bzip2 bzip2.c)
	target_link_libraries(bzip2 bz2)

	install(TARGETS bzip2 DESTINATION libexec/darling/bin)
	install(FILES bzcat bunzip2 DESTINATION libexec/darling/bin)
endif (NOT DARLING_NO_EXECUTABLES)

set(man1
	bzdiff.1
	bzgrep.1
	bzip2.1
	bzmore.1
)

install(FILES ${man1} DESTINATION libexec/darling/usr/share/man/man1)
