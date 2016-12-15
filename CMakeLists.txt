cmake_minimum_required(VERSION 2.8.3)
project(exodriver)

set(PROJECT_VERSION 2.5.3)

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

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
target_include_directories(labjackusb PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/liblabjackusb>
	$<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)
# specify which header files to install
set_target_properties(labjackusb PROPERTIES PUBLIC_HEADER liblabjackusb/labjackusb.h)
target_link_libraries(labjackusb PUBLIC ${LibUSB_LIBRARIES})

#  convenience function for U3, U6, UE9 and modbus
foreach (sub u3 u6 ue9 modbus)
	string(TOUPPER ${sub} SUB)
	if(sub STREQUAL "modbus")
		set(SUB "Modbus") # Argh, here it's not simply uppercase
	endif()
	add_library(labjack${sub} SHARED examples/${SUB}/${sub}.c)
	target_include_directories(labjackusb PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/examples/${SUB}>
		$<INSTALL_INTERFACE:include/${PROJECT_NAME}>
	)
	set_target_properties(labjack${sub} PROPERTIES PUBLIC_HEADER examples/${SUB}/${sub}.h)
	target_link_libraries(labjack${sub} PUBLIC labjackusb)
endforeach()

set(BINARY_DEMOS "")
## demo executables
set(U3EXLIST "u3Stream;u3allio;u3BasicConfigU3;u3EFunctions;u3Feedback;u3LJTDAC")
foreach(U3EX ${U3EXLIST})
  add_executable(${U3EX} examples/U3/${U3EX}.c)
  target_link_libraries(${U3EX} labjacku3 m)
  list(APPEND BINARY_DEMOS ${U3EX})
endforeach()

set(U6EXLIST "u6Stream;u6allio;u6BasicConfigU6;u6ConfigU6;u6EFunctions;u6Feedback;u6LJTDAC")
foreach(U6EX ${U6EXLIST})
  add_executable(${U6EX} examples/U6/${U6EX}.c)
  target_link_libraries(${U6EX} labjacku6 m)
  list(APPEND BINARY_DEMOS ${U6EX})
endforeach()

set(UE9EXLIST "ue9Stream;ue9allio;ue9BasicCommConfig;ue9ControlConfig;ue9EthernetExample;ue9SingleIO;ue9TimerCounter;ue9EFunctions;ue9Feedback;ue9LJTDAC")
foreach(UE9EX ${UE9EXLIST})
  add_executable(${UE9EX} examples/UE9/${UE9EX}.c)
  target_link_libraries(${UE9EX} labjackue9 m)
  list(APPEND BINARY_DEMOS ${UE9EX})
endforeach()

set(UMODBUSEXLIST "readModbusExample;testModbusFunctions;writeModbusExample")
foreach(UMODBUSEX ${UMODBUSEXLIST})
  add_executable(${UMODBUSEX} examples/Modbus/${UMODBUSEX}.c)
  target_link_libraries(${UMODBUSEX} labjackmodbus m)
  list(APPEND BINARY_DEMOS ${UMODBUSEX})
endforeach()

add_executable(u12AISample examples/U12/u12AISample.c)
target_link_libraries(u12AISample labjackusb m)
list(APPEND BINARY_DEMOS u12AISample)

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
)

# install the export set for use in the install tree
install(EXPORT ${PROJECT_NAME}Targets DESTINATION share)

install(TARGETS ${BINARY_DEMOS}
  RUNTIME DESTINATION bin
)

## cmake package config
include(CMakePackageConfigHelpers)

configure_file("${PROJECT_NAME}Config.cmake.in"
               "${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
               @ONLY)

write_basic_package_version_file(
  "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION "${PROJECT_VERSION}"
  COMPATIBILITY AnyNewerVersion
)

# install the *Config.cmake and *ConfigVersion.cmake
install(FILES
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION share)

## pkgconfig file
configure_file(${PROJECT_NAME}.pc.in ${PROJECT_NAME}.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
DESTINATION lib/pkgconfig)
