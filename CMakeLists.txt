cmake_minimum_required(VERSION 3.16...3.21)
file (STRINGS ".version" PLUGIN_VERSION)
project(hyperion-obs VERSION ${PLUGIN_VERSION})

message(STATUS "${CMAKE_PROJECT_NAME} version: ${PLUGIN_VERSION}")
add_library(${CMAKE_PROJECT_NAME} MODULE)

# Import libobs
find_package(libobs QUIET)
find_package(LibObs QUIET)
include(cmake/ObsPluginHelpers.cmake)

# Import OBS Frontend API
find_package(obs-frontend-api REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::obs-frontend-api)

# Import QT
find_qt(COMPONENTS Widgets Core Network REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE Qt::Core Qt::Widgets Qt::Network)
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES AUTOMOC ON AUTOUIC ON AUTORCC ON)

# Compile Flatbuffers library or use Flatbuffers library from system
option(USE_SYSTEM_FLATBUFFERS_LIBS "Use Flatbuffers library from system (default: OFF)" OFF)

if (USE_SYSTEM_FLATBUFFERS_LIBS)
	find_program(FLATBUFFERS_FLATC_EXECUTABLE NAMES flatc REQUIRED)
	find_package(Flatbuffers REQUIRED)
else ()
	set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared flatbuffers library")
	set(FLATBUFFERS_BUILD_TESTS OFF CACHE BOOL "Build Flatbuffers with tests")
	add_subdirectory(external/flatbuffers)

	# export the flatc compiler
	export(TARGETS flatc FILE "${CMAKE_BINARY_DIR}/flatc_export.cmake")

	# define the include for the flatbuffers library
	set(FLATBUFFERS_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/external/flatbuffers/include")

	# define the flatc executable 
	set(FLATBUFFERS_FLATC_EXECUTABLE "$<TARGET_FILE:flatc>")
endif()

set(FLATBUFFERS_FLATC_EXECUTABLE ${FLATBUFFERS_FLATC_EXECUTABLE})
set(FLATBUFFERS_INCLUDE_DIRS ${FLATBUFFERS_INCLUDE_DIRS})

include_directories(
	${CMAKE_CURRENT_BINARY_DIR}
	${FLATBUFFERS_INCLUDE_DIRS}
)

# Compile Flatbuffers schema file
set(FLATBUFFERS_SCHEMA "${CMAKE_SOURCE_DIR}/src/FlatBufferSchema.fbs")
set(FLATBUFFERS_HEADER "${CMAKE_BINARY_DIR}/FlatBufferSchema_generated.h")

add_custom_command(
	OUTPUT ${FLATBUFFERS_HEADER}
	COMMAND ${FLATBUFFERS_FLATC_EXECUTABLE}
	ARGS -c --no-includes --gen-mutable -o ${CMAKE_CURRENT_BINARY_DIR} ${FLATBUFFERS_SCHEMA}
	DEPENDS ${FLATBUFFERS_FLATC_EXECUTABLE} ${FLATBUFFERS_SCHEMA}
	COMMENT "Building Flatbuffer header file"
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

set_source_files_properties(${FLATBUFFERS_HEADER} PROPERTIES GENERATED TRUE)

# Set project libraries
FILE ( GLOB hyperion_obs_SOURCES "${CMAKE_SOURCE_DIR}/src/*.h" "${CMAKE_SOURCE_DIR}/src/*.cpp" "${CMAKE_SOURCE_DIR}/src/*.ui" )
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE
	OBS::libobs
	OBS::obs-frontend-api
	flatbuffers
)

# Set compiled Flatbuffers and project source files
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
	${hyperion_obs_SOURCES}
	${FLATBUFFERS_HEADER}
)

option(GLOBAL_INSTALLATION "Whether to install for all users (default: OFF)" OFF)

if(OS_WINDOWS)
	set(PLUGIN_AUTHOR "hyperion-project")
	set(PLUGIN_GIT "hyperion-obs-plugin")
	set(PLUGIN_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
	set(PLUGIN_ICON "${CMAKE_SOURCE_DIR}/resources/installer.ico")
	configure_file(package/Windows.iss.in ${CMAKE_BINARY_DIR}/Windows.iss)
elseif(OS_MACOS)
	set(BUNDLE_IDENTIFIER "com.hyperion-project.hyperion-obs")
	configure_file(package/macOS.pkgproj.in ${CMAKE_BINARY_DIR}/macOS.pkgproj)

	set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES PREFIX "")
	# add_custom_command(
	# 	TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
	# 	COMMAND install_name_tool
	# 		-change @rpath/libobs.0.dylib @executable_path/../Frameworks/libobs.0.dylib
	# 		-change @rpath/libobs-frontend-api.dylib @executable_path/../Frameworks/libobs-frontend-api.dylib
	# 		-change ${CMAKE_PREFIX_PATH}/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore
	# 		-change @rpath/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore
	# 		-change ${CMAKE_PREFIX_PATH}/lib/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui
	# 		-change @rpath/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui
	# 		-change ${CMAKE_PREFIX_PATH}/lib/QtNetwork.framework/Versions/5/QtNetwork @executable_path/../Frameworks/QtNetwork.framework/Versions/5/QtNetwork
	# 		-change @rpath/QtNetwork.framework/Versions/5/QtNetwork @executable_path/../Frameworks/QtNetwork.framework/Versions/5/QtNetwork
	# 		-change ${CMAKE_PREFIX_PATH}/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets
	# 		-change @rpath/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets
	# 		$<TARGET_FILE:${CMAKE_PROJECT_NAME}>
	# )

	if (GLOBAL_INSTALLATION)
		# Install for all user
		install(TARGETS ${CMAKE_PROJECT_NAME} LIBRARY DESTINATION "/Library/Application Support/obs-studio/plugins/${CMAKE_PROJECT_NAME}/bin/")
		install(DIRECTORY data/locale/ DESTINATION "/Library/Application Support/obs-studio/plugins/${CMAKE_PROJECT_NAME}/data/locale")
	else()
		# Install into home directory
		install(TARGETS ${CMAKE_PROJECT_NAME} LIBRARY DESTINATION "$ENV{HOME}/Library/Application Support/obs-studio/plugins/${CMAKE_PROJECT_NAME}/bin/")
		install(DIRECTORY data/locale/ DESTINATION "$ENV{HOME}/Library/Application Support/obs-studio/plugins/${CMAKE_PROJECT_NAME}/data/locale")
	endif()

	target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
else()
	include(GNUInstallDirs)
	set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES PREFIX "")

	if (GLOBAL_INSTALLATION)
		# For *.deb installer or install for all user
		install(TARGETS ${CMAKE_PROJECT_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/obs-plugins)
		install(DIRECTORY data/locale/ DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/obs/obs-plugins/${CMAKE_PROJECT_NAME}/locale)
	else()
		# Installs into home directory
		install(TARGETS ${CMAKE_PROJECT_NAME} LIBRARY DESTINATION "$ENV{HOME}/.config/obs-studio/plugins/${CMAKE_PROJECT_NAME}/bin/64bit")
		install(DIRECTORY data/locale/ DESTINATION "$ENV{HOME}/.config/obs-studio/plugins/${CMAKE_PROJECT_NAME}/data/locale")
	endif()

	target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
endif()
