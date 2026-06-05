cmake_minimum_required(VERSION 3.8)
# Don't define own project -- use parent instead

if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

add_compile_options(-Wall -fPIC)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif(NOT CMAKE_BUILD_TYPE)

# Find packages
find_package(gazebo_dev QUIET)
find_package(gazebo_plugins QUIET)
find_package(gazebo_ros QUIET)
if(NOT gazebo_dev_FOUND OR NOT gazebo_plugins_FOUND OR NOT gazebo_ros_FOUND)
  message(FATAL_ERROR
    "rosflight_viz_gazebo_classic was found and ROS_DISTRO is humble, but Gazebo Classic dependencies were not found. "
    "Install the Humble Gazebo Classic packages, for example.")
endif()

find_package(geometry_msgs REQUIRED)
find_package(std_srvs REQUIRED)

link_directories(${GAZEBO_LIBRARY_DIRS})

# Gazebo plugin needs to be compiled as a shared library
add_library(gazebo_dynamics_plugin SHARED
  ../../src/dynamics_interface.cpp
  src/gazebo_dynamics_plugin.cpp
  src/gazebo_dynamics.cpp)
target_include_directories(gazebo_dynamics_plugin
  PRIVATE
    ../../include
    include)
target_link_libraries(gazebo_dynamics_plugin
  ${rclcpp_INLCUDE_DIRS}
  ${ament_INCLUDE_DIRS}
  #${SDFormat_INCLUDE_DIRS}
  ${GAZEBO_LIBRARIES}
  Eigen3::Eigen)
ament_target_dependencies(gazebo_dynamics_plugin
  rclcpp
  std_srvs
  rosflight_msgs
  geometry_msgs
  gazebo_ros)
ament_export_targets(gazebo_dynamics_pluginTargets HAS_LIBRARY_TARGET)
ament_export_dependencies(rclcpp std_srvs geometry_msgs rosflight_msgs gazebo_ros)
install(
  TARGETS gazebo_dynamics_plugin
  EXPORT gazebo_dynamics_pluginTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include)

# Install 
install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

install(
  DIRECTORY xacro
  DESTINATION share/${PROJECT_NAME}
)

install(
  DIRECTORY gazebo_resource ../common_resource
  DESTINATION share/${PROJECT_NAME}
)
