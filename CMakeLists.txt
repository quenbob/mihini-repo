#*******************************************************************************
# Copyright (c) 2012 Sierra Wireless and others.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
#
# Contributors:
#     Sierra Wireless - initial API and implementation
#*******************************************************************************

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.3)
INCLUDE(cmake/add_lua_library.cmake)
INCLUDE(cmake/add_public_header.cmake)
INCLUDE(cmake/add_unit_test.cmake)

PROJECT(EMBEDDED)

SET(CMAKE_C_FLAGS_MINSIZEREL "-Os -s")

ADD_DEFINITIONS(-Wall)
SET(CMAKE_INSTALL_PREFIX ${EMBEDDED_BINARY_DIR}/runtime)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${EMBEDDED_BINARY_DIR}/static-libs)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${EMBEDDED_BINARY_DIR}/runtime/bin)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${EMBEDDED_BINARY_DIR}/runtime/lib)
SET(CMAKE_LUA_LIBRARY_OUTPUT_DIRECTORY ${EMBEDDED_BINARY_DIR}/runtime/lua)
SET(CMAKE_PUBLIC_HEADERS_OUTPUT_DIRECTORY ${EMBEDDED_BINARY_DIR}/runtime/itf)

# remove the lib prefix on unix systems...
SET(CMAKE_SHARED_MODULE_PREFIX "")

# handle build version.
# First try to get it from git. If not found look for a build_version file in the root source tree
# If not found set the build to unknown
SET(BUILD_VERSION_FILE "${PROJECT_SOURCE_DIR}/build_version")


# Set the git revision if it exists
IF(EXISTS "${PROJECT_SOURCE_DIR}/.git/")
    find_package(Git)
    IF(GIT_FOUND)
        SET(ENV{GIT_DIR} ${PROJECT_SOURCE_DIR}/.git)
        execute_process(COMMAND git rev-parse --short HEAD OUTPUT_VARIABLE GIT_REVISION)
        string(REPLACE "\n" "" GIT_REVISION "${GIT_REVISION}")
        ADD_DEFINITIONS(-DGIT_REV=\"${GIT_REVISION}\")
        FILE(WRITE ${BUILD_VERSION_FILE} "${GIT_REVISION}\n")
        unset(ENV{GIT_DIR})
    ENDIF(GIT_FOUND)
ELSE()
    IF(EXISTS ${BUILD_VERSION_FILE})
        FILE (STRINGS ${BUILD_VERSION_FILE} BUILD_VERSION)
        ADD_DEFINITIONS(-DGIT_REV=\"${BUILD_VERSION}\")
    ENDIF()
ENDIF()

IF(EXISTS "${PROJECT_SOURCE_DIR}/non-free/porting/${PLATFORM}/CMakeLists.txt")
    ADD_SUBDIRECTORY(non-free/porting/${PLATFORM})
ELSE()
    ADD_SUBDIRECTORY(porting/${PLATFORM})
ENDIF()

ADD_SUBDIRECTORY (libs)
ADD_SUBDIRECTORY (luafwk)
ADD_SUBDIRECTORY (agent)
ADD_SUBDIRECTORY (cfwk)
ADD_SUBDIRECTORY (tools/appmon_daemon)
ADD_SUBDIRECTORY (tools/luasignalcmd)
#ADD_SUBDIRECTORY (porting)
ADD_SUBDIRECTORY (tests)
ADD_SUBDIRECTORY (tools/LuaPlatformServer)
ADD_SUBDIRECTORY (doc)

IF(EXISTS "${PROJECT_SOURCE_DIR}/non-free/CMakeLists.txt")
ADD_SUBDIRECTORY (non-free)
ENDIF()

ADD_CUSTOM_TARGET(mihini)
ADD_DEPENDENCIES(mihini agent cfwk luafwk)

IF(CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
    ADD_CUSTOM_COMMAND(TARGET mihini
        POST_BUILD
        COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/bin/compress-build-artifacts.sh ${CMAKE_INSTALL_PREFIX}
    )
ENDIF()

SET(CPACK_GENERATOR "TGZ;DEB")
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Open Source framework for M2M")
SET(CPACK_PACKAGE_NAME "mihini")
SET(CPACK_PACKAGE_VENDOR "Sierra Wireless")
SET(CPACK_PACKAGE_VERSION_MAJOR "0")
SET(CPACK_PACKAGE_VERSION_MINOR "8")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
SET(CPACK_PACKAGE_CONTACT "gsaurel@sierrawireless.com")
SET(CPACK_DEBIAN_PACKAGE_HOMEPAGE "http://www.eclipse.org/mihini/")
SET(CPACK_PACKAGING_INSTALL_PREFIX "/opt/mihini")
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_CURRENT_SOURCE_DIR}/porting/debian/postinst;${CMAKE_CURRENT_SOURCE_DIR}/porting/debian/prerm;")
SET(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}-${CPACK_DEBIAN_PACKAGE_ARCHITECTURE})

INCLUDE(CPack)
