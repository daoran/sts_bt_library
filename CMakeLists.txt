cmake_minimum_required(VERSION 2.8.3)
project(sts_bt_library)

## Use C++11
add_definitions(-Wall -lglut -lGL -std=c++17)

# openMP
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")

# Set to Release in order to speed up the program significantly
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release) #None, Debug, Release, RelWithDebInfo, MinSizeRel
endif()


# Make sure symbols are there, and not fail at runtime
set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined")

find_package(catkin REQUIRED 
  COMPONENTS
    roscpp
    std_msgs

    actionlib
    actionlib_msgs 

    ### message_generation is referenced implicitly by actionlib_msgs
    message_generation
    message_runtime
    sts_virtual_interface_msgs
)

add_message_files(
    DIRECTORY msg
    FILES
        bt_log.msg
        bt_event.msg
        bt_draw_event.msg
        bt_draw_update.msg
    )

#add_action_files(
#  DIRECTORY action
#  FILES BT.action
#)
#
#add_service_files(
#    FILES
#    BTService.srv
#)
generate_messages(
  DEPENDENCIES
#    actionlib_msgs
    std_msgs
)

#########################################################
# FIND GLUT
#########################################################
find_package(GLUT REQUIRED)
include_directories(${GLUT_INCLUDE_DIRS})
link_directories(${GLUT_LIBRARY_DIRS})
add_definitions(${GLUT_DEFINITIONS})
if(NOT GLUT_FOUND)
    message(ERROR " GLUT not found!")
endif(NOT GLUT_FOUND)

#########################################################
# FIND OPENGL
#########################################################
find_package(OpenGL REQUIRED)
include_directories(${OpenGL_INCLUDE_DIRS})
link_directories(${OpenGL_LIBRARY_DIRS})
add_definitions(${OpenGL_DEFINITIONS})
if(NOT OPENGL_FOUND)
    message(ERROR " OPENGL not found!")
endif(NOT OPENGL_FOUND)

#########################################################
# FIND X11
#########################################################
MESSAGE ( STATUS "Searching for X11..." )
FIND_PACKAGE ( X11 REQUIRED )
IF ( X11_FOUND )
      INCLUDE_DIRECTORIES ( ${X11_INCLUDE_DIR} )
      LINK_LIBRARIES ( ${X11_LIBRARIES} )
      MESSAGE ( STATUS " X11_INCLUDE_DIR: " ${X11_INCLUDE_DIR} )
      MESSAGE ( STATUS " X11_LIBRARIES: " ${X11_LIBRARIES} )
ENDIF ( X11_FOUND )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS
    include
    ${EXTERNAL_INCLUDE_DIRS}

  LIBRARIES
    #${EXTERNAL_LIBS}
    ${PROJECT_NAME}_core

  CATKIN_DEPENDS
    std_msgs
    actionlib
    actionlib_msgs
    message_generation
    message_runtime
    sts_virtual_interface_msgs

    #sts libs if needed

#  DEPENDS
)

###########
## Build ##
###########

#cascade files recusively - could be usefull if multiple files exist
FILE(GLOB_RECURSE HeaderFiles "include/*.h")
FILE(GLOB_RECURSE SourceFiles "*.cpp")

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

## Declare a cpp library
add_library(${PROJECT_NAME}_core
  ${HeaderFiles}
  ${SourceFiles}
)

## Declare cpp executables
#add_executable(${PROJECT_NAME}
#  ${SourceFiles}
#)

## Specify libraries to link executable targets against
target_link_libraries(${PROJECT_NAME}_core
  ${catkin_LIBRARIES}
  ${OPENGL_LIBRARIES} 
  ${GLUT_LIBRARY}
  ${X11_LIBRARIES}
)

add_dependencies(${PROJECT_NAME}_core
    ${${PROJECT_NAME}_EXPORTED_TARGETS}
    ${catkin_EXPORTED_TARGETS}
  )

#target_link_libraries(${PROJECT_NAME}
#  ${PROJECT_NAME}_core
#  ${catkin_LIBRARIES}
#)
