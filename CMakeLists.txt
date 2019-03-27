cmake_minimum_required(VERSION 2.8)

project(ttyd)
set(PROJECT_VERSION "1.4.2")

find_package(Git)
if(GIT_FOUND)
    execute_process(
            COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
            WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
            RESULT_VARIABLE GIT_RET
            OUTPUT_VARIABLE GIT_COMMIT
            OUTPUT_STRIP_TRAILING_WHITESPACE
            ERROR_QUIET
    )
    if("${GIT_RET}" STREQUAL "0")
        set(PROJECT_VERSION "${PROJECT_VERSION}-${GIT_COMMIT}")
    endif()
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE")
if(CMAKE_VERSION VERSION_LESS "3.1")
    if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu99")
    else()
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
    endif()
else()
    set(CMAKE_C_STANDARD 99)
endif()

set(BUILD_SHARED_LIBS OFF CACHE INTERNAL "build static json-c")

add_subdirectory(deps/ae)
add_subdirectory(deps/wslay)
add_subdirectory(deps/json-c)

set(SOURCE_FILES
        src/server.c
        src/http.c
        src/protocol.c
        src/utils.c)

find_package(OpenSSL REQUIRED)
find_package(Libwebsockets 1.7.0 REQUIRED)

find_program(CMAKE_XXD NAMES xxd)
add_custom_command(OUTPUT html.h
        COMMAND ${CMAKE_XXD} -i index.html html.h
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/src
        COMMENT "Generating html.h from index.html")
list(APPEND SOURCE_FILES html.h)

set(INCLUDE_DIRS ${OPENSSL_INCLUDE_DIR} ${LIBWEBSOCKETS_INCLUDE_DIR})
set(LINK_LIBS pthread ae wslay json-c ${OPENSSL_LIBRARIES} ${LIBWEBSOCKETS_LIBRARIES})

if(APPLE)
    # required for the new homebrew version of libwebsockets
    find_path(LIBUV_INCLUDE_DIR NAMES uv.h)
    find_path(LIBEVENT_INCLUDE_DIR NAMES event2/event.h)
    list(APPEND INCLUDE_DIRS ${LIBUV_INCLUDE_DIR} ${LIBEVENT_INCLUDE_DIR})
else()
    list(APPEND LINK_LIBS util)
endif()

if(WIN32)
    list(APPEND LINK_LIBS shell32)
endif()

add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_include_directories(${PROJECT_NAME} PUBLIC ${INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${LINK_LIBS})
target_compile_definitions(${PROJECT_NAME} PRIVATE TTYD_VERSION="${PROJECT_VERSION}")

include(GNUInstallDirs)

install(TARGETS ${PROJECT_NAME} DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT prog)
install(FILES man/ttyd.1 DESTINATION "${CMAKE_INSTALL_MANDIR}/man1" COMPONENT doc)
