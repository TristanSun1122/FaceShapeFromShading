cmake_minimum_required(VERSION 2.8.11)

project(FaceShapeFromShading)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX14)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
        message("Using c++11")
elseif(COMPILER_SUPPORTS_CXX11)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
        message("Using c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
        message("Using c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Boost
find_package(Boost COMPONENTS filesystem timer program_options REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
link_libraries(${Boost_LIBRARIES} -lboost_filesystem -lboost_system)

# OpenMP
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")

find_package( OpenCV REQUIRED )
link_libraries(${OpenCV_LIBS})

# OpenGL
find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)
include_directories( ${OPENGL_INCLUDE_DIRS}  ${GLUT_INCLUDE_DIRS} )
link_libraries(${OPENGL_LIBRARIES} ${GLUT_LIBRARY})

find_package(GLEW REQUIRED)
if (GLEW_FOUND)
    include_directories(${GLEW_INCLUDE_DIRS})
    link_libraries(${GLEW_LIBRARIES})
endif()

# GLM
find_package(glm REQUIRED)
if (GLM_FOUND)
    message([] ${GLM_INCLUDE_DIRS})
    include_directories(${GLM_INCLUDE_DIRS})
endif()

# GLI
find_package(gli REQUIRED)
if (GLI_FOUND)
    message([] ${GLI_INCLUDE_DIRS})
    include_directories(${GLI_INCLUDE_DIRS})
endif()

# SuiteSparse
set(SUITESPARSE_INCLUDE_DIR /usr/include/suitesparse)
include_directories(${SUITESPARSE_INCLUDE_DIR})

# Eigen
find_package(Eigen)
include_directories(${EIGEN_INCLUDE_DIR})

# MKL
set(MKL_INCLUDE_DIRS "/opt/intel/mkl/include")
message([] ${MKL_INCLUDE_DIRS} )
include_directories(${MKL_INCLUDE_DIRS})
set(MKLROOT "/opt/intel/mkl")
set(MKLLIBS_DIRS "${MKLROOT}/lib/intel64/" "/opt/intel/lib/intel64_lin")
link_directories(${MKLLIBS_DIRS})
set(MKLLIBS "-Wl,--start-group -lmkl_intel_lp64 -lmkl_core -lmkl_intel_thread -Wl,--end-group -liomp5 -ldl -lpthread -lm")

# PhGLib
include_directories("$ENV{HOME}/SDKs/PhGLib/include")
link_directories("$ENV{HOME}/SDKs/PhGLib/lib")
set(PhGLib "-lPhGLib")

# Qt5
find_package(Qt5Core)
find_package(Qt5Widgets)
find_package(Qt5OpenGL)

# Ceres solver
find_package(Ceres REQUIRED)
include_directories(${CERES_INCLUDE_DIRS})
link_libraries(${CERES_LIBRARIES})

# embree
include_directories("/usr/local/include")
link_libraries("/usr/local/lib/libembree.so.2.13.0")

# json
message(STATUS "Found json cpp")
message(STATUS "${CMAKE_CURRENT_LIST_DIR}/json/include")
include_directories("${CMAKE_CURRENT_LIST_DIR}/json/include")

# Face shape from shading program
add_executable(FaceShapeFromShading faceshapefromshading.cpp common.h MultilinearReconstruction/OffscreenMeshVisualizer.cpp MultilinearReconstruction/OffscreenMeshVisualizer.h utils.h)
target_link_libraries(FaceShapeFromShading
                      multilinearmodel
                      basicmesh
                      tensor
                      ioutilities
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      ${MKLLIBS}
                      ${PhGLib})

# Face shape from shading using blendshapes program
add_executable(FaceShapeFromShading_exp faceshapefromshading_exp.cpp common.h MultilinearReconstruction/OffscreenMeshVisualizer.cpp MultilinearReconstruction/OffscreenMeshVisualizer.h utils.h)
target_link_libraries(FaceShapeFromShading_exp
                      multilinearmodel
                      basicmesh
                      tensor
                      ioutilities
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      ${MKLLIBS}
                      ${PhGLib})

add_executable(refine_mesh_with_normal refine_mesh_with_normal.cpp common.h MultilinearReconstruction/OffscreenMeshVisualizer.cpp MultilinearReconstruction/OffscreenMeshVisualizer.h)
target_link_libraries(refine_mesh_with_normal
                      multilinearmodel
                      basicmesh
                      tensor
                      ioutilities
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      ${MKLLIBS}
                      ${PhGLib})

add_executable(refine_mesh_with_normal_exp refine_mesh_with_normal_exp.cpp common.h MultilinearReconstruction/OffscreenMeshVisualizer.cpp MultilinearReconstruction/OffscreenMeshVisualizer.h)
target_link_libraries(refine_mesh_with_normal_exp
                      multilinearmodel
                      basicmesh
                      tensor
                      ioutilities
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      ${MKLLIBS}
                      ${PhGLib})
add_executable(ptrender ptrender.cpp)

add_executable(explicit explicit.cpp)
target_link_libraries(explicit CGAL basicmesh ioutilities ${PhGLib})

add_executable(ambient_occlusion ambient_occlusion.cpp)
target_link_libraries(ambient_occlusion
                      basicmesh
                      offscreenmeshvisualizer
                      ioutilities
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      ${PhGLib})

link_directories(MultilinearReconstruction)

add_subdirectory(MultilinearReconstruction)
add_subdirectory(tests)
