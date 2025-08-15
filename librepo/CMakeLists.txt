cmake_minimum_required(VERSION 3.14)

set(MPIWrapper_USE_MPI "" CACHE STRING "Flag to use MPI (ON or OFF)")

# Project name and version
project(MPIWrapper VERSION 1.0.0
	           DESCRIPTION "Wrapper for MPI libraries"
		   LANGUAGES CXX)

# Add the library as a target
add_library(MPIWrapper)

# Specify the C++ standard
target_compile_features(MPIWrapper PUBLIC cxx_std_11)

# Create an alias for MPIWrapper, so that it can be referenced as MPIWrapper::MPIWrapper
# This is helpful to users, as it helps avoid potential namespace conflicts, and it makes it clear that MPIWrapper::MPIWrapper is a library
add_library(MPIWrapper::MPIWrapper ALIAS MPIWrapper)

# Add the library's source files
target_sources(MPIWrapper
	PRIVATE src/mpiwrapper.cpp)

set_target_properties(
	MPIWrapper
	PROPERTIES VERSION ${PROJECT_VERSION}
	           SOVERSION ${PROJECT_VERSION_MAJOR}
#                   CXX_VISIBILITY_PRESET "hidden"
#                   VISIBILITY_INLINES_HIDDEN TRUE
)

# Set the include directories for the library
# Generally, headers in src should only be visible within the library
# Headers meant to be visible to users should be in /include
target_include_directories(
	MPIWrapper
	PRIVATE src
	PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	       $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Generate an export header, and add it to the include directories
#include(GenerateExportHeader)
#generate_export_header(
#	MPIWrapper
#	EXPORT_FILE_NAME export/MPIWrapper/export_mpiwrapper.hpp
#)
#target_include_directories(
#	MPIWrapper
#	PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/export>
#	       $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDE}>
#)

# Find MPI
if ( MPIWrapper_USE_MPI STREQUAL "ON" )
  find_package(MPI REQUIRED)
elseif( MPIWrapper_USE_MPI STREQUAL "OFF" )
else()
  find_package(MPI)
endif()
if( MPI_FOUND )
  target_link_libraries(MPIWrapper PRIVATE MPI::MPI_CXX
)
  target_compile_definitions(MPIWrapper PRIVATE __use_mpi=1)
  set(MPIWrapper_USE_MPI "ON")
else()
  message(WARNING "MPI was not found. Disabling MPI support.")
  set(MPIWrapper_USE_MPI "OFF")
endif()

if (BUILD_SHARED_LIBS)
  set(MPIWrapper_STATIC_BUILD OFF)
else()
  set(MPIWrapper_STATIC_BUILD ON)
endif()
#message(WARNING "VALUE OF STATIC: ${MPIWrapper_STATIC_BUILD}")


##########################################################################
########################## BEGIN INSTALL SECTION #########################
##########################################################################

# Include the GNUInstallDirs module to get the canonical install paths defined
include(GNUInstallDirs)

# Set install locations
install(
	TARGETS MPIWrapper
	EXPORT MPIWrapperTargets
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Explicitly install the header files
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
#install(
#	FILES "${CMAKE_CURRENT_BINARY_DIR}/export/MPIWrapper/export_mpiwrapper.hpp"
#	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/MPIWrapper
#)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
	"MPIWrapperConfigVersion.cmake"
	VERSION ${PROJECT_VERSION}
	COMPATIBILITY SameMajorVersion)

# message(WARNING "DATAROOT: ${CMAKE_INSTALL_DATAROOTDIR}"
configure_package_config_file(
	"${CMAKE_CURRENT_LIST_DIR}/cmake/MPIWrapperConfig.cmake.in"
	"${CMAKE_CURRENT_BINARY_DIR}/MPIWrapperConfig.cmake"
	INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/MPIWrapper
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/MPIWrapperConfig.cmake
	      ${CMAKE_CURRENT_BINARY_DIR}/MPIWrapperConfigVersion.cmake
	DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/MPIWrapper
)
install(
	EXPORT MPIWrapperTargets
	FILE MPIWrapperTargets.cmake
	NAMESPACE MPIWrapper::
	DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/MPIWrapper
)



set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "Organization Name")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "CPack example project")
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${CPACK_PACKAGE_NAME})
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_VERBATIM_VARIABLES YES)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_LIST_DIR}/LICENSE)

set(CPACK_PACKAGE_CONTACT "example@test.org")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "My Name")
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_DEBIAN_PACKAGE_DEPENDS "mpich (>=4.0), libmpich-dev (>=4.0)")
include(CPack)
