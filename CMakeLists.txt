cmake_minimum_required(VERSION 3.12)
#include(GNUInstallDirs)

set( SIGMA_COMPARE "sigma-compare")

project (${SIGMA_COMPARE} DESCRIPTION "EXR comparison tool designed by Gary Demos" LANGUAGES CXX C)

set(CMAKE_MODULE_PATH
	"${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules"
	${CMAKE_MODULE_PATH}
)

set(CMAKE_CXX_STANDARD 11)

if (MSVC)
  add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
endif()
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
  add_compile_options(
  -fexceptions 
  -Wall 
  -Wextra 
  -Wconversion 
  -Wunused-parameter
  )
endif()

set(CMAKE_CXX_FLAGS_ASAN
  "-fsanitize=address -fno-optimize-sibling-calls -fsanitize-address-use-after-scope -fno-omit-frame-pointer -g -O1"
  CACHE STRING "Flags used by the C++ compiler during AddressSanitizer builds."
  FORCE)

if( NOT DEFINED CMAKE_BUILD_TYPE )
  MESSAGE( STATUS "To use Release mode, use \"cmake .. -DCMAKE_BUILD_TYPE=Release\"" )
  MESSAGE( STATUS "To use AddressSanitizer, use \"cmake .. -DCMAKE_BUILD_TYPE=asan\"" )
elseif( CMAKE_BUILD_TYPE STREQUAL "Release" )
  MESSAGE( STATUS "Using Release mode - CMAKE_BUILD_TYPE : \"${CMAKE_BUILD_TYPE}\"")
elseif( CMAKE_BUILD_TYPE STREQUAL "asan" )
  MESSAGE( STATUS "Using AddressSanitizer - CMAKE_BUILD_TYPE : \"${CMAKE_BUILD_TYPE}\"")
else()
  MESSAGE( STATUS "CMAKE_BUILD_TYPE : \"${CMAKE_BUILD_TYPE}\"")
  MESSAGE( STATUS "To use Release mode, use \"cmake .. -DCMAKE_BUILD_TYPE=Release\"" )
  MESSAGE( STATUS "To use AddressSanitizer, use \"cmake .. -DCMAKE_BUILD_TYPE=asan\"" )
endif()

# Set a default build type if none was specified
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

find_package(OpenEXR 3 CONFIG QUIET)
if(OpenEXR_FOUND)
  message(STATUS "Found OpenEXR ${OpenEXR_VERSION}")
else()
  message(STATUS "did not find OpenEXR 3, looking for OpenEXR 2")
  find_package(IlmBase REQUIRED)
  find_package(OpenEXR 2 REQUIRED)
endif()

file(GLOB SRC_FILES source/*)
add_executable(${SIGMA_COMPARE} ${SRC_FILES})

target_link_libraries( ${SIGMA_COMPARE} PRIVATE  
    # For OpenEXR/Imath 3.x:
    $<$<TARGET_EXISTS:OpenEXR::OpenEXR>:OpenEXR::OpenEXR>
    $<$<TARGET_EXISTS:Imath::Imath>:Imath::Imath>
    $<$<TARGET_EXISTS:Imath::Half>:Imath::Half>
    # For OpenEXR 2.4/2.5:
    $<$<TARGET_EXISTS:OpenEXR::IlmImf>:OpenEXR::IlmImf>
    $<$<TARGET_EXISTS:IlmBase::Imath>:IlmBase::Imath>
    $<$<TARGET_EXISTS:IlmBase::Half>:IlmBase::Half>
    $<$<TARGET_EXISTS:IlmBase::IlmThread>:IlmBase::IlmThread>
    $<$<TARGET_EXISTS:IlmBase::Iex>:IlmBase::Iex>
    # ZLIB for OpenEXR 2.x:
    $<$<TARGET_EXISTS:ZLIB::ZLIB>:ZLIB::ZLIB>
    )

# # add basic tests
enable_testing()

add_test(NAME "none-vs-zip" COMMAND ${SIGMA_COMPARE} "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.NONE.exr" "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.ZIP.exr")
add_test(NAME "none-vs-piz" COMMAND ${SIGMA_COMPARE} "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.NONE.exr" "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.PIZ.exr")
add_test(NAME "DWAA10-vs-none" COMMAND ${SIGMA_COMPARE} "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.DWAA_10.exr" "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.NONE.exr")
add_test(NAME "B44-vs-none" COMMAND ${SIGMA_COMPARE} "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.B44.exr" "${PROJECT_SOURCE_DIR}/resources/test/exr/AllHalfValues.NONE.exr")
