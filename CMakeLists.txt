project(darling-bzip2)

cmake_minimum_required(VERSION 3.13)
enable_language(C ASM)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

# The SSE options are for the Intel slices only
darling_archs(_sse_archs FAMILY x86)
darling_xarch_flags(_sse_flags ARCHS ${_sse_archs} OPTIONS -msse -msse2 -msse3)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${_sse_flags}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w -nostdinc")
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

# The vectorised CRC is assembly, one file per architecture (bzlib.c declares it for x86_64 and
# arm64): each of those slices gets an object library of its own, compiled for that slice only.
set(bz2_crc_objects "")
foreach (arch IN LISTS DARLING_ARCHS)
	if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/bzip2/${arch}/crc32vec.S")
		add_darling_object_library(bz2_crc_${arch} ARCHS ${arch} SOURCES bzip2/${arch}/crc32vec.S)
		list(APPEND bz2_crc_objects $<TARGET_OBJECTS:bz2_crc_${arch}>)
	endif ()
endforeach ()

set(DYLIB_INSTALL_NAME "/usr/lib/libbz2.1.0.dylib")
set(DYLIB_COMPAT_VERSION "1.0.0")
set(DYLIB_CURRENT_VERSION "1.0.5")

add_darling_library(bz2 SHARED ${bzip2_sources} ${bz2_crc_objects})
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
	InstallSymlink(../../bin/bzip2 "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/bin/bzip2")
	InstallSymlink(../../bin/bzip2 "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/bin/bzcat")
	InstallSymlink(../../bin/bzip2 "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/bin/bunzip2")
endif (NOT DARLING_NO_EXECUTABLES)

set(man1
	bzip2/bzdiff.1
	bzip2/bzgrep.1
	bzip2/bzip2.1
	bzip2/bzmore.1
)

install(FILES ${man1} DESTINATION libexec/darling/usr/share/man/man1)
