cmake_minimum_required(VERSION 3.13)

project(deepin-qfiledialog)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)

if(PROJECT_VERSION_MAJOR EQUAL 6)
  set(VERSION_SUFFIX 6)
  find_package(QT NAMES Qt6 REQUIRED COMPONENTS Core)
else()
  set(VERSION_SUFFIX)
  find_package(QT NAMES Qt5 REQUIRED COMPONENTS Core)
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX /usr)
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
string(TOLOWER ${CMAKE_BUILD_TYPE} CMAKE_BUILD_TYPE_LOWER)
if(CMAKE_BUILD_TYPE_LOWER STREQUAL release)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")
elseif(CMAKE_BUILD_TYPE_LOWER STREQUAL debug)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g")
endif()

# Some util functions
function(dtk_add_plugin)
  set(options)
  set(oneValueArgs NAME OUTPUT_NAME OUTPUT_DIR INSTALL_DIR)
  set(multiValueArgs SOURCES HEADERS RESOURCES DEPENDENCIES DEFINITIONS OPTIONS INCLUDE_DIRS)
  cmake_parse_arguments(TARGET "${options}" "${oneValueArgs}"
                        "${multiValueArgs}" ${ARGN})
  set(RAW_SOURCES ${TARGET_SOURCES} ${TARGET_RESOURCES})
  foreach(HEADER ${TARGET_HEADERS})
    get_filename_component(HEADER_ABS_PATH ${HEADER} ABSOLUTE BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    list(APPEND ABS_PATH_SOURCES ${SOURCE_ABS_PATH})
    get_filename_component(HEADER_DIR ${HEADER_ABS_PATH} DIRECTORY)
    list(APPEND TARGET_INTERFACE_INCLUDE_DIRS ${HEADER_DIR})
  endforeach()
  list(REMOVE_DUPLICATES TARGET_INTERFACE_INCLUDE_DIRS)
  foreach(SOURCE ${RAW_SOURCES})
    get_filename_component(SOURCE_ABS_PATH ${SOURCE} ABSOLUTE BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    list(APPEND ABS_PATH_SOURCES ${SOURCE_ABS_PATH})
  endforeach()
  add_library(${TARGET_NAME} SHARED ${TARGET_HEADERS} ${TARGET_SOURCES} ${TARGET_RESOURCES})

  # set rpath
  set_target_properties(${TARGET_NAME}
            PROPERTIES
            INSTALL_RPATH "$ORIGIN/../../lib:$ORIGIN"
            BUILD_WITH_INSTALL_RPATH TRUE
            )

  target_compile_definitions(${TARGET_NAME} PRIVATE ${TARGET_DEFINITIONS})
  target_compile_options(${TARGET_NAME} PRIVATE ${TARGET_OPTIONS})
  target_link_libraries(${TARGET_NAME} PRIVATE ${TARGET_DEPENDENCIES})
  target_include_directories(${TARGET_NAME} PRIVATE ${TARGET_INCLUDE_DIRS})
  if(TARGET_OUTPUT_NAME)
    set_target_properties(${TARGET_NAME} PROPERTIES OUTPUT_NAME ${OUTPUT_NAME})
  endif()
  if(TARGET_OUTPUT_DIR)
    set_target_properties(${TARGET_NAME}
                          PROPERTIES
                          LIBRARY_OUTPUT_DIRECTORY ${TARGET_OUTPUT_DIR})
  endif()
  if(TARGET_INSTALL_DIR)
    install(TARGETS ${TARGET_NAME} DESTINATION ${TARGET_INSTALL_DIR})
  endif()
endfunction()

set(PLUGIN_OUTPUT_BASE_DIR ${CMAKE_BINARY_DIR}/plugins CACHE STRING "Plugin output base path")
set(PLUGIN_INSTALL_BASE_DIR ${CMAKE_INSTALL_LIBDIR}/qt${QT_VERSION_MAJOR}/plugins CACHE STRING "Plugin install base path")

add_subdirectory(platformthemeplugin)

