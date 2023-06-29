# Copyright © 2015 Academy of Motion Picture Arts and Sciences ("A.M.P.A.S.").
# Portions contributed by others as indicated. All rights reserved.
# 
# A worldwide, royalty-free, non-exclusive right to copy, modify, create
# derivatives, and use, in source and binary forms, is hereby granted, subject 
# to acceptance of this license. Performance of any of the aforementioned acts
# indicates acceptance to be bound by the following terms and conditions:
# 
# * Copies of source code, in whole or in part, must retain the above copyright
# notice, this list of conditions and the Disclaimer of Warranty.
# 
# * Use in binary form must retain the above copyright notice, this list of
# conditions and the Disclaimer of Warranty in the documentation and/or other
# materials provided with the distribution.
# 
# * Nothing in this license shall be deemed to grant any rights to trademarks,
# copyrights, patents, trade secrets or any other intellectual property of
# A.M.P.A.S. or any contributors, except as expressly stated herein.
# 
# * Neither the name "A.M.P.A.S." nor the name of any other contributors to 
# this software may be used to endorse or promote products derivative of or 
# based on this software without express prior written permission of A.M.P.A.S. 
# or the contributors, as appropriate.
# 
# This license shall be construed pursuant to the laws of the State of 
# California, and any disputes related thereto shall be subject to the 
# jurisdiction of the courts therein.
# 
# Disclaimer of Warranty: THIS SOFTWARE IS PROVIDED BY A.M.P.A.S. AND 
# CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT 
# NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A 
# PARTICULAR PURPOSE, AND NON-INFRINGEMENT ARE DISCLAIMED. IN NO EVENT SHALL 
# A.M.P.A.S., OR ANY CONTRIBUTORS OR DISTRIBUTORS, BE LIABLE FOR ANY DIRECT, 
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, RESITUTIONARY, OR CONSEQUENTIAL 
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR 
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER 
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, 
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE 
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# WITHOUT LIMITING THE GENERALITY OF THE FOREGOING, THE ACADEMY SPECIFICALLY
# DISCLAIMS ANY REPRESENTATIONS OR WARRANTIES WHATSOEVER RELATED TO PATENT OR
# OTHER INTELLECTUAL PROPERTY RIGHTS IN THE ACES CONTAINER REFERENCE
# IMPLEMENTATION, OR APPLICATIONS THEREOF, HELD BY PARTIES OTHER THAN 
# A.M.P.A.S., WHETHER DISCLOSED OR UNDISCLOSED.

cmake_minimum_required (VERSION 3.1)
project (AcesContainer)

include (GenerateExportHeader)

set( AcesContainer_MAJOR_VERSION 1 )
set( AcesContainer_MINOR_VERSION 0 )
set( AcesContainer_PATCH_VERSION 2 )
set( AcesContainer_VERSION ${AcesContainer_MAJOR_VERSION}.${AcesContainer_MINOR_VERSION}.${AcesContainer_PATCH_VERSION} )

set( INSTALL_LIB_DIR lib CACHE PATH "Install directory for libraries" )
set( INSTALL_INCLUDE_DIR include CACHE PATH "Install directory for public header files" )


if(APPLE)
  set( CMAKE_MACOSX_RPATH 1 )
endif()

if( WIN32 AND NOT CYGWIN )
  set(DEF_INSTALL_CMAKE_DIR CMake)
  set ( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W0" )
else()
  set(DEF_INSTALL_CMAKE_DIR lib/CMake/AcesContainer)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Install directory for project CMake files" )

## convert install paths to absolute
foreach( p LIB INCLUDE CMAKE )
  set( var INSTALL_${p}_DIR )
  if( NOT IS_ABSOLUTE "${${var}}" )
    set( ${var} "${CMAKE_INSTALL_PREFIX}/${${var}}" )
  endif()
endforeach()

OPTION (BUILD_SHARED_LIBS "Build Shared Libraries" ON)
IF ( BUILD_SHARED_LIBS )
  SET (DO_SHARED SHARED)
ELSE ()
  SET (DO_SHARED STATIC)
ENDIF ()

add_library( AcesContainer ${DO_SHARED}
	aces_iostat.cpp
	aces_attributestructs.cpp
	aces_formatter.cpp
	aces_Writer.cpp
	aces_timing.cpp
	aces_typesAndRationals.cpp
	aces_writeattributes.cpp
	aces_md5.cc
)

GENERATE_EXPORT_HEADER( AcesContainer 
             BASE_NAME AcesContainer
             EXPORT_MACRO_NAME AcesContainer_EXPORT
             EXPORT_FILE_NAME AcesContainer_Export.h
             STATIC_DEFINE AcesContainer_BUILT_AS_STATIC
)

install (TARGETS AcesContainer EXPORT AcesContainerTargets DESTINATION ${INSTALL_LIB_DIR})
install (FILES 
			aces_errors.h
			aces_genericWriter.h
			aces_iostat.h
			aces_items.h
			aces_log.h
			aces_attributestructs.h
			aces_formatter.h
			aces_Writer.h
			aces_timing.h
			aces_typesAndRationals.h
			aces_types.h
			aces_writeattributes.h
		 DESTINATION 
		 	${INSTALL_INCLUDE_DIR}/aces
		 )


find_package( PkgConfig )
if ( PKG_CONFIG_FOUND )
configure_file(config/AcesContainer.pc.in "${PROJECT_BINARY_DIR}/AcesContainer.pc" @ONLY)
install( FILES "${PROJECT_BINARY_DIR}/AcesContainer.pc" DESTINATION lib/pkgconfig COMPONENT dev )
endif()

include_directories(
  "${PROJECT_SOURCE_DIR}"   # to find foo/foo.h
  "${PROJECT_BINARY_DIR}")  # to find foo/config.h
  
# Add all targets to the build-tree export set
export(TARGETS AcesContainer
  FILE "${PROJECT_BINARY_DIR}/AcesContainerTargets.cmake")
# export(TARGETS AcesContainer_lib
#  FILE "${PROJECT_BINARY_DIR}/AcesContainerTargets.cmake")
 
# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry)
export(PACKAGE AcesContainer)
# export(PACKAGE AcesContainer_lib)

# Create the FooBarConfig.cmake and FooBarConfigVersion files
file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
   "${INSTALL_INCLUDE_DIR}")

# ... for the build tree
set(CONF_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}" "${PROJECT_BINARY_DIR}")
set(CONF_LIB_DIRS "${PROJECT_BINARY_DIR}")
configure_file(config/AcesContainerConfig.cmake.in
  "${PROJECT_BINARY_DIR}/AcesContainerConfig.cmake" @ONLY)

# ... for the install tree
set(CONF_INCLUDE_DIRS "\${AcesContainer_CMAKE_DIR}/${REL_INCLUDE_DIR}" "\${AcesContainer_CMAKE_DIR}/${REL_INCLUDE_DIR}/aces")
set(CONF_LIB_DIRS "${INSTALL_LIB_DIR}")
configure_file(config/AcesContainerConfig.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/AcesContainerConfig.cmake" @ONLY)
# ... for both
configure_file(config/AcesContainerConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/AcesContainerConfigVersion.cmake" @ONLY)
 
# Install the FooBarConfig.cmake and FooBarConfigVersion.cmake
install(FILES
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/AcesContainerConfig.cmake"
  "${PROJECT_BINARY_DIR}/AcesContainerConfigVersion.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)
 
# Install the export set for use with the install-tree
install(EXPORT AcesContainerTargets DESTINATION
  "${INSTALL_CMAKE_DIR}" COMPONENT dev)
  
  
