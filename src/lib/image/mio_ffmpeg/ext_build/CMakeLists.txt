#
# Copyright (C) 2022  Autodesk, Inc. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

CMAKE_MINIMUM_REQUIRED(VERSION 3.24)

PROJECT("ext_mio_ffmpeg")

SET(_target
    "ext_mio_ffmpeg"
)

LIST(APPEND _sources init.cpp)

SET(CMAKE_CXX_STANDARD
    17
)
SET(CMAKE_C_STANDARD
    17
)
SET(CMAKE_CXX_EXTENSIONS
    OFF
)
SET(CMAKE_CXX_STANDARD_REQUIRED
    TRUE
)
SET(CMAKE_C_STANDARD_REQUIRED
    TRUE
)

IF(APPLE)
  SET(CMAKE_OSX_ARCHITECTURES
      "x86_64"
      CACHE STRING "Force compilation for Intel processors." FORCE
  )
ENDIF()

ADD_LIBRARY(
  ${_target} SHARED
  ${_sources}
)

IF(APPLE)
  # Darwin is the name of the mach BSD-base kernel :-)
  SET(MIO_TARGET_DARWIN
      BOOL TRUE "Detected target is Apple's macOS"
      CACHE INTERNAL ""
  )
  TARGET_COMPILE_OPTIONS(
    ${_target}
    PRIVATE -DARCH_IA32_64
            -DPLATFORM_DARWIN
            -DTWK_LITTLE_ENDIAN
            -D__LITTLE_ENDIAN__
            -DPLATFORM_APPLE_MACH_BSD
            -DPLATFORM_OPENGL
            -DGL_SILENCE_DEPRECATION
            # _XOPEN_SOURCE, Required on macOS to resolve such compiling error:
            # /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX12.3.sdk/usr/include/ucontext.h:51:2: error: The
            # deprecated ucontext routines require _XOPEN_SOURCE to be defined error The deprecated ucontext routines require _XOPEN_SOURCE to be defined
            # https://pubs.opengroup.org/onlinepubs/009695399/
            -D_XOPEN_SOURCE=600
            -D_DARWIN_C_SOURCE # at least on MacOS: needed in Twk for strcasecmp which is not standard
            -DOSX_VERSION=${_macos_version_string}
  )
ELSEIF(UNIX)
  SET(MIO_TARGET_LINUX
      BOOL TRUE "Detected a generic Linux OS"
      CACHE INTERNAL ""
  )
  TARGET_COMPILE_OPTIONS(
    ${_target}
    PRIVATE -DARCH_IA32_64
            -DPLATFORM_LINUX
            -DTWK_LITTLE_ENDIAN
            -D__LITTLE_ENDIAN__
            -DTWK_NO_SGI_BYTE_ORDER
            -DGL_GLEXT_PROTOTYPES
            -DPLATFORM_OPENGL=1
  )
ELSEIF(WIN32)
  SET(MIO_TARGET_WINDOWS
      BOOL TRUE "Detected target is Microsoft's Windows (64bit)"
      CACHE INTERNAL ""
  )
  TARGET_COMPILE_OPTIONS(
    ${_target}
    PRIVATE -DARCH_IA32_64
            -DPLATFORM_WINDOWS
            -DTWK_LITTLE_ENDIAN
            -D__LITTLE_ENDIAN__
            -DTWK_NO_SGI_BYTE_ORDER
            -DGL_GLEXT_PROTOTYPES
            -DPLATFORM_OPENGL=1
            -DBOOST_ALL_NO_LIB=1
            -DBOOST_ALL_DYN_LINK=1
  )
ELSE()
  MESSAGE(FATAL_ERROR "Unsupported platform")
ENDIF()

SET(BOOST_MAJOR_MINOR_VERSION
    "1_76"
)
SET(BOOST_WINDOWS_RELEASE_SUFFIX
    "vc142-mt-x64"
)
SET(BOOST_WINDOWS_DEBUG_SUFFIX
    "vc142-mt-gd-x64"
)

