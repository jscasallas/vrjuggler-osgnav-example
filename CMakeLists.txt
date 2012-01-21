# Minimal VR Juggler+OpenSourceGraph Build
# CMake cross-platform build system recipe
# 2009 Ryan Pavlik <rpavlik@iastate.edu>
# http://academic.cleardefinition.com
# Iowa State University HCI Graduate Program/VRAC

cmake_minimum_required(VERSION 2.6)

# Set package properties
project(osgNav)

###
# CMake Modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
include(UseBackportedModules)
include(SearchProgramFilesForOpenSceneGraph)
include(CreateLaunchers)
include(GetDirectoryList)

###
# Dependencies

# VR Juggler
find_package(VRJuggler 3.1 REQUIRED)
include_directories(${VRJUGGLER_INCLUDE_DIRS})
list(APPEND RUNTIME_LIBRARY_DIRS ${VRJUGGLER_RUNTIME_LIBRARY_DIRS})
add_definitions(${VRJUGGLER_DEFINITIONS})
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${VRJUGGLER_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${VRJUGGLER_CXX_FLAGS}")

# OpenSceneGraph
find_package(OpenSceneGraph REQUIRED COMPONENTS osgUtil osgDB)
include_directories(${OPENSCENEGRAPH_INCLUDE_DIRS})
get_directory_list(OPENSCENEGRAPH_RUNTIME_LIBRARY_DIRS ${OPENSCENEGRAPH_LIBRARIES})
list(APPEND RUNTIME_LIBRARY_DIRS ${OPENSCENEGRAPH_RUNTIME_LIBRARY_DIRS})
if(WIN32)
	list(APPEND RUNTIME_LIBRARY_DIRS ${OPENSCENEGRAPH_RUNTIME_LIBRARY_DIRS}/../bin)
endif()

###
# Build the project
###
set(SOURCES
	main.cpp
	nav.h
	OsgNav.cpp
	OsgNav.h)

#--------------------------------------------------------------------------------
# For Apple set the icns file containing icons
IF(APPLE)
  # set how it shows up in the Info.plist file
  SET(MACOSX_BUNDLE_ICON_FILE vrjuggler.icns) 
  # set where in the bundle to put the icns file
  SET_SOURCE_FILES_PROPERTIES(/usr/local/Cellar/vrjuggler-3.0/HEAD/share/vrjuggler-3.1.6/data/bundle/vrjuggler.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  # include the icns file in the target
  SET(SOURCES ${SOURCES} /usr/local/Cellar/vrjuggler-3.0/HEAD/share/vrjuggler-3.1.6/data/bundle/vrjuggler.icns)
ENDIF(APPLE)

add_executable(osgnav MACOSX_BUNDLE ${SOURCES})

target_link_libraries(osgnav
	${OPENSCENEGRAPH_LIBRARIES}
	${VRJUGGLER_LIBRARIES})
#create_default_target_launcher(osgnav
#	FORWARD_ARGS
#	RUNTIME_LIBRARY_DIRS
#	${RUNTIME_LIBRARY_DIRS}
#	ENVIRONMENT
#	${VRJUGGLER_ENVIRONMENT})

#--------------------------------------------------------------------------------
# Now the installation stuff below
#--------------------------------------------------------------------------------
SET(APPS "\${CMAKE_INSTALL_PREFIX}/bin/osgnav")
IF(APPLE)
  SET(plugin_dest_dir osgnav.app/Contents/MacOS)
  SET(APPS "\${CMAKE_INSTALL_PREFIX}/osgnav.app")
ENDIF(APPLE)
IF(WIN32)
  SET(APPS "\${CMAKE_INSTALL_PREFIX}/bin/osgnav.exe")
ENDIF(WIN32)

#--------------------------------------------------------------------------------
# Install the osgnav application, on Apple, the bundle is at the root of the
# install tree, and on other platforms it'll go into the bin directory.
INSTALL(TARGETS osgnav
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION bin COMPONENT Runtime
    )

#--------------------------------------------------------------------------------
# Use BundleUtilities to get all other dependencies for the application to work.
# It takes a bundle or executable along with possible plugins and inspects it
# for dependencies.  If they are not system dependencies, they are copied.

# directories to look for dependencies
SET(DIRS ${RUNTIME_LIBRARY_DIRS})

# Now the work of copying dependencies into the bundle/package
# The quotes are escaped and variables to use at install time have their $ escaped
# An alternative is the do a configure_file() on a script and use install(SCRIPT  ...).
INSTALL(CODE "
    file(GLOB_RECURSE QTPLUGINS
      \"\${CMAKE_INSTALL_PREFIX}/${plugin_dest_dir}/plugins/*${CMAKE_SHARED_LIBRARY_SUFFIX}\")
    include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"${DIRS}\")
    " COMPONENT Runtime)


# To Create a package, one can run "cpack -G DragNDrop CPackConfig.cmake" on Mac OS X
# where CPackConfig.cmake is created by including CPack
# And then there's ways to customize this as well
set(CPACK_BINARY_DRAGNDROP ON)
include(CPack)
