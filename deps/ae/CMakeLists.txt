cmake_minimum_required(VERSION 2.8)

set(HEADERS
        ae.h
        anet.h
        zmalloc.h)
set(SOURCES
        ae.c
        anet.c
        zmalloc.c)

add_library(ae STATIC ${SOURCES} ${HEADERS})
set_property(TARGET ae PROPERTY C_STANDARD 99)
target_include_directories(ae PUBLIC .)