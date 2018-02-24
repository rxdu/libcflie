cmake_minimum_required(VERSION 2.6)
project(cflie)

## Version
set(${PROJECT_NAME}_VERSION_MAJOR_0)
set(${PROJECT_NAME}_VERSION_MINOR_1)

## Set compiler to use c++ 11 features
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX14)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
elseif(COMPILER_SUPPORTS_CXX11)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
	message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++14 or C++11 support. Please use a different C++ compiler.")
endif()

## Put all binary files into /bin and libraries into /lib
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
set(ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

#include_directories("${PROJECT_SOURCE_DIR}/include")
#link_directories("/usr/lib/x86_64-linux-gnu/")

find_library(USB_LIB usb-1.0)
if(${USB_LIB} STREQUAL "USB_LIB-NOTFOUND")
  message(STATUS "The USB libs appear to not be installed.")
  message(FATAL_ERROR "On Ubuntu, do: sudo apt-get install libusb-1.0-0 libusb-1.0-0-dev")
endif()

### Libraries ###
# ARM needs librt linked in
if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "arm")
  set(DEP_LIBS rt ${USB_LIB})
else()
  set(DEP_LIBS ${USB_LIB})
endif()

# Add Cflie library
set(CFLIE_LIB_SRC
  src/CCrazyRadio.cpp
  src/CCrazyflie.cpp
  src/CCRTPPacket.cpp
  src/CTOC.cpp
)
add_library(${PROJECT_NAME}  STATIC ${CFLIE_LIB_SRC})
target_link_libraries(${PROJECT_NAME} ${DEP_LIBS})
target_include_directories(${PROJECT_NAME}  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  PRIVATE src)

### Examples ###
add_subdirectory(examples)

### Install ###

install(DIRECTORY include/cflie
  DESTINATION include
  FILES_MATCHING
  PATTERN "*.h"
  PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

install(
  TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  COMPONENT library)
