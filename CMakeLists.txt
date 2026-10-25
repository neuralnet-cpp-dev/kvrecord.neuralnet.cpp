# kvrecord.neuralnet.cpp — 自描述二进制键值记录容器（header-only，轻量 JSON 替代）
#
# 内容：model_keyvalue_record.hpp —— KeyValueRecord（UInt/Str/UIntArray/嵌套 Record，
#       v1 内联 + v2 地址布局，版本自举解析，前向兼容未知类型）。
#
# 依赖：base.neuralnet.cpp（Result<T>），经 extern/base submodule。
#
# 消费方式：
#   git submodule add .../base.neuralnet.cpp.git extern/base
#   add_subdirectory(kvrecord)
#   target_link_libraries(app PRIVATE neuralnet::kvrecord)
#   #include <neuralnet.cpp/model_keyvalue_record.hpp>
cmake_minimum_required(VERSION 3.20)
project(neuralnet_kvrecord LANGUAGES CXX)

if(NOT TARGET neuralnet_base)
    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/extern/base
                     ${CMAKE_CURRENT_BINARY_DIR}/extern/base)
endif()

add_library(neuralnet_kvrecord INTERFACE)
add_library(neuralnet::kvrecord ALIAS neuralnet_kvrecord)
target_include_directories(neuralnet_kvrecord INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
target_link_libraries(neuralnet_kvrecord INTERFACE neuralnet::base)
target_compile_features(neuralnet_kvrecord INTERFACE cxx_std_23)
