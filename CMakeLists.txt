############################################################################
# CMakeLists.txt
# Copyright (C) 2019  Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#
############################################################################

cmake_minimum_required(VERSION 3.0)
project(MSAAUDIO VERSION 0.1 LANGUAGES CXX)

option(ENABLE_SHARED "Build shared library." YES)
option(ENABLE_STATIC "Build static library." NO)

include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}")

if(LINPHONE_BUILDER_GROUP_EXTERNAL_SOURCE_PATH_BUILDERS)
	include("${EP_ms2_CONFIG_DIR}/Mediastreamer2Config.cmake")
	include("${EP_bctoolbox_CONFIG_DIR}/BcToolboxConfig.cmake")
	include("${EP_ortp_CONFIG_DIR}/OrtpConfig.cmake")
else()
	find_package(Mediastreamer2 REQUIRED)
	find_package(BcToolbox REQUIRED)
	find_package(ORTP REQUIRED)
endif()

include_directories(
        ${CMAKE_CURRENT_BINARY_DIR}
	${MEDIASTREAMER2_INCLUDE_DIRS}
)

set(LIBS ${MEDIASTREAMER2_LIBRARIES} aaudio ${ORTP_LIBRARIES} ${BCTOOLBOX_CORE_LIBRARIES})

set(SOURCE_FILES androidsound_aaudio.cpp)

if(ENABLE_STATIC)
	add_library(msaaudio-static STATIC ${SOURCE_FILES})
	set_target_properties(msaaudio-static PROPERTIES OUTPUT_NAME msaaudio)
	target_link_libraries(msaaudio-static ${LIBS})
	install(TARGETS msaaudio-static
		RUNTIME DESTINATION lib/mediastreamer/plugins
		LIBRARY DESTINATION lib/mediastreamer/plugins
		ARCHIVE DESTINATION lib/mediastreamer/plugins
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
endif()
if(ENABLE_SHARED)
	add_library(msaaudio MODULE ${SOURCE_FILES})
	target_link_libraries(msaaudio ${LIBS})
	install(TARGETS msaaudio
		RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/mediastreamer/plugins
		FRAMEWORK DESTINATION Frameworks
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
	)
endif()