cmake_minimum_required(VERSION 3.5)
project(bspline_opt)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# uncomment the following section in order to fill in
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(std_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(plan_env REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(NLopt QUIET)
if(NOT NLopt_FOUND)
  find_package(nlopt QUIET)
endif()
if(NOT NLopt_FOUND AND NOT nlopt_FOUND)
  find_path(NLOPT_INCLUDE_DIR nlopt.hpp)
  find_library(NLOPT_CXX_LIBRARY NAMES nlopt_cxx)
  find_library(NLOPT_LIBRARY NAMES nlopt)
  if(NOT NLOPT_INCLUDE_DIR OR (NOT NLOPT_CXX_LIBRARY AND NOT NLOPT_LIBRARY))
    message(FATAL_ERROR
      "NLopt not found. Install development package (e.g. libnlopt-cxx-dev) "
      "or set NLOPT_INCLUDE_DIR/NLOPT_CXX_LIBRARY/NLOPT_LIBRARY manually.")
  endif()
endif()
find_package(fast_planner REQUIRED)

include_directories( 
    include 
    include/plan_env   
    ${Eigen3_INCLUDE_DIRS} 
    ${PCL_INCLUDE_DIRS}
    ${NLopt_INCLUDE_DIRS}
    ${NLOPT_INCLUDE_DIRS}
    ${NLOPT_INCLUDE_DIR}
    ${plan_env_INCLUDE_DIRS}
)

add_library( bspline_opt 
    src/bspline_optimizer.cpp 
    )

ament_target_dependencies(bspline_opt
    rclcpp
    rclpy
    std_msgs
    visualization_msgs
    cv_bridge
    plan_env
    fast_planner
)

target_link_libraries(bspline_opt
    Eigen3::Eigen
)

if(TARGET NLopt::nlopt)
  target_link_libraries(bspline_opt NLopt::nlopt)
elseif(TARGET nlopt::nlopt)
  target_link_libraries(bspline_opt nlopt::nlopt)
else()
  target_link_libraries(bspline_opt ${NLOPT_CXX_LIBRARY} ${NLOPT_LIBRARY})
endif()

install(DIRECTORY include/
  DESTINATION include/)  

install(TARGETS bspline_opt
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

ament_export_libraries(bspline_opt)
ament_export_include_directories(include)
ament_package()
