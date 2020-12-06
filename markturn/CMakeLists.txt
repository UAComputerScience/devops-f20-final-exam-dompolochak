# Build for markturn

cmake_minimum_required(VERSION 3.18.4)

project(markturn)

# Find neccessary packages
find_package(CURL REQUIRED)
find_package(LibXml2 REQUIRED)
find_package(yaml-cpp REQUIRED)
find_package(jsoncpp REQUIRED)

# get jsoncpp include directory
# @NOTE Need to explain why you are not using the find_package() results
get_target_property(JSON_INC_PATH jsoncpp_lib INTERFACE_INCLUDE_DIRECTORIES)

# get yaml include directory
# @NOTE Be consistent on capitalization of comments
set(YAML_CPP_DIR /usr/include/yaml-cpp)

# option to use static or shared libary
option(LINK_STATIC "Build with static libary" ON)

# Create executable
add_executable(markturn "src/markturn.cpp")
if(LINK_STATIC)
    message("-- Static")
    target_link_libraries(markturn PUBLIC markturn_static)
else()
    message("-- Shared")
    target_link_libraries(markturn PUBLIC markturn_shared)
endif()
# install executable
install(TARGETS markturn RUNTIME)

# Libmarkturn library sources
file(GLOB LIBRARY_SOURCES "src/*Format.cpp" "src/libmarkturn.cpp")

# Create object library markturnobj
add_library(markturnobj OBJECT ${LIBRARY_SOURCES})
target_include_directories(markturnobj PUBLIC ${LIBXML2_INCLUDE_DIR} ${YAML_CPP_DIR} ${JSON_INC_PATH})
set_target_properties(markturnobj PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Create shared library libmarkturn_shared
add_library(markturn_shared SHARED $<TARGET_OBJECTS:markturnobj>)
target_link_libraries(markturn_shared PUBLIC ${LIBXML2_LIBRARY} ${YAML_CPP_LIBRARIES} ${JSONCPP_LIBRARY})
install(TARGETS markturn_shared LIBRARY)

# Create static library libmarkturn_static
add_library(markturn_static STATIC $<TARGET_OBJECTS:markturnobj>)
target_link_libraries(markturn_static PUBLIC ${LIBXML2_LIBRARY} ${YAML_CPP_LIBRARIES} ${JSONCPP_LIBRARY})
install(TARGETS markturn_static LIBRARY)

# run target
add_custom_target(run DEPENDS markturn COMMAND ./markturn)

# setup testing
include(CTest)
enable_testing()

# glob test files
file(GLOB TEST_SOURCES test/*.cpp)
foreach(TEST_FILE ${TEST_SOURCES})
    get_filename_component(TEST_NAME ${TEST_FILE} NAME_WLE)
    add_executable(${TEST_NAME} ${TEST_FILE})
    target_include_directories(${TEST_NAME} PUBLIC ${CMAKE_SOURCE_DIR}/src)
    target_link_libraries(${TEST_NAME} PUBLIC markturn_shared)
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})   
endforeach()

# CPACK
include(CPack)
