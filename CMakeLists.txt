##
## This file is part of the sigrok-androidutils project.
##
## Copyright (C) 2020 Uwe Hermann <uwe@hermann-uwe.de>
##
## This program is free software: you can redistribute it and/or modify
## it under the terms of the GNU General Public License as published by
## the Free Software Foundation, either version 2 of the License, or
## (at your option) any later version.
##
## This program is distributed in the hope that it will be useful,
## but WITHOUT ANY WARRANTY; without even the implied warranty of
## MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
## GNU General Public License for more details.
##
## You should have received a copy of the GNU General Public License
## along with this program.  If not, see <http://www.gnu.org/licenses/>.
##

cmake_minimum_required(VERSION 3.12)

project(sigrokandroidutils VERSION 0.1.0 LANGUAGES C CXX)

include(GNUInstallDirs)

find_package(Git QUIET)
if(Git_FOUND)
	execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} ERROR_QUIET
		OUTPUT_VARIABLE GITHASH OUTPUT_STRIP_TRAILING_WHITESPACE)
	execute_process(COMMAND
		${GIT_EXECUTABLE} name-rev --tags --name-only ${GITHASH}
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} ERROR_QUIET
		OUTPUT_VARIABLE GITTAG OUTPUT_STRIP_TRAILING_WHITESPACE)
	if((DEFINED GITTAG) AND ("${GITTAG}" STREQUAL "undefined"))
		string(APPEND PROJECT_VERSION "-git-${GITHASH}")
	endif()
endif()

option(BUILD_SHARED_LIBS "Shared (non-static) build" TRUE)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE RelWithDebInfo)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS TRUE)

find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBGLIB REQUIRED IMPORTED_TARGET "libsigrokcxx>=0.5.0")

# find_package(Java REQUIRED COMPONENTS Development)
# find_package(JNI REQUIRED)

add_library(${PROJECT_NAME} "")
add_library(sigrok::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

target_sources(${PROJECT_NAME} PRIVATE 
	lib/jvm_glue.cpp lib/envsetup.cpp
	lib/libsigrokandroidutils.h
	lib/libsigrokandroidutils-internal.h)

target_compile_options(${PROJECT_NAME} PRIVATE -fno-exceptions)

# target_include_directories(${PROJECT_NAME} PRIVATE ${JNI_INCLUDE_DIRS})

# target_link_libraries(${PROJECT_NAME} PRIVATE ${JNI_LIBRARIES})

set(ANDROID_SDK "/home/uwe/android/android-sdk-linux")
set(ANDROID_PLATFORM "android-16")
set(BINDINGS_VERSION "0.5.0")

set(ANTFLAGS
	-Dandroid.sdk=${ANDROID_SDK} -Dandroid.platform=${ANDROID_PLATFORM}
	-Dprefix=${CMAKE_INSTALL_PREFIX} -Dbindings.version=${BINDINGS_VERSION}
	-Dant.build.javac.target=1.6 -Dant.build.javac.source=1.6)

add_custom_target(ant COMMAND
	ant -S -q ${ANTFLAGS} utils &&
	ant -S -q anttasks &&
	ant -S -q ${ANTFLAGS} aar &&
	ant -S -q ${ANTFLAGS} pom
	DEPENDS ${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lib${PROJECT_NAME})
