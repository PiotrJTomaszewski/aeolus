cmake_minimum_required(VERSION 3.20)

set(PREFIX /usr/local)
set(BINDIR ${PREFIX}/bin)
set(LIBDIR ${PREFIX}/lib)

project(
    aeolus
    VERSION 0.10.4
    LANGUAGES CXX)
set(XIFACE_NAME aeolus_x11)
set(TIFACE_NAME aeolus_txt)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_FLAGS_DEBUG_INIT "-Wall -Wextra -g")
set(CMAKE_CXX_FLAGS_RELEASE_INIT "-Wall -Wextra -O2")

add_compile_definitions(${PROJECT_NAME} LIBDIR="${LIBDIR}")
add_compile_definitions(${PROJECT_NAME} VERSION="${PROJECT_VERSION}")
add_compile_definitions(${PROJECT_NAME} _REENTRANT)

include_directories(${PROJECT_NAME} source)

set(AEOLUS_SOURCES
    source/main.cc
    source/audio.cc
    source/model.cc
    source/slave.cc
    source/imidi.cc
    source/addsynth.cc
    source/scales.cc
    source/reverb.cc
    source/asection.cc
    source/division.cc
    source/rankwave.cc
    source/rngen.cc
    source/exp2ap.cc
    source/lfqueue.cc
)

set(XIFACE_SOURCES
    source/styles.cc
    source/mainwin.cc
    source/midiwin.cc
    source/audiowin.cc
    source/instrwin.cc
    source/editwin.cc
    source/midimatrix.cc
    source/multislider.cc
    source/functionwin.cc
    source/xiface.cc
    source/addsynth.cc
)

set(TIFACE_SOURCES
    source/tiface.cc
)

find_package(Threads REQUIRED)
find_package(ALSA REQUIRED)
find_package(X11 REQUIRED COMPONENTS Xft)
find_package(Freetype REQUIRED)
if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    find_library(ZITA_ALSA_PCMI_LIBRARY zita-alsa-pcmi REQUIRED)
    find_path(ZITA_ALSA_PCMI_INCLUDE_DIRS zita-alsa-pcmi.h REQUIRED)
endif()
find_library(CLTHREADS_LIBRARY clthreads REQUIRED)
find_path(CLTHREADS_INCLUDE_DIRS clthreads.h REQUIRED)
find_library(CLXCLIENT_LIBRARY clxclient REQUIRED)
find_path(CLXCLIENT_INCLUDE_DIRS clxclient.h REQUIRED)
find_library(JACK_LIBRARY jack REQUIRED)
find_path(JACK_INCLUDE_DIRS jack/jack.h REQUIRED)
find_library(READLINE_LIBRARY readline REQUIRED)


add_executable(${PROJECT_NAME} ${AEOLUS_SOURCES})
target_include_directories(${PROJECT_NAME} PRIVATE
    ${ZITA_ALSA_PCMI_INCLUDE_DIRS}
    ${CLTHREADS_INCLUDE_DIRS}
    ${JACK_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME} PRIVATE
    Threads::Threads
    ALSA::ALSA
    dl
    rt
    ${ZITA_ALSA_PCMI_LIBRARY}
    ${CLTHREADS_LIBRARY}
    ${JACK_LIBRARY}
)
install(TARGETS ${PROJECT_NAME} DESTINATION ${BINDIR})


add_library(${XIFACE_NAME} SHARED ${XIFACE_SOURCES})
set_target_properties(${XIFACE_NAME} PROPERTIES PREFIX "")
target_include_directories(${XIFACE_NAME} PRIVATE
    ${THREADS_INCLUDE_DIRS}
    ${CLXCLIENT_INCLUDE_DIRS}
)
target_link_libraries(${XIFACE_NAME} PRIVATE
    Threads::Threads
    X11::X11
    Freetype::Freetype
    ${CLTHREADS_LIBRARY}
    ${CLXCLIENT_LIBRARY}
)
install(TARGETS ${XIFACE_NAME} DESTINATION ${LIBDIR})


add_library(${TIFACE_NAME} SHARED ${TIFACE_SOURCES})
set_target_properties(${TIFACE_NAME} PROPERTIES PREFIX "")
target_include_directories(${TIFACE_NAME} PRIVATE
    ${CLTHREADS_INCLUDE_DIRS}
    ${READLINE_INCLUDE_DIRS}
)
target_link_libraries(${TIFACE_NAME} PRIVATE
    Threads::Threads
    ${CLTHREADS_LIBRARY}
    ${READLINE_LIBRARY}
)
install(TARGETS ${TIFACE_NAME} DESTINATION ${LIBDIR})
