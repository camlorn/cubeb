cmake_MINIMUM_REQUIRED(VERSION 2.8)

project(cubeb)

include(CheckIncludeFile)
include(CheckIncludeFiles)
include(FindPkgConfig)

#these defaults are overridden below.
#Unless something toggles it on, they're all off.
SET(CUBEB_ALSA_DEFAULT OFF)
SET(CUBEB_PULSE_DEFAULT OFF)
SET(CUBEB_JACK_DEFAULT OFF)
SET(CUBEB_AUDIOUNIT_DEFAULT OFF)
SET(CUBEB_AUIOQUEUE_DEFAULT OFF)
SET(CUBEB_WASAPI_DEFAULT OFF)
SET(CUBEB_WINMM_DEFAULT OFF)
SET(CUBEB_DIRECTSOUND_DEFAULT OFF)
SET(CUBEB_SNDIO_DEFAULT OFF)
SET(CUBEB_OPENSL_DEFAULT OFF)
SET(CUBEB_AUDIOTRACK_DEFAULT OFF)

#Whether to build static or shared library.
SET(CUBEB_LIBRARY_TYPE STATIC)

if(${WIN32})
set(${CUBEB_WINMM_DEFAULT} ON)
CHECK_INCLUDE_FILE(dsound.h HAVE_DSOUND_H)
if(${HAVE_DSOUND_H})
#Direct Sound doesn't currently even compile, uncomment this later.
#set(CUBEB_DIRECTSOUN_DEFAULT ON)
endif()
CHECK_INCLUDE_FILES("mmdeviceapi.h;audioclient.h" HAVE_WASAPI)
if(${HAVE_WASAPI})
set(CUBEB_WASAPI_DEFAULT ON)
endif()
endif()

if(${PKG_CONFIG_FOUND})
PKG_CHECK_MODULES(PULSE libpulse>=0.9.16)
if(${PULSE_FOUND})
SET(CUBEB_PULSE_DEFAULT ON)
endif()

PKG_CHECK_MODULES(JACK jack)
if(${JACK_FOUND})
set(CUBEB_JACK_DEFAULT ON)
endif()
endif()

#CMake gives us a module for this one.
find_package(ALSA)
if(${ALSA_FOUND})
SET(CUBEB_ALSA_DEFAULT ON)
endif()

option(CUBEB_USE_ALSA "Use Alsa backend" ${CUBEB_ALSA_DEFAULT})
option(CUBEB_USE_PULSE "Use Pulseaudio Backend" ${CUBEB_PULSE_DEFAULT})
option(CUBEB_USE_JACK "Use Jack Backend" ${CUBEB_JACK_DEFAULT})
option(CUBEB_USE_AUDIOUNIT "Use Audio Unit Backend" ${CUBEB_AUDIOUNIT_DEFAULT})
option(CUBEB_USE_AUDIOQUEUE "Use Audioqueue Backend" ${CUBEB_AUDIOQUEUE_DEFAULT})
option(CUBEB_USE_WASAPI "Use Wasapi Backend" ${CUBEB_WASAPI_DEFAULT})
option(CUBEB_USE_WINMM "Use Winmm Backend" ${CUBEB_WINMM_DEFAULT})
option(CUBEB_USE_DIRECTSOUND "Use DirectSound Backend" ${CUBEB_DIRECTSOUND_DEFAULT})
option(CUBEB_USE_SNDIO "Use Sndio Backend" ${CUBEB_SNDIO_DEFAULT})
option(CUBEB_USE_OPENSL "Use OpenSL backend" ${CUBEB_OPENSL_DEFAULT})
OPTION(CUBEB_USE_AUDIOTRACK "Use Audiotrack Backend" ${CUBEB_AUDIOTRACK_DEFAULT})

OPTION(CUBEB_BUILD_SHARED "Enable to build a shared library" ON)
if(${CUBEB_BUILD_SHARED})
SET(CUBEB_LIBRARY_TYPE SHARED)
endif()

