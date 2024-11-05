# Copyright 2019 Xilinx, Inc.
# Copyright 2022 - 2024 Advanced Micro Devices, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

aux_source_directory(. LOG_SRC_LIST)

add_library(${PROJECT_NAME}
  ${PROJECT_SOURCE_DIR}/include/UniLog/UniLog.hpp
  ${PROJECT_SOURCE_DIR}/include/UniLog/UniLogExport.hpp
  ${PROJECT_SOURCE_DIR}/include/UniLog/ErrorCode.hpp
  ${PROJECT_SOURCE_DIR}/include/UniLog/vitis_ai_pp.hpp
  ${LOG_SRC_LIST})
add_library(unilog::unilog ALIAS ${PROJECT_NAME})
set_target_properties(${PROJECT_NAME} PROPERTIES
  VERSION "${PROJECT_VERSION}"
  SOVERSION "${PROJECT_VERSION_MAJOR}")
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
find_package(Boost COMPONENTS system filesystem REQUIRED)
# for MSVC we must add it into LIBPATH even the lib file is full path.
target_link_directories(${PROJECT_NAME} PRIVATE "${Boost_LIBRARY_DIRS}")
target_link_libraries(${PROJECT_NAME}
  PRIVATE Boost::system Boost::filesystem
  PUBLIC glog::glog)
  # Enable Export of public APIs
target_compile_definitions(${PROJECT_NAME} PRIVATE -DUNILOG_EXPORT)
target_compile_definitions(${PROJECT_NAME} PUBLIC $<$<CONFIG:RELEASE>:UNI_LOG_NDEBUG>)
if(BUILD_SHARED_LIBS)
    target_compile_definitions(${PROJECT_NAME} PUBLIC -DUNILOG_USE_DLL=1)
else(BUILD_SHARED_LIBS)
    target_compile_definitions(${PROJECT_NAME} PUBLIC -DUNILOG_USE_DLL=0)
endif(BUILD_SHARED_LIBS)
if(NOT MSVC)
  target_compile_options(${PROJECT_NAME} PUBLIC $<$<CONFIG:DEBUG>:-U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=0>)
endif(NOT MSVC)
if(CMAKE_SOURCE_DIR STREQUAL unilog_SOURCE_DIR)
  install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-targets
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib)
  install(FILES
    ${PROJECT_SOURCE_DIR}/include/UniLog/UniLog.hpp
    ${PROJECT_SOURCE_DIR}/include/UniLog/UniLogExport.hpp
    ${PROJECT_SOURCE_DIR}/include/UniLog/ErrorCode.hpp
    ${PROJECT_SOURCE_DIR}/include/UniLog/vitis_ai_pp.hpp
    DESTINATION include/UniLog)
  install(
    EXPORT ${PROJECT_NAME}-targets
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION share/cmake/${PROJECT_NAME})
endif()
