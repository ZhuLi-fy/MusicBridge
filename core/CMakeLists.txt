add_library(core STATIC test.cpp)

#包含正在处理的文件夹内容，即指定头文件位置
target_include_directories(core INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})