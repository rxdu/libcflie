## Config common include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Add executables
add_executable(ex-replugging replugging.cpp)
target_link_libraries(ex-replugging ${PROJECT_NAME})

add_executable(ex-simple simple.cpp)
target_link_libraries(ex-simple ${PROJECT_NAME})