cmake_MINIMUM_REQUIRED(VERSION 2.8)

project(cubeb)

include(CheckIncludeFile)
include(CheckIncludeFiles)

#these defaults are overridden below.
#Unless something toggles it on, they're all off.
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

#The following are not platform-specific because it is possible that we may wish to cross-compile in future.

#If we have windows.h, turn on at least MME and do the further windows checks.
CHECK_INCLUDE_FILE(windows.h HAVE_WINDOWS_H)
if(${HAVE_WINDOWS_H})
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

#the Cubeb library.
add_library(cubeb ${CUBEB_LIBRARY_TYPE} ${CUBEB_SOURCES})
#Cubeb-specific preprocessor macros.
set_target_properties(cubeb PROPERTIES COMPILE_DEFINITIONS "${CUBEB_DEFINES}")
if(${CUBEB_ADDITIONAL_LIBRARIES})
target_link_libraries(cubeb ${CUBEB_ADDITIONAL_LIBRARIES})
endif()

#add its include dir.
#The latter is so that we can make a cubeb-stdint.h, see below.
include_directories("${CMAKE_SOURCE_DIR}/include" "${CMAKE_BINARY_DIR}/include")

#make cubeb-stdint.h.
#TODO: this should do more checking.
file(WRITE "${CMAKE_BINARY_DIR}/include/cubeb/cubeb-stdint.h" "#include <stdint.h>")
