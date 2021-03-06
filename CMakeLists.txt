CMAKE_MINIMUM_REQUIRED(VERSION 2.8.4 FATAL_ERROR)
PROJECT(ukf)
INCLUDE(ExternalProject)
INCLUDE(CheckCXXCompilerFlag)

# Check for C++14 support.
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
IF(COMPILER_SUPPORTS_CXX14)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
ELSE()
    MESSAGE(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++14 support. Please use a different C++ compiler.")
ENDIF()

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-register")

# Set default ExternalProject root directory
SET_DIRECTORY_PROPERTIES(PROPERTIES EP_PREFIX .)

# Add eigen3.2.8
ExternalProject_Add(
eigen3
URL https://bitbucket.org/eigen/eigen/get/3.2.8.tar.gz
TIMEOUT 30
# Apply the LLT malloc fix patch
PATCH_COMMAND patch -p1 -t -N < $ENV{UKF_SOURCE_DIR}/patch/eigen-3.2.8-no-malloc-in-llt.patch
# Disable install step
INSTALL_COMMAND ""
# Wrap download, configure and build steps in a script to log output
LOG_DOWNLOAD ON
LOG_CONFIGURE ON
LOG_BUILD ON)

ExternalProject_Get_Property(eigen3 source_dir)
SET(eigen_dir ${source_dir})

INCLUDE_DIRECTORIES(${eigen_dir})

ENABLE_TESTING()

ADD_SUBDIRECTORY(test EXCLUDE_FROM_ALL)
ADD_SUBDIRECTORY(benchmark EXCLUDE_FROM_ALL)
ADD_SUBDIRECTORY(examples EXCLUDE_FROM_ALL)
