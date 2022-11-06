cmake_minimum_required(VERSION 2.8.9)

# Maybe stop from CMAKEing in the wrong place
if (CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
    message(FATAL_ERROR "Source and build directories cannot be the same. Go use the /build directory.")
endif()

file(GLOB IMGUI_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/../deps/polyscope/deps/imgui/imgui/*.h")
set(CEPS_HEADERS )
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Clausen.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/CommonRefinement.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/ConeFlattening.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/ConePlacement.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Cutter.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/DirectSum.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/DirectSum.ipp")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/FlipFormulas.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/GeometryUtils.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/IO.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/IO.ipp")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Layout.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Logger.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Optimizer.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Tracing.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Triangulation.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Utils.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/Utils.ipp")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/polyscope/SurfaceProjectiveParameterizationQuantity.h")
list(APPEND CEPS_HEADERS "${CMAKE_CURRENT_LIST_DIR}/polyscope/SurfaceProjectiveParameterizationQuantity.ipp")
add_library(
  CEPS
  Triangulation.cpp
  Optimizer.cpp
  Tracing.cpp
  FlipFormulas.cpp
  Layout.cpp
  CommonRefinement.cpp
  Utils.cpp
  GeometryUtils.cpp
  Cutter.cpp
  ConePlacement.cpp
  ConeFlattening.cpp
  IO.cpp
  Clausen.cpp
  Logger.cpp
  polyscope/SurfaceProjectiveParameterizationQuantity.cpp
  ../deps/predicates/predicates.c
)
target_link_libraries(CEPS polyscope geometry-central)
set_target_properties(CEPS PROPERTIES PUBLIC_HEADER "${CEPS_HEADERS}")
install(TARGETS CEPS polyscope geometry-central)
install(FILES ${IMGUI_HEADERS} DESTINATION "${CMAKE_INSTALL_PREFIX}/include/")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../deps/polyscope/include/"
  DESTINATION "${CMAKE_INSTALL_PREFIX}/include/"
  FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../deps/polyscope/include/"
  DESTINATION "${CMAKE_INSTALL_PREFIX}/include/"
  FILES_MATCHING PATTERN "*.ipp"
)
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../deps/geometry-central/include/"
  DESTINATION "${CMAKE_INSTALL_PREFIX}/include/"
  FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../deps/geometry-central/include/"
  DESTINATION "${CMAKE_INSTALL_PREFIX}/include/"
  FILES_MATCHING PATTERN "*.ipp"
)

add_executable(
  parameterize
  Parameterize.cpp
)
target_include_directories(parameterize PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}")
# borrow args.hxx directly from polyscope
target_include_directories(parameterize PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../deps/polyscope/deps/args/")
target_link_libraries(parameterize polyscope geometry-central CEPS)

option(SPHERICAL_UNIFORMIZATION "Build spherical uniformizer" OFF)
option(PACKAGE_TESTS "Build the cone flattening tests" OFF)

message("Spherical uniformization: ${SPHERICAL_UNIFORMIZATION}")
message("Tests: ${PACKAGE_TESTS}")

if(SPHERICAL_UNIFORMIZATION)
  # PETSC_DIR=/home/mark/Documents/Code/petsc PETSC_ARCH=arch-linux2-c-opt all
  add_subdirectory(SphericalUniformization)
endif()

if(PACKAGE_TESTS)
  enable_testing()
  add_subdirectory(
    tests
  )
endif()
