cmake_minimum_required(VERSION 3.8)
project(lio_sam)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# ========================
# Dependencies
# ========================
find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)

find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)

find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(pcl_msgs REQUIRED)

find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_eigen REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)

find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)

find_package(PCL REQUIRED)
find_package(pcl_conversions REQUIRED)

find_package(GTSAM REQUIRED)
find_package(Eigen3 REQUIRED)

# ========================
# Include directories
# ========================
include_directories(
  include
  ${PCL_INCLUDE_DIRS}
)

# ========================
# Generate messages
# ========================
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/CloudInfo.msg"
  "srv/SaveMap.srv"
  DEPENDENCIES std_msgs sensor_msgs
)

# VERY IMPORTANT
ament_export_dependencies(rosidl_default_runtime)

# ========================
# Common dependencies
# ========================
set(common_deps
  rclcpp
  rclpy
  std_msgs
  sensor_msgs
  geometry_msgs
  nav_msgs
  visualization_msgs
  pcl_msgs
  pcl_conversions
  tf2
  tf2_ros
  tf2_eigen
  tf2_sensor_msgs
  tf2_geometry_msgs
  cv_bridge
)

# ========================
# Helper macro to link msg
# ========================
function(add_lio_node target_name source_file)
  add_executable(${target_name} ${source_file})

  ament_target_dependencies(${target_name}
    ${common_deps}
    OpenCV
    PCL
  )

  target_link_libraries(${target_name}
    ${PCL_LIBRARIES}
    gtsam
  )

  # Link generated message typesupport
  rosidl_get_typesupport_target(cpp_typesupport_target
    ${PROJECT_NAME} "rosidl_typesupport_cpp")

  target_link_libraries(${target_name}
    ${cpp_typesupport_target}
  )

  install(TARGETS ${target_name}
    DESTINATION lib/${PROJECT_NAME}
  )
endfunction()

# ========================
# Nodes
# ========================
add_lio_node(${PROJECT_NAME}_featureExtraction src/featureExtraction.cpp)
add_lio_node(${PROJECT_NAME}_imageProjection src/imageProjection.cpp)
add_lio_node(${PROJECT_NAME}_imuPreintegration src/imuPreintegration.cpp)
add_lio_node(${PROJECT_NAME}_mapOptimization src/mapOptmization.cpp)
add_lio_node(${PROJECT_NAME}_simpleGpsOdom src/simpleGpsOdom.cpp)

# ========================
# Install folders
# ========================
install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

install(DIRECTORY config
  DESTINATION share/${PROJECT_NAME}
)

install(DIRECTORY include/
  DESTINATION include
)

ament_export_include_directories(include)

# ========================
# Testing
# ========================
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
