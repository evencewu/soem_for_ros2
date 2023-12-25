cmake_minimum_required(VERSION 3.5)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules")
project(soem_ros2)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  # Default to installing in SOEM source directory
  set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)

set(SOEM_INCLUDE_INSTALL_DIR include/soem_ros2) # altered to facilitate ROS2 package, nothing else touched !
set(SOEM_LIB_INSTALL_DIR lib)
set(BUILD_TESTS FALSE)

if(WIN32)
  set(OS "win32")
  include_directories(oshw/win32/wpcap/Include)
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    link_directories(oshw/win32/wpcap/Lib/x64)
  elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
    link_directories(oshw/win32/wpcap/Lib)
  endif()
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /D _CRT_SECURE_NO_WARNINGS")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  /WX")
  set(OS_LIBS wpcap.lib Packet.lib Ws2_32.lib Winmm.lib)
elseif(UNIX AND NOT APPLE)
  set(OS "linux")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Werror")
  set(OS_LIBS pthread rt)
elseif(APPLE)
  # This must come *before* linux or MacOSX will identify as Unix.
  set(OS "macosx")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Werror")
  set(OS_LIBS pthread pcap)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "rt-kernel")
  set(OS "rtk")
  message("ARCH is ${ARCH}")
  message("BSP is ${BSP}")
  include_directories(oshw/${OS}/${ARCH})
  file(GLOB OSHW_EXTRA_SOURCES oshw/${OS}/${ARCH}/*.c)
  set(OSHW_SOURCES "${OS_HW_SOURCES} ${OSHW_ARCHSOURCES}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Werror")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-but-set-variable")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-function")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-format")
  set(OS_LIBS "-Wl,--start-group -l${BSP} -l${ARCH} -lkern -ldev -lsio -lblock -lfs -lusb -llwip -leth -li2c -lrtc -lcan -lnand -lspi -lnor -lpwm -ladc -ltrace -lc -lm -Wl,--end-group")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "rtems")
  message("Building for RTEMS")
  set(OS "rtems")
  set(SOEM_LIB_INSTALL_DIR ${LIB_DIR})
  set(BUILD_TESTS FALSE)
endif()

message(STATUS "OS is ${OS}")

file(GLOB SOEM_SOURCES soem/*.c)
file(GLOB OSAL_SOURCES osal/${OS}/*.c)
file(GLOB OSHW_SOURCES oshw/${OS}/*.c)

file(GLOB SOEM_HEADERS soem/*.h)
file(GLOB OSAL_HEADERS osal/osal.h osal/${OS}/*.h)
file(GLOB OSHW_HEADERS oshw/${OS}/*.h)

#include_directories(soem)
#include_directories(osal)
#include_directories(osal/${OS})
#include_directories(oshw/${OS})

add_library(soem_ros2 SHARED
  ${SOEM_HEADERS}
  ${OSAL_HEADERS}
  ${OSHW_HEADERS}
  ${SOEM_SOURCES}
  ${OSAL_SOURCES}
  ${OSHW_SOURCES}
  ${OSHW_EXTRA_SOURCES})

  ament_target_dependencies(soem_ros2
        PUBLIC
        rclcpp
        )
#target_link_libraries(soem_ros2 ${OS_LIBS})

message(STATUS "LIB_DIR: ${SOEM_LIB_INSTALL_DIR}")

#install(TARGETS soem_ros2 DESTINATION ${SOEM_LIB_INSTALL_DIR})
#install(FILES
#  ${SOEM_HEADERS}
#  ${OSAL_HEADERS}
#  ${OSHW_HEADERS}
#  DESTINATION ${SOEM_INCLUDE_INSTALL_DIR})

target_include_directories(soem_ros2 PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/soem>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/osal>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/osal/${OS}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/oshw/${OS}>
  $<INSTALL_INTERFACE:include>
  )

message(STATUS "LIB_DIR: ${SOEM_LIB_INSTALL_DIR}")

ament_export_targets(soem_ros2 HAS_LIBRARY_TARGET)
ament_export_dependencies(
  rclcpp
)


# 注册 导出头文件
install(
  FILES
  ${SOEM_HEADERS}
  ${OSAL_HEADERS}
  ${OSHW_HEADERS}
  DESTINATION include/soem_ros2
)

# 注册 导出库文件
install(
  TARGETS soem_ros2
  EXPORT soem_ros2
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)


if(BUILD_TESTS) 
  add_subdirectory(test/linux/slaveinfo)
  add_subdirectory(test/linux/eepromtool)
  add_subdirectory(test/linux/simple_test)
endif()

ament_package()
