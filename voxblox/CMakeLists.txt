cmake_minimum_required(VERSION 2.8.12)
project(voxblox)

find_package(catkin_simple REQUIRED)
catkin_simple(ALL_DEPS_REQUIRED)

# find_package(protobuf_catkin REQUIRED)
# # 确保 protobuf_catkin 的库路径优先级更高
# link_directories(${protobuf_catkin_LIBRARY_DIRS})

# 强制查找动态库
set(CMAKE_FIND_LIBRARY_SUFFIXES .so)  # 优先动态库
set(PROTOBUF_USE_STATIC_LIBS OFF)    # 明确禁用静态库
# 显式添加动态库路径
# link_directories(/usr/lib/x86_64-linux-gnu)
# link_directories(/opt/ros/melodic/lib/protobuf)
find_package(Protobuf REQUIRED)
message(STATUS "Found a suitable version of system protobuf: ${Protobuf_VERSION}")
message(STATUS "PROTOBUF_LIBRARIES: ${PROTOBUF_LIBRARIES}")
# message(STATUS "protobuf_catkin_INCLUDE_DIRS: ${protobuf_catkin_INCLUDE_DIRS}")
# message(STATUS "protobuf_catkin_LIBRARY_DIRS: ${protobuf_catkin_LIBRARY_DIRS}")
# message(STATUS "protobuf_catkin_LIBRARIES: ${protobuf_catkin_LIBRARIES}")
message(STATUS "===== PROTOBUF_LIBRARIES: ${PROTOBUF_LIBRARIES}")

include_directories(${Protobuf_INCLUDE_DIRS})

set(CMAKE_MACOSX_RPATH 0)
add_definitions(-std=c++11 -Wall -Wextra)

############
# PROTOBUF #
############
# General idea: first check if we have protobuf catkin, then use that.
# Otherwise use system protobuf.
set(PROTO_DEFNS proto/voxblox/Block.proto
                proto/voxblox/Layer.proto)
set(BASE_PATH "proto")
set(PROTOBUF_COMPILE_PYTHON true)

PROTOBUF_CATKIN_GENERATE_CPP2(${BASE_PATH} PROTO_SRCS PROTO_HDRS ${PROTO_DEFNS})

####################
# SET SOURCE FILES #
####################

set("${PROJECT_NAME}_SRCS"
  src/alignment/icp.cc
  src/core/block.cc
  src/core/esdf_map.cc
  src/core/tsdf_map.cc
  src/integrator/esdf_integrator.cc
  src/integrator/esdf_occ_integrator.cc
  src/integrator/integrator_utils.cc
  src/integrator/intensity_integrator.cc
  src/integrator/tsdf_integrator.cc
  src/io/mesh_ply.cc
  src/io/sdf_ply.cc
  src/mesh/marching_cubes.cc
  src/simulation/objects.cc
  src/simulation/simulation_world.cc
  src/utils/camera_model.cc
  src/utils/evaluation_utils.cc
  src/utils/layer_utils.cc
  src/utils/neighbor_tools.cc
  src/utils/protobuf_utils.cc
  src/utils/timing.cc
  src/utils/voxel_utils.cc
)

#############
# LIBRARIES #
#############
cs_add_library(${PROJECT_NAME}_proto
  ${PROTO_SRCS}
)

# 确保链接 protobuf_catkin 或者 系统的 Protobuf 库
target_link_libraries(${PROJECT_NAME}_proto
  /usr/lib/x86_64-linux-gnu/libprotobuf.so
  -lpthread
  # ${PROTOBUF_LIBRARIES}  # 添加 Protobuf 库
)

cs_add_library(${PROJECT_NAME}
  ${${PROJECT_NAME}_SRCS}
)
target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_proto) # 添加 Protobuf 库



############
# BINARIES #
############

add_executable(tsdf_to_esdf
  test/tsdf_to_esdf.cc
)
target_link_libraries(tsdf_to_esdf ${PROJECT_NAME})
# target_link_libraries(tsdf_to_esdf ${PROJECT_NAME} ${PROTOBUF_LIBRARIES})  # 添加 Protobuf 库


add_executable(test_load_esdf
  test/test_load_esdf.cc
)
target_link_libraries(test_load_esdf ${PROJECT_NAME})
# target_link_libraries(test_load_esdf ${PROJECT_NAME} ${PROTOBUF_LIBRARIES})  # 添加 Protobuf 库

#########
# TESTS #
#########
add_custom_target(test_data)
add_custom_command(TARGET test_data
                  COMMAND rm -rf test_data
                  COMMAND mkdir -p test_data
                  COMMAND cp -r ${CMAKE_SOURCE_DIR}/test/test_data/*
                                test_data/ || :)

#add_definitions(-DVISUALIZE_UNIT_TEST_RESULTS)

catkin_add_gtest(test_approx_hash_array
  test/test_approx_hash_array.cc
)
target_link_libraries(test_approx_hash_array ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_map
  test/test_tsdf_map.cc
)
target_link_libraries(test_tsdf_map ${PROJECT_NAME})

catkin_add_gtest(test_protobuf
  test/test_protobuf.cc
)
target_link_libraries(test_protobuf ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_interpolator
  test/test_tsdf_interpolator.cc
)
target_link_libraries(test_tsdf_interpolator ${PROJECT_NAME})

catkin_add_gtest(test_layer
  test/test_layer.cc
)
target_link_libraries(test_layer ${PROJECT_NAME})

catkin_add_gtest(test_merge_integration
  test/test_merge_integration.cc
)
target_link_libraries(test_merge_integration ${PROJECT_NAME})

catkin_add_gtest(test_layer_utils
  test/test_layer_utils.cc
)
target_link_libraries(test_layer_utils ${PROJECT_NAME})

catkin_add_gtest(test_sdf_integrators
  test/test_sdf_integrators.cc
)
target_link_libraries(test_sdf_integrators ${PROJECT_NAME})

catkin_add_gtest(test_bucket_queue
  test/test_bucket_queue.cc
)
target_link_libraries(test_bucket_queue ${PROJECT_NAME})

catkin_add_gtest(test_clear_spheres
  test/test_clear_spheres.cc
)
target_link_libraries(test_clear_spheres ${PROJECT_NAME})

##########
# EXPORT #
##########
cs_install()
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include)
