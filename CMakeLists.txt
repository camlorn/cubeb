cmake_MINIMUM_REQUIRED(VERSION 2.8)

project(cubeb)

#backend options for Cubeb.
#These are Linux, and usually don't apply on windows.
option(CUBEB_USE_PULSE "Use Pulseaudio Backend" OFF)
option(CUBEB_USE_JACK "Use Jack Backend" OFF)

#OSX and iOS
option(CUBEB_USE_AUDIOUNIT "Use Audio Unit Backend" OFF)
option(CUBEB_USE_AUDIOQUEUE "Use Audioqueue Backend" OFF)

#Windows.
option(CUBEB_USE_WASAPI "Use Wasapi Backend" ON)
option(CUBEB_USE_WINMM "Use Winmm Backend" ON)
option(CUBEB_USE_DIRECTSOUND "Use DirectSound Backend" ON)

#BSD.
option(CUBEB_USE_SNDIO "Use Sndio Backend" OFF)

#Mobile Platforms.
#The latter is only android, but the former can technically be other platforms too.
#I am aware of no other platforms that implement it.
option(CUBEB_USE_OPENSL "Use OpenSL backend" OFF)
OPTION(CUBEB_USE_AUDIOTRACK "Use Audiotrack Backend" OFF)

#Build the Cubeb sources variable.
SET(CUBEB_SOURCES src/cubeb.c src/speex/resample.c)

#Each option has one source file.
if(${CUBEB_USE_PULSE})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_PULSE")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_pulse.c)
endif()
if(${CUBEB_USE_JACK})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_JACK")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_jack.cpp)
endif()
if(${CUBEB_USE_ALSA})
SET(CUBEB_DEFINES "${CUBEB_DEFINES};USE_ALSA")
SET(CUBEB_SOURCES ${CUBEB_SOURCES} src/cubeb_alsa.c)
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

#the Cubeb library.
add_library(cubeb ${CUBEB_SOURCES})
#Cubeb-specific preprocessor macros.
set_target_properties(cubeb PROPERTIES COMPILE_DEFINITIONS "${CUBEB_DEFINES}")

#add its include dir.
#The latter is so that we can make a cubeb-stdint.h, see below.
include_directories("${CMAKE_SOURCE_DIR}/include" "${CMAKE_BINARY_DIR}/include")

#make cubeb-stdint.h.
#TODO: this should do more checking.
file(WRITE "${CMAKE_BINARY_DIR}/include/cubeb/cubeb-stdint.h" "#include <stdint.h>")
