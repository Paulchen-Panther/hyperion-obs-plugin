# --- Detect if the plugin is build out of tree or not ---
if(CMAKE_PROJECT_NAME STREQUAL "obs-studio")
  set(BUILD_OUT_OF_TREE OFF)
else()
  set(BUILD_OUT_OF_TREE ON)
  cmake_minimum_required(VERSION 3.18)
endif()

project(hyperion-obs VERSION 1.0.2)
set(PROJECT_FULL_NAME "hyperion-obs")

message(STATUS "${PROJECT_NAME} version: ${PLUGIN_VERSION}")
add_library(${PROJECT_NAME} MODULE)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/version.h.in ${CMAKE_CURRENT_SOURCE_DIR}/version.h)

if(BUILD_OUT_OF_TREE)
	find_package(libobs REQUIRED)
	find_package(obs-frontend-api REQUIRED)
	include(cmake/ObsPluginHelpers.cmake)
	set(OBS_FRONTEND_API_NAME "obs-frontend-api")
else()
	set(OBS_FRONTEND_API_NAME "frontend-api")
endif()

# Import QT
find_qt(COMPONENTS Widgets Core Network REQUIRED)
target_link_libraries(${PROJECT_NAME} Qt::Core Qt::Widgets Qt::Network)
set_target_properties(${PROJECT_NAME} PROPERTIES AUTOMOC ON AUTOUIC ON AUTORCC ON)

# Find Flatbuffers and compile schema
find_package(Flatbuffers CONFIG REQUIRED Flatbuffers flatc)

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/FlatBufferSchema_generated.h
	COMMAND flatbuffers::flatc -c --no-includes --gen-mutable -o ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/FlatBufferSchema.fbs
	DEPENDS flatbuffers::flatc ${CMAKE_CURRENT_SOURCE_DIR}/src/FlatBufferSchema.fbs
)

add_custom_target(schema DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/FlatBufferSchema_generated.h)
add_dependencies(${PROJECT_NAME} schema)

# Set project includes
target_include_directories(${PROJECT_NAME} PRIVATE
 	${CMAKE_CURRENT_BINARY_DIR}
 	${FLATBUFFERS_INCLUDE_DIRS}
)

# Set project libraries
FILE ( GLOB hyperion_obs_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h" "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/src/*.ui" )
target_link_libraries(${PROJECT_NAME}
	OBS::${OBS_FRONTEND_API_NAME}
	OBS::libobs
	flatbuffers
)

# Set compiled Flatbuffers and project source files
target_sources(${PROJECT_NAME} PRIVATE ${hyperion_obs_SOURCES})

if(OS_WINDOWS)
	set(PLUGIN_AUTHOR "hyperion-project")
	set(PLUGIN_GIT "hyperion-obs-plugin")
	set(PLUGIN_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
	set(PLUGIN_ICON "${CMAKE_CURRENT_SOURCE_DIR}/resources/installer.ico")
	configure_file(package/Windows.iss.in ${CMAKE_BINARY_DIR}/Windows.iss)
	target_link_libraries(${PROJECT_NAME} w32-pthreads)
elseif(OS_MACOS)
	set(BUNDLE_IDENTIFIER "com.hyperion-project.hyperion-obs")
	configure_file(package/macOS.pkgproj.in ${CMAKE_BINARY_DIR}/macOS.pkgproj)

	set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
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

	target_compile_options(${PROJECT_NAME} PRIVATE -Wall)
else()
	if(BUILD_OUT_OF_TREE)
		if(NOT LIB_OUT_DIR)
			set(LIB_OUT_DIR "/lib/obs-plugins")
		endif()

		if(NOT DATA_OUT_DIR)
			set(DATA_OUT_DIR "/share/obs/obs-plugins/${PROJECT_NAME}")
		endif()

		set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
		install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/${LIB_OUT_DIR})
		install(DIRECTORY data/locale DESTINATION ${CMAKE_INSTALL_PREFIX}/${DATA_OUT_DIR})
	else()
		target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_SOURCE_DIR}/UI/obs-frontend-api")
		set_target_properties(${PROJECT_NAME} PROPERTIES FOLDER "plugins/hyperion-obs")
	endif()
endif()

setup_plugin_target(${PROJECT_NAME})
