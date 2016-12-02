cmake_minimum_required(VERSION 2.8.3)
project(exodriver)

set(LIB_VERSION 2.5.3)

find_package(PkgConfig) # try find PKGConfig as it will be used if found
set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)
find_package(LibUSB REQUIRED)

set(HEADERS 
  liblabjackusb/labjackusb.h
  examples/U3/u3.h
  examples/U6/u6.h
  examples/UE9/ue9.h
  examples/Modbus/modbus.h
)

include_directories(
  liblabjackusb 
  examples/U3/
  examples/U6/
  examples/UE9/
  examples/Modbus/
  ${LibUSB_INCLUDE_DIR}
)

SET(EXPORTED_LIBS "")
# library (low-level)
add_library(labjackusb SHARED liblabjackusb/labjackusb.c)
set_target_properties(labjackusb PROPERTIES PUBLIC_HEADER liblabjackusb/labjackusb.h)
target_link_libraries(labjackusb ${LibUSB_LIBRARIES})
LIST (APPEND EXPORTED_LIBS labjackusb)

#  convenience function for U3, U6, UE9 and modbus
add_library(labjacku3 SHARED examples/U3/u3.c)
set_target_properties(labjacku3 PROPERTIES PUBLIC_HEADER examples/U3/u3.h)
target_link_libraries(labjacku3 labjackusb ${LibUSB_LIBRARIES})
LIST (APPEND EXPORTED_LIBS labjacku3)

add_library(labjacku6 SHARED examples/U6/u6.c)
set_target_properties(labjacku6 PROPERTIES PUBLIC_HEADER examples/U6/u6.h)
target_link_libraries(labjacku6 labjackusb ${LibUSB_LIBRARIES})
LIST (APPEND EXPORTED_LIBS labjacku6)

add_library(labjackue9 SHARED examples/UE9/ue9.c)
set_target_properties(labjackue9 PROPERTIES PUBLIC_HEADER examples/UE9/ue9.h)
target_link_libraries(labjackue9 labjackusb ${LibUSB_LIBRARIES})
LIST (APPEND EXPORTED_LIBS labjackue9)

add_library(labjackmodbus SHARED examples/Modbus/modbus.c)
set_target_properties(labjackmodbus PROPERTIES PUBLIC_HEADER examples/Modbus/modbus.h)
target_link_libraries(labjackmodbus labjackusb ${LibUSB_LIBRARIES})
LIST (APPEND EXPORTED_LIBS labjackmodbus)

SET(BINARY_DEMOS "")
## demo executables
SET(U3EXLIST "u3Stream;u3allio;u3BasicConfigU3;u3EFunctions;u3Feedback;u3LJTDAC")
foreach(U3EX ${U3EXLIST})
  add_executable(${U3EX} examples/U3/${U3EX}.c)
  target_link_libraries(${U3EX} labjackusb labjacku3 m ${LibUSB_LIBRARIES})
  LIST (APPEND BINARY_DEMOS ${U3EX})
endforeach()

SET(U6EXLIST "u6Stream;u6allio;u6BasicConfigU6;u6ConfigU6;u6EFunctions;u6Feedback;u6LJTDAC")
foreach(U6EX ${U6EXLIST})
  add_executable(${U6EX} examples/U6/${U6EX}.c)
  target_link_libraries(${U6EX} labjackusb labjacku6 m ${LibUSB_LIBRARIES})
  LIST (APPEND BINARY_DEMOS ${U6EX})
endforeach()

SET(UE9EXLIST "ue9Stream;ue9allio;ue9BasicCommConfig;ue9ControlConfig;ue9EthernetExample;ue9SingleIO;ue9TimerCounter;ue9EFunctions;ue9Feedback;ue9LJTDAC")
foreach(UE9EX ${UE9EXLIST})
  add_executable(${UE9EX} examples/UE9/${UE9EX}.c)
  target_link_libraries(${UE9EX} labjackusb labjackue9 m ${LibUSB_LIBRARIES})
  LIST (APPEND BINARY_DEMOS ${UE9EX})
endforeach()

SET(UMODBUSEXLIST "readModbusExample;testModbusFunctions;writeModbusExample")
foreach(UMODBUSEX ${UMODBUSEXLIST})
  add_executable(${UMODBUSEX} examples/Modbus/${UMODBUSEX}.c)
  target_link_libraries(${UMODBUSEX} labjackusb labjackmodbus m ${LibUSB_LIBRARIES})
  LIST (APPEND BINARY_DEMOS ${UMODBUSEX})
endforeach()

add_executable(u12AISample examples/U12/u12AISample.c)
target_link_libraries(u12AISample labjackusb m ${LibUSB_LIBRARIES})
LIST (APPEND BINARY_DEMOS u12AISample)

install(TARGETS 
  labjackusb
  labjacku3
  labjacku6
  labjackue9
  labjackmodbus
  
  EXPORT ${PROJECT_NAME}Targets
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  # specify where to install our headers
  PUBLIC_HEADER DESTINATION include/${PROJECT_NAME}
  # specify where includes were installed (for package export)
  INCLUDES DESTINATION include
)

install(TARGETS ${BINARY_DEMOS}
  RUNTIME DESTINATION bin
)

## cmake package config
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION "${LIB_VERSION}"
  COMPATIBILITY AnyNewerVersion
  )

export(EXPORT  ${PROJECT_NAME}Targets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake"
  NAMESPACE Upstream::
)

configure_file("${PROJECT_NAME}Config.cmake.in"
               "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
               @ONLY)

# install the export set for use in the install tree
install(EXPORT ${PROJECT_NAME}Targets
  DESTINATION share)

# install the *Config.cmake and *ConfigVersion.cmake
install(FILES
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION share)

## pkgconfig file
configure_file(${PROJECT_NAME}.pc.in ${PROJECT_NAME}.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
DESTINATION lib/pkgconfig)
