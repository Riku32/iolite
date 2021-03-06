cmake_minimum_required(VERSION 3.16)
set(CMAKE_CXX_STANDARD 17)

project(iolite)

# Lexer and Parser
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

set(LEXER_OUT "${CMAKE_CURRENT_BINARY_DIR}/lexer.c")
set(PARSER_OUT "${CMAKE_CURRENT_BINARY_DIR}/parser.c")

flex_target(Scanner ${CMAKE_CURRENT_LIST_DIR}/src/lexer.l ${LEXER_OUT} DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/lexer.h)
bison_target(Parser ${CMAKE_CURRENT_LIST_DIR}/src/parser.y ${PARSER_OUT} DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/parser.h)

# Get sources
file(GLOB_RECURSE SRC
        ${CMAKE_CURRENT_LIST_DIR}/src/*.h
        ${CMAKE_CURRENT_LIST_DIR}/src/*.c
        ${CMAKE_CURRENT_LIST_DIR}/src/*.cpp
        ${CMAKE_CURRENT_LIST_DIR}/src/*.hpp)

message(${LEXER_OUT})

add_executable(iolite
        ${SRC}
        ${LEXER_OUT}
        ${PARSER_OUT})

target_include_directories(iolite PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src)
target_include_directories(iolite PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

target_precompile_headers(iolite PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src/pch.h)

# Add LLVM
find_package(LLVM 10.0.0 REQUIRED CONFIG)
target_include_directories(iolite PRIVATE ${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})
llvm_map_components_to_libnames(llvm_libs support core)
target_link_libraries(iolite ${llvm_libs} ${FLEX_LIBRARIES} ${BISON_LIBRARIES})