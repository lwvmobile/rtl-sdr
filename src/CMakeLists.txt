# Copyright 2012-2020 Osmocom Project
#
# This file is part of rtl-sdr
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

########################################################################
# Setup shared library variant
########################################################################
add_library(rtlsdr SHARED librtlsdr.c
  tuner_e4k.c tuner_fc0012.c tuner_fc0013.c tuner_fc2580.c tuner_r82xx.c)
target_link_libraries(rtlsdr ${LIBUSB_LIBRARIES} ${THREADS_PTHREADS_LIBRARY})
target_include_directories(rtlsdr PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>  # <prefix>/include
  ${LIBUSB_INCLUDE_DIRS}
  ${THREADS_PTHREADS_INCLUDE_DIR}
  )
set_target_properties(rtlsdr PROPERTIES DEFINE_SYMBOL "rtlsdr_EXPORTS")
set_target_properties(rtlsdr PROPERTIES OUTPUT_NAME rtlsdr)
set_target_properties(rtlsdr PROPERTIES SOVERSION 0)
set_target_properties(rtlsdr PROPERTIES VERSION ${LIBVER})
generate_export_header(rtlsdr)

########################################################################
# Setup static library variant
########################################################################
add_library(rtlsdr_static STATIC librtlsdr.c
  tuner_e4k.c tuner_fc0012.c tuner_fc0013.c tuner_fc2580.c tuner_r82xx.c)
target_link_libraries(rtlsdr ${LIBUSB_LIBRARIES} ${THREADS_PTHREADS_LIBRARY})
target_include_directories(rtlsdr_static PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>  # <prefix>/include
  ${LIBUSB_INCLUDE_DIRS}
  ${THREADS_PTHREADS_INCLUDE_DIR}
  )
set_property(TARGET rtlsdr_static APPEND PROPERTY COMPILE_DEFINITIONS "rtlsdr_STATIC" )
if(NOT WIN32)
# Force same library filename for static and shared variants of the library
set_target_properties(rtlsdr_static PROPERTIES OUTPUT_NAME rtlsdr)
endif()
generate_export_header(rtlsdr_static)

########################################################################
# Set up Windows DLL resource files
########################################################################
IF(MSVC)
    include(${CMAKE_SOURCE_DIR}/cmake/Modules/Version.cmake)

    configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/rtlsdr.rc.in
        ${CMAKE_CURRENT_BINARY_DIR}/rtlsdr.rc
    @ONLY)
  target_sources(rtlsdr PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/rtlsdr.rc)
  target_sources(rtlsdr_static PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/rtlsdr.rc)
ENDIF(MSVC)

########################################################################
# Setup libraries used in executables
########################################################################
add_library(convenience_static STATIC
    convenience/convenience.c
)
target_include_directories(convenience_static
  PRIVATE ${CMAKE_SOURCE_DIR}/include)
if(WIN32)
add_library(libgetopt_static STATIC
    getopt/getopt.c
)
target_link_libraries(convenience_static
    rtlsdr
)
endif()

########################################################################
# Build utility
########################################################################
# omitted
########################################################################
# Install built library files & utilities
########################################################################
install(TARGETS rtlsdr EXPORT RTLSDR-export
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # .so/.dylib file
  )
install(TARGETS rtlsdr_static EXPORT RTLSDR-export
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # .so/.dylib file
  )
# omitted install for utilities