#Build the Cubeb sources variable.
SET(CUBEB_SOURCES src/cubeb.c src/speex/resample.c src/cubeb_resampler.cpp)
#Speex:
#warning: RANDOM_PREFIX=cubeb causes issues with cubeb_resampler.cpp.
SET(CUBEB_DEFINES "OUTSIDE_SPEEX;RANDOM_PREFIX=cubeb_speex;FLOATING_POINT;EXPORT=")

#Each option has one source file.
if(${CUBEB_USE_ALSA})
SET(CUBEB_DEFINES"${CUBEB_DEFINES};USE_ALSA")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_alsa.c)
SET(CUBEB_ADDITIONAL_LIBRARIES${CUBEB_ADDITIONAL_LIBRARIES} ${ALSA_LIBRARIES})
endif()
if(${CUBEB_USE_PULSE})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_PULSE")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_pulse.c)
#this is probably not needed on newer machines.
SET(CUBEB_ADDITIONAL_LIBRARIES ${CUBEB_ADDITIONAL_LIBRARIES} ${PULSE_LIBRARIES} dl)
endif()
if(${CUBEB_USE_JACK})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_JACK")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_jack.cpp)
SET(CUBEB_ADDITIONAL_LIBRARIES ${CUBEB_ADDITIONAL_LIBRARIES} ${JACK_LIBRARIES})
endif()
if(${CUBEB_USE_AUDIOUNIT})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_AUDIOUNIT")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_audiounit.c)
endif()
if(${CUBEB_USE_AUDIOQUEUE})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_AUDIOQUEUE")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_audioqueue.c)
endif()
if(${CUBEB_USE_WASAPI})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_WASAPI")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_wasapi.cpp)
SET(CUBEB_ADDITIONAL_LIBRARIES ${CUBEB_ADDITIONAL_LIBRARIES} ole32)
endif()
if(${CUBEB_USE_WINMM})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_WINMM")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_winmm.c)
endif()
if(${CUBEB_USE_DIRECTSOUND})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_DIRECTSOUND")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_directsound.cpp)
endif()
if(${CUBEB_USE_SNDIO})
SET(CUBEB_DEFINES"${CUBEB_DEFINES};USE_SNDIO")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_sndio.c)
endif()
IF(${CUBEB_USE_OPENSL})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_OPENSL")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_opensl.c)
endif()
if(${CUBEB_USE_AUDIOTRACK})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_AUDIOTRACK")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_audiotrack.c)
endif()

find_package(Threads) #magically makes threads work and sets CMAKE_THREAD_LINBS_INIT
if(${CMAKE_COMPILER_IS_GNUCC})
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu99")
endif()

#the Cubeb library.
add_library(cubeb ${CUBEB_LIBRARY_TYPE} ${CUBEB_SOURCES})
#Cubeb-specific preprocessor macros.
set_target_properties(cubeb PROPERTIES COMPILE_DEFINITIONS "${CUBEB_DEFINES}")
if("${CUBEB_ADDITIONAL_LIBRARIES}" OR "${CMAKE_THREAD_LIBS_INIT}")
target_link_libraries(cubeb ${CUBEB_ADDITIONAL_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
endif()

#add its include dir.
#The latter is so that we can make a cubeb-stdint.h, see below.
include_directories("${CMAKE_SOURCE_DIR}/include" "${CMAKE_BINARY_DIR}/include")

#make cubeb-stdint.h.
#TODO: this should do more checking.
file(WRITE "${CMAKE_BINARY_DIR}/include/cubeb/cubeb-stdint.h" "#include <stdint.h>")

#add the tests
add_executable(test_audio test/test_audio.cpp)
target_link_libraries(test_audio cubeb ${CUBEB_ADDITIONAL_LIBRARIES})

add_executable(test_latency test/test_latency.cpp)
target_link_libraries(test_latency cubeb ${CUBEB_ADDITIONAL_LIBRARIES})

add_executable(test_sanity test/test_sanity.cpp)
target_link_libraries(test_sanity cubeb ${CUBEB_ADDITIONAL_LIBRARIES})

add_executable(test_tone test/test_tone.cpp)
target_link_libraries(test_tone cubeb ${CUBEB_ADDITIONAL_LIBRARIES})
