# Build for markturn

cmake_minimum_required(VERSION 3.18.4)

project(markturn VERSION "1.0.0")

option(LINK_STATIC "Build with static library" ON)

# Determine distribution
if(EXISTS /etc/os-release)
    file(STRINGS /etc/os-release DISTRO REGEX "^NAME=")
endif()

# find curl
find_package(CURL REQUIRED)
message("-- curl include: " ${CURL_INCLUDE_DIR})
message("-- curl library: " ${CURL_LIBRARY})

# find libxml2
find_package(LibXml2 REQUIRED)
message("-- LibXml2 include: " ${LIBXML2_INCLUDE_DIR})
message("-- LibXml2 library: " ${LIBXML2_LIBRARY})

#find yaml # @NOTE Use a space after the '#' of a comment. Be consistent.
find_package(yaml-cpp REQUIRED)
find_path(YAML_CPP_DIR NAMES yaml-cpp PATHS /usr/include/)
find_path(YAML_CPP_LIBRARY_DIR NAMES libyaml-cpp.so PATHS /usr/lib/x86_64-linux-gnu/ /usr/lib64/)

set(YAML_CPP_FULL_DIR ${YAML_CPP_DIR}/yaml-cpp)
set(YAML_CPP_LIBRARY_FULL_PATH ${YAML_CPP_LIBRARY_DIR}/libyaml-cpp.so)
message("-- YAML include: " ${YAML_CPP_FULL_DIR})
message("-- YAML library: " ${YAML_CPP_LIBRARY_FULL_PATH})

#Find JSON
find_package(jsoncpp REQUIRED)
find_path(JSONCPP_DIR NAMES jsoncpp json PATHS /usr/include)
find_path(JSONCPP_LIBRARY_DIR NAMES libjsoncpp.so PATHS /usr/lib/x86_64-linux-gnu/ /usr/lib64/)
message(${JSONCPP_DIR})

# if the distro is fedora then use json instead of jsoncpp
if(DISTRO MATCHES "Fedora")
    set(JSONCPP_FULL_DIR ${JSONCPP_DIR}/json)
    message("-- FEDORA")
else()
    set(JSONCPP_FULL_DIR ${JSONCPP_DIR}/jsoncpp)
endif()
set(JSONCPP_LIBRARY_FULL_PATH ${JSONCPP_LIBRARY_DIR}/libjsoncpp.so)
message("-- JSON include: " ${JSONCPP_FULL_DIR})
message("-- JSON library: " ${JSONCPP_LIBRARY_FULL_PATH})
# @NOTE What is the following code for? Add a comment.
set(MARKTURN_EXE "src/markturn.cpp")
add_executable(markturn ${MARKTURN_EXE})
target_link_libraries(markturn PUBLIC ${CURL_LIBRARY})

if(LINK_STATIC)
	message("-- Using Static Library")
	target_link_libraries(markturn PUBLIC markturn_static)
else()
	message("-- Using Shared Library")
	target_link_libraries(markturn PUBLIC markturn_shared)
endif()
install(TARGETS markturn RUNTIME)

# GLOB Libraries
file(GLOB LIBRARY_SOURCES "src/*Format.cpp" "src/libmarkturn.cpp")

# Create object of all library formats and libmarkturn
add_library(library_sources_obj OBJECT ${LIBRARY_SOURCES})
target_include_directories(library_sources_obj PUBLIC ${LIBXML2_INCLUDE_DIR})
target_include_directories(library_sources_obj PUBLIC ${YAML_CPP_FULL_DIR})
target_include_directories(library_sources_obj PUBLIC ${JSONCPP_FULL_DIR})

# Static markturn_static Library
add_library(markturn_static STATIC $<TARGET_OBJECTS:library_sources_obj>)
target_link_libraries(markturn_static PUBLIC ${LIBXML2_LIBRARY})
target_link_libraries(markturn_static PUBLIC ${YAML_CPP_LIBRARY_FULL_PATH})
target_link_libraries(markturn_static PUBLIC ${JSONCPP_LIBRARY_FULL_PATH})
install(TARGETS markturn_static LIBRARY)

# shared markturn_shared library
add_library(markturn_shared SHARED $<TARGET_OBJECTS:library_sources_obj>)
target_link_libraries(markturn_shared PUBLIC ${LIBXML2_LIBRARY})
target_link_libraries(markturn_shared PUBLIC ${YAML_CPP_LIBRARY_FULL_PATH})
target_link_libraries(markturn_shared PUBLIC ${JSONCPP_LIBRARY_FULL_PATH})
install(TARGETS markturn_shared LIBRARY)
# @NOTE What is the following section for?
enable_testing()
file(GLOB TEST_SOURCES "test/*.cpp")
foreach(TESTFILE ${TEST_SOURCES})
    get_filename_component(TEST_NAME ${TESTFILE} NAME_WLE)
    message("-- Adding test ${TEST_NAME}")

    add_executable(${TEST_NAME} ${TESTFILE})
    target_include_directories(${TEST_NAME} PUBLIC ${LIBXML2_INCLUDE_DIR})
    target_include_directories(${TEST_NAME} PUBLIC src)
    target_link_libraries(${TEST_NAME} PUBLIC ${LIBXML2_LIBRARY})
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})

endforeach()

Include(CPack) # @NOTE Be consistent. Use all lowercase for CMAKE instructions
# @NOTE Leave a single, blank line at the end of the file