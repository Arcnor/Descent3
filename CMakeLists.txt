# CMake compatibility issues: don't modify this, please!
CMAKE_MINIMUM_REQUIRED( VERSION 2.4.6 )

MARK_AS_ADVANCED(CMAKE_BACKWARDS_COMPATIBILITY)
# allow more human readable "if then else" constructs
SET( CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE )

if (${CMAKE_TOOLCHAIN_FILE} MATCHES ".*vcpkg.cmake.*")
	message(STATUS "Building with vcpkg toolchain.")
	set(USING_VCPKG ON)
endif()

# TODO: Apple Silicon (arm64) later
SET( CMAKE_OSX_ARCHITECTURES x86_64 CACHE STRING "" FORCE)


PROJECT(Descent3)

IF (USING_VCPKG)
	FIND_PACKAGE(SDL2 CONFIG REQUIRED)
	SET(SDL_LIBRARY $<TARGET_NAME_IF_EXISTS:SDL2::SDL2main> $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>)
ELSE ()
	FIND_PACKAGE(SDL REQUIRED)
	MESSAGE( "SDL Include Dir is " ${SDL_INCLUDE_DIR} )
ENDIF ()

IF (UNIX)
SET (D3_GAMEDIR "~/Descent3/")

  SET(CMAKE_CXX_COMPILER "g++")
  SET(CMAKE_CXX_FLAGS "-O0 -g -Wno-write-strings -Wno-multichar -Wno-address-of-temporary")
  SET(CMAKE_C_FLAGS "-O0 -g")
  SET(CMAKE_C_COMPILER "gcc")
  IF (NOT APPLE)
    LIST(APPEND CMAKE_CXX_FLAGS "-m32")
    LIST(APPEND CMAKE_C_FLAGS "-m32")
  ENDIF()
ENDIF()

IF (UNIX AND NOT APPLE)
  MESSAGE("Building for Linux")

  ADD_DEFINITIONS( -D_DEBUG -D__LINUX__ -DLINUX -D_MAX_PATH=260 -D_MAX_FNAME=256 -D_REENRANT -DMONO -D__32BIT__ -DHAVEALLOCA_H -D_USE_OGL_ACTIVE_TEXTURES)
  SET(PLATFORM_INCLUDES "lib/linux" ${SDL_INCLUDE_DIR} )
ENDIF()

IF (APPLE)
	SET(D3_GAMEDIR "~/Descent3X")	
  MESSAGE("Building for MAC OSX")
  ADD_DEFINITIONS( -DMONO -D_DEBUG -D__LINUX__ -DLINUX -D_MAX_PATH=260 -D_MAX_FNAME=256 -D_REENRANT -DMACOSX=1 -D_USE_OGL_ACTIVE_TEXTURES)
  SET(PLATFORM_INCLUDES "lib/linux" ${SDL_INCLUDE_DIR} "/usr/X11/include" )
ENDIF()

IF (WIN32)
	if(NOT CMAKE_SIZEOF_VOID_P EQUAL 4)
		message(FATAL_ERROR "Descent 3 needs 32 bit compilation on Windows for now")
	endif()

	SET(CMAKE_FIND_LIBRARY_PREFIXES "")
	SET(CMAKE_FIND_LIBRARY_SUFFIXES ".lib")

	SET(D3_GAMEDIR "c:/games/Descent3/")
	set (CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "lib/win" "lib/win/directx")
	SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Gm /EHsc /RTC1 /W3 /nologo /c /ZI /TP /errorReport:prompt")
	SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /GL /FD /EHsc /W3 /nologo /c /Zi /TP /errorReport:prompt")
	
	SET(CMAKE_EXE_LINKER_FLAGS "/SUBSYSTEM:WINDOWS /NODEFAULTLIB:LIBC")
	ADD_DEFINITIONS (-DIS_WINDOWS -D_CRT_SECURE_NO_WARNINGS -DMONO -DWIN32 -D_CRT_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE -DDO_SDL)

	SET(PLATFORM_INCLUDES "lib/win/directx" ${SDL_INCLUDE_DIR} "lib/win")
	
	FIND_LIBRARY(DSOUND_LIBRARY NAMES dsound "${CMAKE_SOURCE_DIR}/lib/win" "${CMAKE_SOURCE_DIR}/lib/win/directx")
	FIND_LIBRARY(DINPUT_LIBRARY NAMES dinput "${CMAKE_SOURCE_DIR}/lib/win" "${CMAKE_SOURCE_DIR}/lib/win/directx")
	FIND_LIBRARY(DXGUID_LIBRARY NAMES dxguid "${CMAKE_SOURCE_DIR}/lib/win" "${CMAKE_SOURCE_DIR}/lib/win/directx")
	FIND_LIBRARY(DDRAW_LIBRARY NAMES ddraw "${CMAKE_SOURCE_DIR}/lib/win" "${CMAKE_SOURCE_DIR}/lib/win/directx")
