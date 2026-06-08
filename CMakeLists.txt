cmake_minimum_required(VERSION 3.10)
project(indi_mxhd CXX)

include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(MXHD_DRIVER_VERSION "1.0")

find_package(PkgConfig REQUIRED)

find_package(INDI QUIET)
if(NOT INDI_FOUND)
    pkg_check_modules(INDI_CORE_PKG REQUIRED IMPORTED_TARGET libindi)
endif()

add_executable(indi_mxhd_telescope
    src/mxhd_telescope.cpp
    src/mxhd_telescope.h
    src/mxhd_serial.cpp
    src/mxhd_serial.h
)

if(INDI_FOUND AND TARGET INDI::INDI)
    target_link_libraries(indi_mxhd_telescope PRIVATE INDI::INDI)
elseif(TARGET PkgConfig::INDI_CORE_PKG)
    target_link_libraries(indi_mxhd_telescope PRIVATE PkgConfig::INDI_CORE_PKG)
    find_library(INDI_DRIVER_LIB NAMES indidriver indidriver1 libindidriver REQUIRED)
    target_link_libraries(indi_mxhd_telescope PRIVATE ${INDI_DRIVER_LIB})
else()
    message(FATAL_ERROR "INDI linkage target not found. Install libindi-dev or provide INDIConfig.cmake.")
endif()

find_path(LIBNOVA_INCLUDE_DIR
    NAMES libnova/libnova.h libnova/julian_day.h
    REQUIRED
)
find_library(LIBNOVA_LIBRARY
    NAMES nova libnova libnova-0.16
    REQUIRED
)

target_include_directories(indi_mxhd_telescope PRIVATE ${LIBNOVA_INCLUDE_DIR})
target_link_libraries(indi_mxhd_telescope PRIVATE ${LIBNOVA_LIBRARY})

install(TARGETS indi_mxhd_telescope RUNTIME DESTINATION bin)

set(MXHD_INDI_DATA_DIR "${INDI_DATA_DIR}")
if(NOT MXHD_INDI_DATA_DIR)
    set(MXHD_INDI_DATA_DIR "${CMAKE_INSTALL_DATAROOTDIR}/indi")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/indi_mxhd_telescope.xml.in
               ${CMAKE_CURRENT_BINARY_DIR}/indi_mxhd_telescope.xml @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/indi_mxhd_telescope.xml DESTINATION ${MXHD_INDI_DATA_DIR})