SET(BOOST_SHARED_PREFIX
    ${CMAKE_SHARED_LIBRARY_PREFIX}
)
SET(MIO_SHARED_PREFIX
    ${CMAKE_SHARED_LIBRARY_PREFIX}
)

IF(MIO_TARGET_WINDOWS)
  SET(BOOST_SHARED_PREFIX
      ""
  )
  SET(MIO_SHARED_PREFIX
      ""
  )
ENDIF()

SET(MIO_INST_LIB_DIR
    "../../lib"
)

SET(MIO_INST_BIN_DIR
    "../../bin"
)

IF(DEFINED ENV{MIO_DEP_ROOT_DIR})
  SET(MIO_INST_LIB_DIR
      "$ENV{MIO_DEP_ROOT_DIR}/lib"
  )

  SET(MIO_INST_BIN_DIR
      "$ENV{MIO_DEP_ROOT_DIR}/bin"
  )
ENDIF()

SET(MIO_FFMPEG_LIB_DIR
    "lib"
)

IF(DEFINED ENV{MIO_FFMPEG_ROOT_DIR})
  SET(MIO_FFMPEG_LIB_DIR
      "$ENV{MIO_FFMPEG_ROOT_DIR}/lib"
  )
ENDIF()

SET(MOVIE_FFMPEG_NAME
    ${CMAKE_STATIC_LIBRARY_PREFIX}MovieFFMpeg${CMAKE_STATIC_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_MOVIEFFMPEG MovieFFMpeg
  NAMES ${MOVIE_FFMPEG_NAME}
  HINTS ${MIO_FFMPEG_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY FFMPEG: ${LIB_MOVIEFFMPEG} -- Name: ${MOVIE_FFMPEG_NAME}")

# ----------------
# Dependencies of Deps
# ----------------

SET(MOVIE_FFMPEG_MP4V2_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}mp4v2${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_MP4V2 mp4v2
  NAMES ${MOVIE_FFMPEG_MP4V2_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY MP4V2: ${LIB_MP4V2} -- Name: ${MOVIE_FFMPEG_MP4V2_NAME}")

SET(MOVIE_FFMPEG_MP4V2Utils_NAME
    ${CMAKE_STATIC_LIBRARY_PREFIX}mp4v2Utils${CMAKE_STATIC_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_MP4V2Utils mp4v2Utils
  NAMES ${MOVIE_FFMPEG_MP4V2Utils_NAME}
  HINTS ${MIO_FFMPEG_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY MP4VS2Utils: ${LIB_MP4V2Utils} -- Name: ${MOVIE_FFMPEG_MP4V2Utils_NAME}")

# FFMpeg Deps (AV & ffmpeg)
SET(FFMPEG_AVCODEC_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}avcodec.58.134.100${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_FFMPEG_AVCODEC avcodec
  NAMES avcodec.58.134.100 ${FFMPEG_AVCODEC_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY AVCODEC: ${LIB_FFMPEG_AVCODEC} -- Name: ${FFMPEG_AVCODEC_NAME}")

SET(FFMPEG_AVFORMAT_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}avformat.58.76.100${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_FFMPEG_AVFORMAT avformat
  NAMES avformat.58.76.100 ${FFMPEG_AVFORMAT_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY AVFORMAT: ${LIB_FFMPEG_AVFORMAT} -- Name: ${FFMPEG_AVFORMAT_NAME}")

SET(FFMPEG_AVUTIL_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}avutil.56.70.100${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_FFMPEG_AVUTIL avutil
  NAMES avutil.56.70.100 ${FFMPEG_AVUTIL_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY AVUTIL: ${LIB_FFMPEG_AVUTIL} -- Name: ${FFMPEG_AVUTIL_NAME}")

SET(MOVIE_FFMPEG_SWRESAMPLE_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}swresample.3.9.100${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_FFMPEG_SWRESAMPLE swresample
  NAMES swresample.3.9.100 ${MOVIE_FFMPEG_SWRESAMPLE_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY FFMPEG_SWRESAMPLE: ${LIB_FFMPEG_SWRESAMPLE} -- Name: ${MOVIE_FFMPEG_SWRESAMPLE_NAME}")

SET(MOVIE_FFMPEG_SWSCALE_NAME
    ${CMAKE_SHARED_LIBRARY_PREFIX}swscale.5.9.100${CMAKE_SHARED_LIBRARY_SUFFIX}
)
FIND_LIBRARY(
  LIB_FFMPEG_SWSCALE swscale
  NAMES swscale.5.9.100 ${MOVIE_FFMPEG_SWSCALE_NAME}
  HINTS ${MIO_INST_LIB_DIR}
  NO_DEFAULT_PATH
)
MESSAGE("DEP LIBRARY FFMPEG_SWSCALE: ${LIB_FFMPEG_SWSCALE} -- Name: ${MOVIE_FFMPEG_SWSCALE_NAME}")

# Tweak Deps
SET(TWK_FB_NAME
    ${MIO_SHARED_PREFIX}TwkFB${CMAKE_SHARED_LIBRARY_SUFFIX}
)
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_TWK_FB TwkFB
    NAMES ${TWK_FB_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
  MESSAGE("DEP LIBRARY TWK FB: ${LIB_TWK_FB} -- Name: ${TWK_FB_NAME}")
ELSE()
  FIND_LIBRARY(
    LIB_TWK_FB TwkFB
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
  MESSAGE("DEP LIBRARY TWK FB Windows: ${LIB_TWK_FB} -- Name: 'TwkFB' -- Looking for a static lib")
ENDIF()

SET(TWEAK_TwkAudio_NAME
    ${MIO_SHARED_PREFIX}TwkAudio${CMAKE_SHARED_LIBRARY_SUFFIX}
)
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_Tweak_TwkAudio TwkAudio
    NAMES ${TWEAK_TwkAudio_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_Tweak_TwkAudio TwkAudio
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY Tweak TwkAudio: ${LIB_Tweak_TwkAudio} -- Name: ${TWEAK_TwkAudio_NAME}")

SET(TWEAK_TwkExc_NAME
    ${MIO_SHARED_PREFIX}TwkExc${CMAKE_SHARED_LIBRARY_SUFFIX}
)
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_Tweak_TwkExc TwkExc
    NAMES ${TWEAK_TwkExc_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_Tweak_TwkExc TwkExc
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY Tweak TwkExc: ${LIB_Tweak_TwkExc} -- Name: ${TWEAK_TwkExc_NAME}")

SET(TWEAK_TwkMovie_NAME
    ${MIO_SHARED_PREFIX}TwkMovie${CMAKE_SHARED_LIBRARY_SUFFIX}
)
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_Tweak_TwkMovie TwkMovie
    NAMES ${TWEAK_TwkMovie_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_Tweak_TwkMovie TwkMovie
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY Tweak TwkMovie: ${LIB_Tweak_TwkMovie} -- Name: ${TWEAK_TwkMovie_NAME}")

SET(TWEAK_TwkUtils_NAME
    ${MIO_SHARED_PREFIX}TwkUtil${CMAKE_SHARED_LIBRARY_SUFFIX}
)
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_Tweak_TwkUtil TwkUtil
    NAMES ${TWEAK_TwkUtils_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_Tweak_TwkUtil TwkUtil
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY Tweak TwkUtil: ${LIB_Tweak_TwkUtil} -- Name: ${TWEAK_TwkUtils_NAME}")

# Boost Dep (Note: Windows: Debug vs Release: we do not know the configuration while configuration hence we have to look for both configs)
IF(MIO_TARGET_WINDOWS)
  SET(BOOST_PGRAM_OPTIONS_NAME
      boost_program_options-${BOOST_WINDOWS_RELEASE_SUFFIX}-${BOOST_MAJOR_MINOR_VERSION}${CMAKE_STATIC_LIBRARY_SUFFIX}
  )
ELSE()
  SET(BOOST_PGRAM_OPTIONS_NAME
      ${BOOST_SHARED_PREFIX}boost_program_options${CMAKE_SHARED_LIBRARY_SUFFIX}
  )
ENDIF()
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_BOOST_PGRAMOPTIONS boost_program_options
    NAMES ${BOOST_PGRAM_OPTIONS_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_BOOST_PGRAMOPTIONS boost_program_options
    NAMES ${BOOST_PGRAM_OPTIONS_NAME} boost_program_options-${BOOST_WINDOWS_DEBUG_SUFFIX}-${BOOST_MAJOR_MINOR_VERSION}${CMAKE_STATIC_LIBRARY_SUFFIX}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY BOOST PGRAM OPTS: ${LIB_BOOST_PGRAMOPTIONS} -- Name: ${BOOST_PGRAM_OPTIONS_NAME}")

IF(MIO_TARGET_WINDOWS)
  SET(BOOST_FS_NAME
      boost_filesystem-${BOOST_WINDOWS_RELEASE_SUFFIX}-${BOOST_MAJOR_MINOR_VERSION}${CMAKE_STATIC_LIBRARY_SUFFIX}
  )
ELSE()
  SET(BOOST_FS_NAME
      ${BOOST_SHARED_PREFIX}boost_filesystem${CMAKE_SHARED_LIBRARY_SUFFIX}
  )
ENDIF()
IF(NOT MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_BOOST_FS boost_filesystem
    NAMES ${BOOST_FS_NAME}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ELSE()
  FIND_LIBRARY(
    LIB_BOOST_FS boost_filesystem
    NAMES ${BOOST_FS_NAME} boost_filesystem-${BOOST_WINDOWS_DEBUG_SUFFIX}-${BOOST_MAJOR_MINOR_VERSION}${CMAKE_STATIC_LIBRARY_SUFFIX}
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()
MESSAGE("DEP LIBRARY BOOST FS: ${LIB_BOOST_FS} -- Name: ${BOOST_FS_NAME}")

# Windows Only Ext Deps: Nedmalloc & win_pthreads
IF(MIO_TARGET_WINDOWS)
  FIND_LIBRARY(
    LIB_NEDMALLOC nedmalloc
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
  FIND_LIBRARY(
    LIB_WINPTHREADS win_pthreads
    HINTS ${MIO_INST_LIB_DIR}
    NO_DEFAULT_PATH
  )
ENDIF()

TARGET_INCLUDE_DIRECTORIES(
  ${_target}
  PRIVATE "includes" "includes/Imath"
)

IF(MIO_TARGET_WINDOWS)
  TARGET_INCLUDE_DIRECTORIES(
    ${_target}
    PRIVATE "includes/win_pthreads"
  )

  TARGET_COMPILE_OPTIONS(
    ${_target}
    PRIVATE "-D__STDC_CONSTANT_MACROS"
  )
ENDIF()

TARGET_LINK_LIBRARIES(
  ${_target}
  PRIVATE ${LIB_MOVIEFFMPEG} ${LIB_BOOST_PGRAMOPTIONS} ${LIB_TWK_FB}
  PRIVATE ${LIB_FFMPEG_AVCODEC} ${LIB_FFMPEG_AVFORMAT} ${LIB_FFMPEG_AVUTIL} ${LIB_FFMPEG_SWRESAMPLE} ${LIB_FFMPEG_SWSCALE}
  PRIVATE ${LIB_MP4V2} ${LIB_MP4V2Utils}
  PRIVATE ${LIB_BOOST_FS}
  PRIVATE ${LIB_Tweak_TwkAudio} ${LIB_Tweak_TwkExc} ${LIB_Tweak_TwkMovie} ${LIB_Tweak_TwkUtil}
)

IF(MIO_TARGET_WINDOWS)
  TARGET_LINK_LIBRARIES(
    ${_target}
    PRIVATE ${LIB_NEDMALLOC} ${LIB_WINPTHREADS}
  )
ENDIF()

SET(final_lib_name_no_ext
    mio_ffmpeg
)
IF(MIO_TARGET_DARWIN
   OR MIO_TARGET_LINUX
)
  SET(mio_lib_source_name
      ${CMAKE_SHARED_LIBRARY_PREFIX}${_target}${CMAKE_SHARED_LIBRARY_SUFFIX}
  )
  SET(mio_lib_dest_name
      ${final_lib_name_no_ext}${CMAKE_SHARED_LIBRARY_SUFFIX}
  )
  SET(mio_lib_source_path
      ${CMAKE_BINARY_DIR}/${mio_lib_source_name}
  )
  SET(mio_lib_dest_path
      ${CMAKE_BINARY_DIR}/${mio_lib_dest_name}
  )

  # as long as the lib name is sync'ed with the lib name on the platform, the rest is cross-platform for Mac & Linux
  ADD_CUSTOM_COMMAND(
    TARGET ${_target}
    POST_BUILD
    COMMENT "Renaming lib ${mio_lib_source_name} into final name: ${mio_lib_dest_name}"
    COMMAND ${CMAKE_COMMAND} -E copy ${mio_lib_source_path} ${mio_lib_dest_path}
    COMMAND ${CMAKE_COMMAND} -E rm ${mio_lib_source_path}
  )
ELSE()
  # CMake Generator Expressions are required for build-time values for Multi-Config Generators (Visual Studio)
  SET(relative_path_prefix
      $<$<CONFIG:Release>:Release>$<$<CONFIG:Debug>:Debug>
  )
  # Hack: We cannot check if a file exists in ADD_CUSTOM_COMMAND especially when mixing with Generator Expression due to multi-config. Therefore since we do not
  # know if we are in Debug or Release for the POST_BUILD commands, we repeat a command for Release since a command cannot be empty.
  SET(hack_source_pdb_file_path_or_dup_exp_path
      $<$<CONFIG:Debug>:Debug/${_target}.pdb>$<$<CONFIG:Release>:${relative_path_prefix}/${_target}.exp>
  )
  SET(hack_dest_pdb_file_path_or_dup_exp_path
      $<$<CONFIG:Debug>:Debug/${final_lib_name_no_ext}.pdb>$<$<CONFIG:Release>:${relative_path_prefix}/${final_lib_name_no_ext}.exp>
  )
  ADD_CUSTOM_COMMAND(
    TARGET ${_target}
    POST_BUILD
    COMMENT "Post Build ${_target} Windows ${relative_path_prefix}: renaming files"
    COMMAND ${CMAKE_COMMAND} -E copy ${relative_path_prefix}/${_target}.dll ${relative_path_prefix}/${final_lib_name_no_ext}.dll
    COMMAND ${CMAKE_COMMAND} -E copy ${relative_path_prefix}/${_target}.lib ${relative_path_prefix}/${final_lib_name_no_ext}.lib
    COMMAND ${CMAKE_COMMAND} -E copy ${relative_path_prefix}/${_target}.exp ${relative_path_prefix}/${final_lib_name_no_ext}.exp
    # Only for Debug, Release will duplicate a previous command.
    COMMAND ${CMAKE_COMMAND} -E copy ${hack_source_pdb_file_path_or_dup_exp_path} ${hack_dest_pdb_file_path_or_dup_exp_path}
    # Finally remove old files, rm -f doesn't fail if the file doesn't exist.
    COMMAND ${CMAKE_COMMAND} -E rm ${relative_path_prefix}/${_target}.dll ${relative_path_prefix}/${_target}.lib ${relative_path_prefix}/${_target}.exp
    COMMAND ${CMAKE_COMMAND} -E rm -f Debug/${_target}.pdb
  )
ENDIF()
