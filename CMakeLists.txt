project(apulse)
cmake_minimum_required (VERSION 2.8.8)
add_definitions(-std=gnu99 -Wall -fPIC -fvisibility=hidden)
add_definitions(-Werror=implicit-function-declaration)

find_package(PkgConfig REQUIRED)

pkg_check_modules(PULSE libpulse REQUIRED)
pkg_check_modules(REQ glib-2.0 alsa REQUIRED)

include_directories(${PULSE_INCLUDE_DIRS})
include_directories(${REQ_INCLUDE_DIRS})

link_directories(${REQ_LIBRARY_DIRS})

add_library(pulse SHARED
    src/apulse-channel-map.c
    src/apulse-context.c
    src/apulse-mainloop.c
    src/apulse-misc.c
    src/apulse-operation.c
    src/apulse-proplist.c
    src/apulse-signal.c
    src/apulse-stream.c
    src/apulse-threaded-mainloop.c
    src/trace.c
    src/util.c
    src/ringbuffer.c
    src/notimplemented.c
)

add_library(pulse-simple SHARED src/apulse-simple.c)
add_library(pulsecommon-5.0 SHARED src/apulse-common.c)

set_target_properties(pulse PROPERTIES VERSION 0)
set_target_properties(pulse-simple PROPERTIES VERSION 0)

set(SYMBOLMAP "-Wl,-version-script=${CMAKE_SOURCE_DIR}/src/symbolmap")

target_link_libraries(pulse ${SYMBOLMAP} ${REQ_LIBRARIES})
target_link_libraries(pulse-simple ${SYMBOLMAP} ${REQ_LIBRARIES})
target_link_libraries(pulsecommon-5.0 ${SYMBOLMAP} ${REQ_LIBRARIES})

add_subdirectory(tests)

set(APULSEPATH ${CMAKE_INSTALL_PREFIX}/lib/apulse)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/apulse.template
               ${CMAKE_CURRENT_BINARY_DIR}/apulse @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/apulse DESTINATION bin
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
install(TARGETS pulse-simple pulsecommon-5.0 pulse DESTINATION ${APULSEPATH})