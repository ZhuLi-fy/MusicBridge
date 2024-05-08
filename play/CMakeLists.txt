add_library(play STATIC play.cpp)

target_include_directories(play INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})