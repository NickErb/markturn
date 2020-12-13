# Build for markturn

cmake_minimum_required(VERSION 3.18.4)

project(markturn VERSION "1.0.0")
message("VERSION: " ${PROJECT_VERSION})
message("MAJOR VERSION: " ${PROJECT_VERSION_MAJOR})

option(LINK_STATIC "Build with static library" OFF)

# find curl
find_package(CURL REQUIRED)
message("-- curl include: " ${CURL_INCLUDE_DIR})
message("-- curl library: " ${CURL_LIBRARY})

# find libxml2
find_package(LibXml2 REQUIRED)
message("-- libxml2 include: " ${LIBXML2_INCLUDE_DIR})
message("-- libxml2 library: " ${LIBXML2_LIBRARY})

# set yaml directory
set(YAMLCPP_DIR /usr/include/yaml-cpp)

# set json directory
find_path(JSONCPP_DIR NAMES json HINTS /usr/include/ /usr/include/jsoncpp/ REQUIRED)
message("-- json include: " ${JSONCPP_DIR})

# libmarkturn include directories
set(LIB_INCLUDE_DIRS ${LIBXML2_INCLUDE_DIR} ${YAMLCPP_DIR} ${JSONCPP_DIR})

# libmarkturn sources
file(GLOB LIB_SOURCES "src/*Format.cpp" "src/libmarkturn.cpp")

# markturn object library
add_library(markturnobj OBJECT ${LIB_SOURCES})
target_include_directories(markturnobj PUBLIC ${LIB_INCLUDE_DIRS})

# build markturn
set(MARKTURN_EXE "src/markturn.cpp")
add_executable(markturn ${MARKTURN_EXE})
target_include_directories(markturn PUBLIC ${CURL_INCLUDE_DIR})
target_link_libraries(markturn PUBLIC ${CURL_LIBRARY})
if(LINK_STATIC)
    message("-- satic library")
    target_link_Libraries(markturn PUBLIC markturn_static)
else()
    message("-- shared library")
    target_link_Libraries(markturn PUBLIC markturn_shared)
endif()
install(TARGETS markturn RUNTIME)

# static library
add_library(markturn_static STATIC $<TARGET_OBJECTS:markturnobj>)
target_link_libraries(markturn_static PUBLIC ${LIBXML2_LIBRARY})
set_target_properties(markturn_static PROPERTIES OUTPUT_NAME markturn)
install(TARGETS markturn_static LIBRARY)

# shared library
add_library(markturn_shared SHARED $<TARGET_OBJECTS:markturnobj>)
target_link_libraries(markturn_shared PUBLIC ${LIBXML2_LIBRARY})
set_target_properties(markturn_shared PROPERTIES OUTPUT_NAME markturn)
set_target_properties(markturn_shared PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(markturn_shared PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})
install(TARGETS markturn_shared LIBRARY)

#run example
add_custom_target(run DEPENDS markturn
                      COMMAND ./markturn)

# setup testing
enable_testing()

# all .cpp files in test/ independent test programs
file(GLOB TEST_SOURCES test/*.cpp)
foreach(TESTFILE ${TEST_SOURCES})
    message("-- added test: ${TESTFILE}")
    get_filename_component(TEST_NAME ${TESTFILE} NAME_WLE)
    add_executable(${TEST_NAME} ${TESTFILE})
    target_link_libraries(${TEST_NAME} PUBLIC markturn_shared)
    target_include_directories(${TEST_NAME} PUBLIC ${CMAKE_SOURCE_DIR}/src)
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})
endforeach()

# packaging
set(CPACK_PACKAGE_NAME "markturn")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_CONTACT "nme26@zips.uakron.edu")
include(CPack)
