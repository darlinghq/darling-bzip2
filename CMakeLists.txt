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

add_definitions(-D_FILE_OFFSET_BITS=64 -DPRIVATE)

set(bzip2_sources
	bzip2/blocksort.c
	bzip2/bzlib.c
	bzip2/compress.c
	bzip2/crctable.c
	bzip2/decompress.c
	bzip2/huffman.c
	bzip2/randtable.c
)

if (TARGET_x86_64)
	add_darling_object_library(bz264 bzip2/x86_64/crc32vec.S)
	set_target_properties(bz264 PROPERTIES COMPILE_FLAGS "-arch x86_64")
else (TARGET_x86_64)
	add_darling_object_library(bz264 empty.c)
endif (TARGET_x86_64)

set(DYLIB_INSTALL_NAME "/usr/lib/libbz2.1.0.dylib")
set(DYLIB_COMPAT_VERSION "1.0.0")
set(DYLIB_CURRENT_VERSION "1.0.5")

add_darling_library(bz2 SHARED ${bzip2_sources} $<TARGET_OBJECTS:bz264>)
make_fat(bz2)
target_link_libraries(bz2 system)
set_target_properties(bz2 PROPERTIES OUTPUT_NAME "bz2.1.0")
install(TARGETS bz2 DESTINATION libexec/darling/usr/lib)
InstallSymlink("libbz2.1.0.dylib" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/libbz2.dylib")
InstallSymlink("libbz2.1.0.dylib" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/libbz2.1.0.5.dylib")

if (NOT DARLING_NO_EXECUTABLES)
	include(darling_exe)
	include(InstallSymlink)

	add_darling_executable(bzip2 bzip2/bzip2.c)
	target_link_libraries(bzip2 bz2)

	install(TARGETS bzip2 DESTINATION libexec/darling/bin)
	InstallSymlink(bzip2 "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/bin/bzcat")
	InstallSymlink(bzip2 "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/bin/bunzip2")
endif (NOT DARLING_NO_EXECUTABLES)

set(man1
	bzip2/bzdiff.1
	bzip2/bzgrep.1
	bzip2/bzip2.1
	bzip2/bzmore.1
)

install(FILES ${man1} DESTINATION libexec/darling/usr/share/man/man1)
