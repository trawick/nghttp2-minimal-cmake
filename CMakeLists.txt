# nghttp2 - HTTP/2 C Library
#
# Copyright (c) 2015 Tatsuhiro Tsujikawa
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
# LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
# OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
# WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Current source: https://github.com/trawick/nghttp2-minimal-cmake

PROJECT(NGHTTP2 C)

CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

OPTION(INSTALL_PDB          "Install .pdb files (if generated)"  ON)

FUNCTION(GET_NGHTTP2_VERSION_STRING output_version)
  # Extract nghttp2 version from canonical source file
  SET(configureac_filename ${PROJECT_SOURCE_DIR}/configure.ac)
  FILE(STRINGS ${configureac_filename} configureac)
  SET(version_found FALSE)
  FOREACH(line ${configureac})
    STRING(REGEX MATCH "^AC_INIT\\(\\[nghttp2\\], \\[(.*)\\], \\[" ignored ${line})
    SET(nghttp2_version_str ${CMAKE_MATCH_1})
    IF(NOT ${nghttp2_version_str} STREQUAL "")
      SET(version_found TRUE)
      BREAK()
    ENDIF()
  ENDFOREACH()
  IF(NOT ${version_found})
    MESSAGE(FATAL_ERROR "nghttp2 version string not found in source file ${configureac_filename}")
  ENDIF()
  SET(${output_version} ${nghttp2_version_str} PARENT_SCOPE)
ENDFUNCTION()

FUNCTION(GET_NGHTTP2_MAJ_MIN_PATCH nghttp2_version_str ver_major ver_minor ver_patch)
  # Given the nghttp2 version string, extract the major minor and patch components.
  STRING(REPLACE "." " " version_components ${nghttp2_version_str})
  STRING(REPLACE "-" " " version_components ${version_components})
  SEPARATE_ARGUMENTS(version_components)
  LIST(GET version_components 0 major)
  LIST(GET version_components 1 minor)
  LIST(GET version_components 2 patch)
  SET(${ver_major} ${major} PARENT_SCOPE)
  SET(${ver_minor} ${minor} PARENT_SCOPE)
  SET(${ver_patch} ${patch} PARENT_SCOPE)
ENDFUNCTION()

FUNCTION(FILL_TO_2_DIGITS in_str out_str)
  # Given a 1 or 2 digit input string, 0-fill to 2 digits
  STRING(LENGTH ${in_str} len)
  IF(len LESS 2)
    SET(in_str "0${in_str}")
  ENDIF()
  SET(${out_str} ${in_str} PARENT_SCOPE)
ENDFUNCTION()

FUNCTION(GET_NGHTTP2_VERSION_HEX nghttp2_version_str ver_hex)
  # Build hex version number from version string, which is either x.y.z or x.y.z-DEV
  GET_NGHTTP2_MAJ_MIN_PATCH(${nghttp2_version_str} ver_major ver_minor ver_patch)
  FILL_TO_2_DIGITS(${ver_major} filled_major)
  FILL_TO_2_DIGITS(${ver_minor} filled_minor)
  FILL_TO_2_DIGITS(${ver_patch} filled_patch)
  SET(${ver_hex} "0x${filled_major}${filled_minor}${filled_patch}" PARENT_SCOPE)
ENDFUNCTION()

FUNCTION(GET_NGHTTP2_VERSION_RC nghttp2_version_str ver_rc)
  # Build maj,min,patch,0 version string for .rc file
  GET_NGHTTP2_MAJ_MIN_PATCH(${nghttp2_version_str} ver_major ver_minor ver_patch)
  SET(${ver_rc} "${ver_major},${ver_minor},${ver_patch},0" PARENT_SCOPE)
ENDFUNCTION()

SET(NGHTTP2_SOURCES
  lib/nghttp2_pq.c
  lib/nghttp2_map.c
  lib/nghttp2_queue.c
  lib/nghttp2_frame.c
  lib/nghttp2_buf.c
  lib/nghttp2_stream.c
  lib/nghttp2_outbound_item.c
  lib/nghttp2_session.c
  lib/nghttp2_submit.c
  lib/nghttp2_helper.c
  lib/nghttp2_npn.c
  lib/nghttp2_hd.c
  lib/nghttp2_hd_huffman.c
  lib/nghttp2_hd_huffman_data.c
  lib/nghttp2_version.c
  lib/nghttp2_priority_spec.c
  lib/nghttp2_option.c
  lib/nghttp2_callbacks.c
  lib/nghttp2_mem.c
  lib/nghttp2_http.c
)

SET(NGHTTP2_INCLUDE_DIRECTORIES
  ${PROJECT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/includes
)

SET(NGHTTP2_SYSTEM_LIBS )  # (nothing yet)

INCLUDE_DIRECTORIES(${NGHTTP2_INCLUDE_DIRECTORIES})

SET(NGHTTP2_PUBLIC_HEADERS_STATIC
  lib/includes/nghttp2/nghttp2.h
)
SET(NGHTTP2_PUBLIC_HEADERS_GENERATED
  ${PROJECT_BINARY_DIR}/nghttp2/nghttp2ver.h
)

# Get nghttp2 version string in different forms for substitution
# in nghttp2ver.h.in.
GET_NGHTTP2_VERSION_STRING(nghttp2_version_str)
GET_NGHTTP2_VERSION_HEX(${nghttp2_version_str} nghttp2_version_hex)
SET(PACKAGE_VERSION "${nghttp2_version_str}")
SET(PACKAGE_VERSION_NUM ${nghttp2_version_hex})
CONFIGURE_FILE(lib/includes/nghttp2/nghttp2ver.h.in
               ${PROJECT_BINARY_DIR}/nghttp2/nghttp2ver.h)
# And in another form for use in the .rc file
GET_NGHTTP2_VERSION_RC(${nghttp2_version_str} nghttp2_rc_version)

SET(install_targets)
SET(install_bin_pdb)

ADD_LIBRARY(nghttp2 SHARED ${NGHTTP2_SOURCES} lib/nghttp2.rc)
SET(install_targets ${install_targets} nghttp2)
SET(install_bin_pdb ${install_bin_pdb} ${PROJECT_BINARY_DIR}/nghttp2.pdb)
TARGET_LINK_LIBRARIES(nghttp2 ${SYSTEM_LIBS})
SET_TARGET_PROPERTIES(nghttp2 PROPERTIES COMPILE_DEFINITIONS "_U_=;ssize_t=long;NGHTTP2_RC_VERSION=${nghttp2_rc_version};BUILDING_NGHTTP2")

INSTALL(TARGETS ${install_targets}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
)

IF(INSTALL_PDB)
  INSTALL(FILES ${install_bin_pdb}
          DESTINATION bin
          CONFIGURATIONS RelWithDebInfo Debug)
ENDIF()

INSTALL(FILES ${NGHTTP2_PUBLIC_HEADERS_STATIC} ${NGHTTP2_PUBLIC_HEADERS_GENERATED} DESTINATION include/nghttp2)

MESSAGE(STATUS "  nghttp2 version ................. : ${nghttp2_version_str}")
MESSAGE(STATUS "  Build type ...................... : ${CMAKE_BUILD_TYPE}")
MESSAGE(STATUS "  Install .pdb (if available)...... : ${INSTALL_PDB}")
MESSAGE(STATUS "  Install prefix .................. : ${CMAKE_INSTALL_PREFIX}")
MESSAGE(STATUS "  C compiler ...................... : ${CMAKE_C_COMPILER}")
