cmake_minimum_required(VERSION 3.1)
project(QuadriFlow)


option(BUILD_PERFORMANCE_TEST "More subdivisition for performance test" OFF)
option(BUILD_OPENMP "Enable support for OpenMP" OFF)
option(BUILD_LOG "Enable verbose log" OFF)
option(BUILD_GUROBI "Enable GUROBI for comparison ONLY" OFF)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
find_package(TBB REQUIRED)
find_package(Eigen REQUIRED)
find_package(glm REQUIRED)
find_package(Boost COMPONENTS program_options REQUIRED)

if (BUILD_GUROBI)
    find_package(GUROBI REQUIRED)
endif(BUILD_GUROBI)

set(LEMON_3RD_PATH 3rd/lemon-1.3.1)

find_path(
    LEMON_SOURCE_ROOT_DIR
    CMakeLists.txt
    PATHS ${CMAKE_SOURCE_DIR}/${LEMON_3RD_PATH} NO_DEFAULT_PATH
    DOC "Location of LEMON source as a CMAKE subproject"
)

if (EXISTS ${LEMON_SOURCE_ROOT_DIR})
    add_subdirectory(${LEMON_SOURCE_ROOT_DIR})
    set(LEMON_INCLUDE_DIRS ${LEMON_SOURCE_ROOT_DIR} ${PROJECT_BINARY_DIR}/${LEMON_3RD_PATH})
    set(LEMON_LIBRARIES lemon)
    unset(LEMON_ROOT_DIR CACHE)
    unset(LEMON_DIR CACHE)
    unset(LEMON_INCLUDE_DIR CACHE)
    unset(LEMON_LIBRARY CACHE)
else()
    find_package(LEMON QUIET NO_MODULE)
    find_package(LEMON REQUIRED)
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-sign-compare")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")  # enable assert
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g")  # enable assert
set(CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS}")
set(CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_LINKER_FLAGS_DEBUG}")

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp -Wno-int-in-bool-context")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address")
    set(CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS}")
    set(CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_LINKER_FLAGS_DEBUG} -fsanitize=address")
endif()


include_directories(src)
include_directories(3rd/pcg32)
include_directories(3rd/pss)
include_directories(3rd/libigl/include)
include_directories(${Boost_INCLUDE_DIRS})
include_directories(${EIGEN_INCLUDE_DIRS})
include_directories(${GLM_INCLUDE_DIRS})
include_directories(${GLUT_INCLUDE_DIRS})
include_directories(${LEMON_INCLUDE_DIRS})
include_directories(${TBB_INCLUDE_DIRS})
include_directories(${GUROBI_INCLUDE_DIRS})

if (BUILD_PERFORMANCE_TEST)
    add_definitions(-DPERFORMANCE_TEST)
endif(BUILD_PERFORMANCE_TEST)

if (BUILD_OPENMP)
    add_definitions(-DWITH_OMP)
endif(BUILD_OPENMP)

if (BUILD_LOG)
    add_definitions(-DLOG_OUTPUT)
endif(BUILD_LOG)

if (BUILD_GUROBI)
    add_definitions(-DWITH_GUROBI)
endif(BUILD_GUROBI)


set(
    quadriflow_SRC
    src/adjacent-matrix.cpp
    src/adjacent-matrix.hpp
    src/compare-key.hpp
    src/config.hpp
    src/dedge.cpp
    src/dedge.hpp
    src/disajoint-tree.hpp
    src/dset.hpp
    src/field-math.hpp
    src/flow.hpp
    src/hierarchy.cpp
    src/hierarchy.hpp
    src/loader.cpp
    src/loader.hpp
    src/localsat.cpp
    src/localsat.hpp
    src/main.cpp
    src/merge-vertex.cpp
    src/merge-vertex.hpp
    src/optimizer.cpp
    src/optimizer.hpp
    src/parametrizer.cpp
    src/parametrizer-flip.cpp
    src/parametrizer-int.cpp
    src/parametrizer-mesh.cpp
    src/parametrizer-scale.cpp
    src/parametrizer-sing.cpp
    src/parametrizer.hpp
    src/serialize.hpp
    src/subdivide.cpp
    src/subdivide.hpp
)

add_executable(
    quadriflow
    ${quadriflow_SRC}
)

target_link_libraries(
    quadriflow
    ${TBB_LIBRARIES}
    ${Boost_LIBRARIES}
    ${LEMON_LIBRARIES}
    ${GUROBI_LIBRARIES}
)
