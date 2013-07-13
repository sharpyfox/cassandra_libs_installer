CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)
set(PROJECT_NAME_STR cassandra)
project(${PROJECT_NAME_STR} C CXX)

#-------------------
# The version number
#-------------------

set (${PROJECT_NAME_STR}_VERSION_MAJOR 0)
set (${PROJECT_NAME_STR}_VERSION_MINOR 1)
set (${PROJECT_NAME_STR}_VERSION_STRING ${${PROJECT_NAME_STR}_VERSION_MAJOR}.${PROJECT_NAME_STR}_VERSION_MINOR)

#-------------------
# cassandra static and shared
#-------------------
SET(LIBDIRNAME "lib")
set(CASSANDRA_VERSION "1.1.9" CACHE STRING "cassandra version to retreive interface")

MESSAGE("Will install libraries to ${CMAKE_INSTALL_PREFIX}/${LIBDIRNAME}")

MESSAGE("Creating project structure")
FILE(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/interface)
FILE(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/include)
FILE(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/src)
MESSAGE("Download cassandra ${CASSANDRA_VERSION} thrift interface file")
FILE(DOWNLOAD https://raw.github.com/apache/cassandra/cassandra-${CASSANDRA_VERSION}/interface/cassandra.thrift ${PROJECT_SOURCE_DIR}/interface/cassandra.thrift SHOW_PROGRESS)
MESSAGE("Generate cpp interface")
EXECUTE_PROCESS(COMMAND thrift -out ${PROJECT_SOURCE_DIR}/src/ --gen cpp ${PROJECT_SOURCE_DIR}/interface/cassandra.thrift)
MESSAGE("Move headers files")
FILE( GLOB_RECURSE pattern_files RELATIVE "${PROJECT_SOURCE_DIR}/src" "*.h" ) 
FOREACH( pattern_file ${pattern_files} )
    FILE(RENAME src/${pattern_file} ${PROJECT_SOURCE_DIR}/include/${pattern_file})
ENDFOREACH(pattern_file)

set(PROJECT_LIB_NAME ${PROJECT_NAME_STR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${LIBDIRNAME})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${LIBDIRNAME})
set(PROJECT_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)

include_directories(${INCLUDES} ${PROJECT_INCLUDE_DIR} ${PROJECT_SOURCE_DIR}/src)
file(GLOB SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cpp)

add_library(${PROJECT_LIB_NAME} SHARED ${SRC_FILES})
add_library(${PROJECT_LIB_NAME}-static STATIC ${SRC_FILES})

target_link_libraries(${PROJECT_LIB_NAME} ${LIBS})
SET_TARGET_PROPERTIES(${PROJECT_LIB_NAME}-static PROPERTIES OUTPUT_NAME ${PROJECT_LIB_NAME})
SET_TARGET_PROPERTIES(${PROJECT_LIB_NAME} PROPERTIES VERSION ${${PROJECT_NAME_STR}_VERSION_MAJOR}.${${PROJECT_NAME_STR}_VERSION_MINOR} SOVERSION ${${PROJECT_NAME_STR}_VERSION_MAJOR})

set_property(
  TARGET ${PROJECT_LIB_NAME} ${PROJECT_LIB_NAME}-static
  APPEND PROPERTY COMPILE_FLAGS "-Wno-deprecated -Winit-self -fno-omit-frame-pointer -std=c++0x -fno-deduce-init-list -msse3 -Wno-unused-but-set-variable -Wno-psabi -D__GXX_EXPERIMENTAL_CXX0X__=1 -DHAVE_NETINET_IN_H -DHAVE_INTTYPES_H")

#-------------------
# install target
#-------------------

FILE(GLOB INSTALL_HEADERS "${PROJECT_INCLUDE_DIR}/*.h")
INSTALL(FILES ${INSTALL_HEADERS} DESTINATION include/cassandra)

INSTALL(TARGETS ${PROJECT_LIB_NAME}-static ${PROJECT_LIB_NAME}
  LIBRARY DESTINATION ${LIBDIRNAME}
  ARCHIVE DESTINATION ${LIBDIRNAME})

#-------------------
# uninstall target
#-------------------

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)