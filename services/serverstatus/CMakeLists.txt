cmake_minimum_required(VERSION 2.6)

project (serverstatus CXX)

set(SERVERSTATUS_HEADERS ServerStatus.h)

set(PROJECT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(SERVERSTATUS_CODEGEN_DIR "${PROJECT_BINARY_DIR}/codegen")

PREPEND(SERVERSTATUS_HEADERS_PATHS ${PROJECT_SOURCE_DIR} ${SERVERSTATUS_HEADERS})

CODEGEN_FILES(SERVERSTATUS_CODEGEN_SOURCES ${SERVERSTATUS_CODEGEN_DIR} ${SERVERSTATUS_HEADERS})

add_custom_command(OUTPUT ${SERVERSTATUS_CODEGEN_SOURCES}
    COMMAND ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/ngrestcg -i "${PROJECT_SOURCE_DIR}" -o ${SERVERSTATUS_CODEGEN_DIR} -t service ${SERVERSTATUS_HEADERS}
    DEPENDS ${SERVERSTATUS_HEADERS_PATHS}
)

file(GLOB SERVERSTATUS_SOURCES ${PROJECT_SOURCE_DIR}/*.cpp)

list(APPEND SERVERSTATUS_SOURCES ${SERVERSTATUS_CODEGEN_SOURCES})

include_directories(${PROJECT_SOURCE_DIR})

add_library(serverstatus MODULE ${SERVERSTATUS_SOURCES})
if (APPLE) # cmake sets .so extension for modules under mac os x
    set_target_properties(serverstatus PROPERTIES SUFFIX ".dylib")
endif()

set_target_properties(serverstatus PROPERTIES PREFIX "")
set_target_properties(serverstatus PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY "${PROJECT_SERVICES_DIR}"
)

target_link_libraries(serverstatus ngrestutils ngrestcommon ngrestjson ngrestengine)
