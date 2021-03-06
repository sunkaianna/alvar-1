# include opencv library
include_directories(${OpenCV_INCLUDE_DIRS})

# include alvar library
include_directories(${ALVAR_INCLUDE_DIRS})

if(IS_ABSOLUTE "${ALVAR_PLUGIN_PATH}")
  file(TO_NATIVE_PATH "${ALVAR_PLUGIN_PATH}" NATIVE_ALVAR_PLUGIN_PATH)
else(IS_ABSOLUTE "${ALVAR_PLUGIN_PATH}")
  file(TO_NATIVE_PATH "${CMAKE_INSTALL_PREFIX}/${ALVAR_PLUGIN_PATH}" NATIVE_ALVAR_PLUGIN_PATH)
endif(IS_ABSOLUTE "${ALVAR_PLUGIN_PATH}")
message(STATUS "Plugin Path: ${NATIVE_ALVAR_PLUGIN_PATH}")
add_definitions(-DALVAR_PLUGIN_PATH="${NATIVE_ALVAR_PLUGIN_PATH}")

# define alvarplatform library source files to compile
set(ALVARPlatform_HEADERS
    Capture.h
    CaptureDevice.h
    CaptureFactory.h
    CapturePlugin.h
    DirectoryIterator.h
    Lock.h
    Mutex.h
    Platform.h
    Plugin.h
    Threads.h
    Timer.h
    Uncopyable.h
)
set(ALVARPlatform_HEADERS_PRIVATE
    CaptureFactory_private.h
    DirectoryIterator_private.h
    Mutex_private.h
    Plugin_private.h
    Threads_private.h
    Timer_private.h
)
set(ALVARPlatform_SOURCES
    CaptureFactory.cpp
    CaptureDevice.cpp
    DirectoryIterator.cpp
    Mutex.cpp
    Platform.cpp
    Plugin.cpp
    Threads.cpp
    Timer.cpp
)
if(WIN32)
    list(APPEND ALVARPlatform_SOURCES
        CaptureFactory_win.cpp
        DirectoryIterator_win.cpp
        Mutex_win.cpp
        Plugin_win.cpp
        Threads_win.cpp
        Timer_win.cpp
    )
else(WIN32)
    list(APPEND ALVARPlatform_SOURCES
        CaptureFactory_unix.cpp
        DirectoryIterator_unix.cpp
        Mutex_unix.cpp
        Plugin_unix.cpp
        Threads_unix.cpp
        Timer_unix.cpp
    )
endif(WIN32)

# add source files to alvarplatform library and set name and properties
add_library(ALVARPlatform
    ${ALVARPlatform_HEADERS}
    ${ALVARPlatform_HEADERS_PRIVATE}
    ${ALVARPlatform_SOURCES}
)
set_target_properties(ALVARPlatform PROPERTIES
  OUTPUT_NAME alvarplatform
  VERSION ${ALVAR_VERSION}
  SOVERSION ${ALVAR_VERSION_MAJOR}
  DEBUG_POSTFIX d
)

# link alvarplatform library target to required libraries
target_link_libraries(ALVARPlatform
    ALVAR
    ${OpenCV_LIBRARIES}
)

# install headers
if(ALVAR_PACKAGE MATCHES sdk)
    install(FILES ${ALVARPlatform_HEADERS} DESTINATION include/platform)
endif(ALVAR_PACKAGE MATCHES sdk)

# install target
install(TARGETS ALVARPlatform DESTINATION bin)

# set include directories and libraries variables for alvar
list(APPEND ALVAR_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}
)
list(APPEND ALVAR_LIBRARIES
    ALVARPlatform
)
set(_prefix "general")
foreach(_item ${ALVARPlatform_LIB_DEPENDS})
    if(_item MATCHES "^general$|^optimized$|^debug$")
        set(_prefix ${_item})
    else(_item MATCHES "^general$|^optimized$|^debug$")
        set(_found FALSE)
        list(FIND ALVAR_LIBRARIES ${_item} _found)
        if(_found EQUAL -1)
            list(APPEND ALVAR_LIBRARIES ${_prefix} ${_item})
        endif(_found EQUAL -1)
        set(_prefix "general")
    endif(_item MATCHES "^general$|^optimized$|^debug$")
endforeach(_item ${ALVARPlatform_LIB_DEPENDS})
set(ALVAR_INCLUDE_DIRS ${ALVAR_INCLUDE_DIRS} PARENT_SCOPE)
set(ALVAR_LIBRARIES ${ALVAR_LIBRARIES} PARENT_SCOPE)

# hack to setup environment runtime path
if(NOT ${MSVC_IDE})
    list(APPEND ALVAR_RUNTIME
        ${CMAKE_CURRENT_BINARY_DIR}\;
    )
else(NOT ${MSVC_IDE})
    list(APPEND ALVAR_RUNTIME
        ${CMAKE_CURRENT_BINARY_DIR}/Debug\;
        ${CMAKE_CURRENT_BINARY_DIR}/Release\;
    )
endif(NOT ${MSVC_IDE})

# process sub directories
add_subdirectory(capture_plugin_highgui)
add_subdirectory(capture_plugin_file)
if(CMU_FOUND)
    add_subdirectory(capture_plugin_cmu)
endif(CMU_FOUND)
if(ALVAR_USE_FLYCAPTURE2)
    add_subdirectory(capture_plugin_ptgrey)
endif(ALVAR_USE_FLYCAPTURE2)
if(ALVAR_USE_DSCAPTURE)
    add_subdirectory(capture_plugin_dscapture)
endif(ALVAR_USE_DSCAPTURE)

# hack to setup environment runtime path
set(ALVAR_RUNTIME ${ALVAR_RUNTIME} PARENT_SCOPE)
