cmake_minimum_required(VERSION 3.23)
project(KadNode C)

OPTION(DEBUG "DEBUG" ON)
OPTION(TLS "TLS Support" ON)
OPTION(CMD "CMD Support" ON)
OPTION(DNS "DNS Support" ON)
OPTION(BOB "BOB Support" ON)
OPTION(LPD "LPD Support" ON)
OPTION(ENABLE_FORWARDING "Port forwarding" OFF)

set(CMAKE_C_STANDARD 11)
#ADD_DEFINITIONS(-Os -Wall -Werror -Wmissing-declarations  -Wwrite-strings -pedantic)
ADD_DEFINITIONS(-Os -Wall)

SET(SOURCES src/announces.c
        src/conf.c
        src/kad.c
        src/log.c
        src/main.c
        src/net.c
        src/peerfile.c
        src/searches.c
        src/utils.c
)

IF(DEBUG)
    message(STATUS "Build with DEBUG")
ENDIF()
IF(UNIX)
    SET(SOURCES ${SOURCES} src/unix.c)
    message(STATUS "Build host runs Linux")
ENDIF()
IF(WIN32)
    SET(SOURCES ${SOURCES} src/unix.c src/windows.c)
ENDIF()


IF(CMD)
    SET(SOURCES ${SOURCES} src/ext-cmd.c)
    ADD_DEFINITIONS(-DCMD)
ENDIF()
IF(BOB)
    SET(SOURCES ${SOURCES} src/ext-bob.c)
    ADD_DEFINITIONS(-DBOB)
ENDIF()
IF(DNS)
    SET(SOURCES ${SOURCES} src/ext-dns.c)
    ADD_DEFINITIONS(-DDNS)
ENDIF()
IF(TLS)
    SET(SOURCES ${SOURCES} src/ext-tls-server.c src/ext-tls-client.c)
    ADD_DEFINITIONS(-DTLS)
ENDIF()
IF(LPD)
    SET(SOURCES ${SOURCES} src/ext-lpd.c)
    ADD_DEFINITIONS(-DLPD)
ENDIF()
IF(ENABLE_FORWARDING)
    SET(SOURCES ${SOURCES} src/ext-fwd.c)
    ADD_DEFINITIONS(-DFWD)
ENDIF()

#include_directories(${CMAKE_SOURCE_DIR}/src)
add_executable(KadNode ${SOURCES})
target_link_libraries(KadNode mbedtls mbedx509 mbedcrypto)
#target_include_directories(KadNode PRIVATE ${CMAKE_SOURCE_DIR}/src/*.h)
target_include_directories(KadNode PRIVATE ${CMAKE_SOURCE_DIR}/headers/)
