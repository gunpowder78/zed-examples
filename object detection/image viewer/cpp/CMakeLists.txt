cmake_minimum_required(VERSION 2.4)
PROJECT(ZED_Object_detection_image_viewer)

option(LINK_SHARED_ZED "Link with the ZED SDK shared executable" ON)

if (NOT LINK_SHARED_ZED AND MSVC)
    message(FATAL_ERROR "LINK_SHARED_ZED OFF : ZED SDK static libraries not available on Windows")
endif()

if(COMMAND cmake_policy)
        cmake_policy(SET CMP0003 OLD)
        cmake_policy(SET CMP0004 OLD)
        cmake_policy(SET CMP0015 OLD)
endif(COMMAND cmake_policy)

SET(EXECUTABLE_OUTPUT_PATH ".")
SET(SPECIAL_OS_LIBS "")

find_package(ZED 3 REQUIRED)
find_package(GLUT REQUIRED)
find_package(GLEW REQUIRED)
find_package(OpenGL REQUIRED)
find_package(CUDA REQUIRED)

IF(NOT WIN32)
    SET(SPECIAL_OS_LIBS "pthread" "X11")
    add_definitions(-Wno-write-strings)
ENDIF()

include_directories(${ZED_INCLUDE_DIRS})
include_directories(${GLEW_INCLUDE_DIRS})
include_directories(${GLUT_INCLUDE_DIR})
include_directories(${CUDA_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

link_directories(${ZED_LIBRARY_DIR})
link_directories(${GLEW_LIBRARY_DIRS})
link_directories(${GLUT_LIBRARY_DIRS})
link_directories(${OpenGL_LIBRARY_DIRS})
link_directories(${CUDA_LIBRARY_DIRS})

FILE(GLOB_RECURSE SRC_FILES src/*.c*)
FILE(GLOB_RECURSE HDR_FILES include/*.h*)

add_executable(${PROJECT_NAME} ${HDR_FILES} ${SRC_FILES})
add_definitions(-std=c++14)

## DEBUG/ SANITIZER options
IF(NOT WIN32)
    add_definitions(-Werror=return-type)
    IF (BUILD_WITH_SANITIZER)
    message("!! Building with address sanitizer and -g !!")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer -fsanitize=address -Wall -Wextra -g")
    set (CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=address -Wall -Wextra -g")
    add_definitions(-g -fsanitize=address -fno-omit-frame-pointer -static-libasan -Wall -Wextra)
    ENDIF()

    IF (BUILD_WITH_DEBUGINFOS)
    message("!! Building with -g !!")
    add_definitions(-g)
    ENDIF()
ENDIF()


if (LINK_SHARED_ZED)
    SET(ZED_LIBS ${ZED_LIBRARIES} ${CUDA_CUDA_LIBRARY} ${CUDA_CUDART_LIBRARY} ${CUDA_DEP_LIBRARIES_ZED})
else()
    SET(ZED_LIBS ${ZED_STATIC_LIBRARIES} ${CUDA_CUDA_LIBRARY} ${CUDA_LIBRARY})
endif()

target_link_libraries(${PROJECT_NAME}
                        ${SPECIAL_OS_LIBS}
                        ${ZED_LIBS}
                        ${OPENGL_LIBRARIES}
                        ${GLUT_LIBRARY}
                        ${GLEW_LIBRARIES})

if(INSTALL_SAMPLES)
    LIST(APPEND SAMPLE_LIST ${PROJECT_NAME})
    SET(SAMPLE_LIST "${SAMPLE_LIST}" PARENT_SCOPE)
endif()
