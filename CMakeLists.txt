cmake_minimum_required(VERSION 2.4)
project(raspicam)
get_directory_property(HAS_PARENT PARENT_DIRECTORY)

set(SOURCES_LIST
  src/raspicam.cpp
  src/raspicam_still.cpp
  src/private/private_impl.cpp
  src/private/threadcondition.cpp
  src/private_still/private_still_impl.cpp
)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/dependencies
  ${CMAKE_CURRENT_SOURCE_DIR}/dependencies/mmal
  ${CMAKE_CURRENT_SOURCE_DIR}/dependencies/vcos
)

if (NOT($ENV{CROSS_TRIPLE} MATCHES arm*))
  set(SOURCES_LIST ${SOURCES_LIST} src/private/fake_mmal_dependencies.cpp)
endif()

add_library(raspicam STATIC ${hdrs_base} ${SOURCES_LIST})

if ($ENV{CROSS_TRIPLE} MATCHES arm*)
  find_library(mmalcore_LIBS NAMES mmal_core PATHS /opt/vc/lib)
  find_library(mmalutil_LIBS NAMES mmal_util PATHS /opt/vc/lib)
  find_library(mmal_LIBS NAMES mmal PATHS /opt/vc/lib)

  message(STATUS "${mmalcore_INCLUDE_DIRS}")
  if ((NOT mmal_LIBS) OR (NOT mmalutil_LIBS) OR (NOT mmalcore_LIBS))
    message(FATAL_ERROR "Could not find mmal libraries")
  endif()

  target_link_libraries(raspicam ${mmalcore_LIBS} ${mmalutil_LIBS} ${mmal_LIBS})
  include_directories("/opt/vc/include" "/opt/vc/include/interface")
endif()

find_package(OpenCV 3 REQUIRED core highgui)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

include_directories(${OpenCV_INCLUDE_DIRS})
add_library(raspicam_cv STATIC ${SOURCES_LIST} src/raspicam_cv.cpp src/raspicam_still_cv.cpp)
target_link_libraries(raspicam_cv raspicam ${OpenCV_LIBS})

if (HAS_PARENT)
  set(raspicam_cv_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src PARENT_SCOPE)
endif()
