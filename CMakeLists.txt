cmake_minimum_required(VERSION 3.12)
project(CGALPlugin VERSION 0.1)

set(PLUGIN_CGAL_SRC_DIR src/CGALPlugin)
set(HEADER_FILES
    ${PLUGIN_CGAL_SRC_DIR}/config.h.in
    ${PLUGIN_CGAL_SRC_DIR}/DecimateMesh.h
    ${PLUGIN_CGAL_SRC_DIR}/DecimateMesh.inl
    ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromPolyhedron.h
    ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromPolyhedron.inl
    ${PLUGIN_CGAL_SRC_DIR}/TriangularConvexHull3D.h
    ${PLUGIN_CGAL_SRC_DIR}/TriangularConvexHull3D.inl
    ${PLUGIN_CGAL_SRC_DIR}/CylinderMesh.h
    ${PLUGIN_CGAL_SRC_DIR}/CylinderMesh.inl
    ${PLUGIN_CGAL_SRC_DIR}/Refine2DMesh.h
    ${PLUGIN_CGAL_SRC_DIR}/Refine2DMesh.inl
    ${PLUGIN_CGAL_SRC_DIR}/PoissonSurfaceReconstruction.h
    ${PLUGIN_CGAL_SRC_DIR}/FrontSurfaceReconstruction.h
    ${PLUGIN_CGAL_SRC_DIR}/UpsamplePointCloud.h
    ${PLUGIN_CGAL_SRC_DIR}/BooleanOperations.h
    )
set(SOURCE_FILES
    ${PLUGIN_CGAL_SRC_DIR}/initCGALPlugin.cpp
    ${PLUGIN_CGAL_SRC_DIR}/DecimateMesh.cpp
    ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromPolyhedron.cpp
    ${PLUGIN_CGAL_SRC_DIR}/TriangularConvexHull3D.cpp
    ${PLUGIN_CGAL_SRC_DIR}/CylinderMesh.cpp
    ${PLUGIN_CGAL_SRC_DIR}/Refine2DMesh.cpp
    ${PLUGIN_CGAL_SRC_DIR}/PoissonSurfaceReconstruction.cpp
    ${PLUGIN_CGAL_SRC_DIR}/FrontSurfaceReconstruction.cpp
    ${PLUGIN_CGAL_SRC_DIR}/UpsamplePointCloud.cpp
    ${PLUGIN_CGAL_SRC_DIR}/BooleanOperations.cpp
    )
set(README_FILES README.md)

# Dependencies
find_package(Sofa.Simulation.Common REQUIRED)
find_package(Sofa.Component.Mass REQUIRED)
find_package(Sofa.Component.MechanicalLoad REQUIRED)
find_package(CGAL REQUIRED)
message(STATUS "CGAL VERSION = ${CGAL_VERSION}")

# Check if image plugin is build, If yes add more files to create mesh on top of ImageContainer
sofa_find_package(image QUIET)
if(image_FOUND)
    find_package(CGAL REQUIRED COMPONENTS ImageIO)
    
    list(APPEND HEADER_FILES ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromImage.h)
    list(APPEND HEADER_FILES ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromImage.inl)
    list(APPEND SOURCE_FILES ${PLUGIN_CGAL_SRC_DIR}/MeshGenerationFromImage.cpp)
endif()

# Create the plugin library.
add_library(${PROJECT_NAME} SHARED ${HEADER_FILES} ${SOURCE_FILES} ${README_FILES})

# Set define dllimport/dllexport mechanism on Windows.
target_compile_definitions(${PROJECT_NAME} PRIVATE "-DSOFA_BUILD_CGALPLUGIN")
if(CGAL_VERSION VERSION_GREATER 4.9) # if CGAL >= 4.10
    target_compile_definitions(${PROJECT_NAME} PUBLIC "-DCGAL_MESH_3_VERBOSE=0")
endif()

# Link the plugin library to its dependencies (other libraries).
target_link_libraries(${PROJECT_NAME} PUBLIC Sofa.Simulation.Common Sofa.Component.Mass Sofa.Component.MechanicalLoad)
target_link_libraries(${PROJECT_NAME} PUBLIC CGAL::CGAL)
if(image_FOUND)
    target_link_libraries(${PROJECT_NAME} PUBLIC image CGAL::CGAL_ImageIO)
endif()

set_property(GLOBAL PROPERTY JOB_POOLS one_jobs=1 two_jobs=2 three_jobs=3 four_jobs=4)
if(CMAKE_GENERATOR STREQUAL "Ninja")
    option(SOFA_CGALPLUGIN_LIMIT_NINJA_JOB_POOL "Limit the number of job pool for ninja to avoid OOM errors" OFF)
    if(UNIX AND NOT APPLE AND SOFA_CGALPLUGIN_LIMIT_NINJA_JOB_POOL)
        set_property(TARGET ${PROJECT_NAME} PROPERTY JOB_POOL_COMPILE one_jobs)
    endif()
endif()



# Install rules for the library and the headers; CMake package configurations files
sofa_create_package_with_targets(
    PACKAGE_NAME ${PROJECT_NAME}
    PACKAGE_VERSION ${PROJECT_VERSION}
    TARGETS ${PROJECT_NAME} AUTO_SET_TARGET_PROPERTIES
    INCLUDE_SOURCE_DIR "src"
    INCLUDE_INSTALL_DIR "CGALPlugin"
    RELOCATABLE "plugins"
    )

# If SOFA_BUILD_TESTS exists and is OFF, then these tests will be auto-disabled
cmake_dependent_option(CGALPLUGIN_BUILD_TESTS "Compile the automatic tests" ON "SOFA_BUILD_TESTS OR NOT DEFINED SOFA_BUILD_TESTS" OFF)
if(CGALPLUGIN_BUILD_TESTS)
    enable_testing()
    add_subdirectory(CGALPlugin_test)
endif()
