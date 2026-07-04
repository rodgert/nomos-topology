# SPDX-License-Identifier: GPL-3.0-or-later
cmake_minimum_required(VERSION 3.20)
project(nomos-topology VERSION 0.1.0 LANGUAGES CXX)

# Header-only INTERFACE library.
# The authoritative schema is the Clojure/Malli definition in src/.
# This target exposes only the C++ key-name constants used by kairos and
# kairos-grid when parsing EDN topology values via edn-cpp.
add_library(nomos-topology INTERFACE)
add_library(nomos::topology ALIAS nomos-topology)

target_include_directories(nomos-topology INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_compile_features(nomos-topology INTERFACE cxx_std_20)

# REUSE licence compliance target
find_program(REUSE_EXECUTABLE reuse)
if(REUSE_EXECUTABLE)
    add_custom_target(lint-reuse
        COMMAND ${REUSE_EXECUTABLE} lint
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Checking REUSE compliance"
        VERBATIM
    )
    add_test(NAME reuse-lint
        COMMAND ${REUSE_EXECUTABLE} lint
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )
endif()