ENDIF ()

MESSAGE("Install will copy files to ${D3_GAMEDIR}")

include_directories("lib" "Descent3" ${PLATFORM_INCLUDES})

# file(GLOB_RECURSE INCS "*.h")

# project version
SET( ${PROJECT_NAME}_MAJOR_VERSION 1 )
SET( ${PROJECT_NAME}_MINOR_VERSION 5 )
SET( ${PROJECT_NAME}_PATCH_LEVEL 500 )

ADD_SUBDIRECTORY (2dlib)
ADD_SUBDIRECTORY (AudioEncode)
ADD_SUBDIRECTORY (bitmap)
ADD_SUBDIRECTORY (cfile)
ADD_SUBDIRECTORY (czip)
ADD_SUBDIRECTORY (d3music)

IF (WIN32)
ADD_SUBDIRECTORY (dd_grwin32)
ADD_SUBDIRECTORY (dd_vidwin32)
ADD_SUBDIRECTORY (ddio_win)
ADD_SUBDIRECTORY (win32)
ADD_SUBDIRECTORY (dd_sndlib)
ENDIF()

IF (UNIX)
ADD_SUBDIRECTORY(linux)
ADD_SUBDIRECTORY(dd_lnxsound)
ADD_SUBDIRECTORY(ddvid_lnx)
ADD_SUBDIRECTORY(lnxcontroller)
ADD_SUBDIRECTORY(ddio_lnx)
ENDIF()

ADD_SUBDIRECTORY (ddio_common)
ADD_SUBDIRECTORY (fix)
ADD_SUBDIRECTORY (grtext)
ADD_SUBDIRECTORY (manage)
ADD_SUBDIRECTORY (mem)
ADD_SUBDIRECTORY (misc)
ADD_SUBDIRECTORY (model)
ADD_SUBDIRECTORY (module)
ADD_SUBDIRECTORY (movie)
ADD_SUBDIRECTORY (music)
ADD_SUBDIRECTORY (networking)
ADD_SUBDIRECTORY (physics)
ADD_SUBDIRECTORY (renderer)
ADD_SUBDIRECTORY (rtperformance)
ADD_SUBDIRECTORY (sndlib)
ADD_SUBDIRECTORY (stream_audio)
ADD_SUBDIRECTORY (ui)
ADD_SUBDIRECTORY (unzip)
ADD_SUBDIRECTORY (vecmat)
ADD_SUBDIRECTORY (libmve)
ADD_SUBDIRECTORY (md5)
ADD_SUBDIRECTORY (libacm)
ADD_SUBDIRECTORY (Descent3 )

# For now we don't need to build the scripts under windows, so we'll only include
# the directory when building for linux/osx. In the future we may want to to fix bugs, etc.
IF(UNIX)
ADD_SUBDIRECTORY (netgames)
ADD_SUBDIRECTORY (netcon)
ADD_SUBDIRECTORY (scripts)
ENDIF()

# set default cmake build type to Debug (None Debug Release RelWithDebInfo MinSizeRel)
IF( NOT CMAKE_BUILD_TYPE )
SET( CMAKE_BUILD_TYPE "Debug" )
ENDIF()
