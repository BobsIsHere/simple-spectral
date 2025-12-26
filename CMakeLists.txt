cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(simple-spectral)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
set(CMAKE_CONFIGURATION_TYPES "Debug;RelWithDebInfo" CACHE STRING "" FORCE )

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
#set(CMAKE_SUPPRESS_REGENERATION true)

set_property( DIRECTORY PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME} )

option(SUPPORT_WINDOWED "Support a windowed mode to show progress (req. GLFW)" ON)

include(FetchContent)

FetchContent_Declare(
    glm
    GIT_REPOSITORY https://github.com/g-truc/glm.git
    GIT_TAG 1.0.2
	GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(glm)

if (UNIX)
	set(EXTERNAL_LIBRARIES ${EXTERNAL_LIBRARIES} pthread)
endif()

if(SUPPORT_WINDOWED)
    add_definitions("-DSUPPORT_WINDOWED")

    find_package(OpenGL REQUIRED)
    set(EXTERNAL_LIBRARIES ${EXTERNAL_LIBRARIES} ${OPENGL_LIBRARIES})

    include(FetchContent)
    FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw.git
        GIT_TAG        3.4
        GIT_SHALLOW TRUE
    )
    FetchContent_MakeAvailable(glfw)

    message(STATUS "GLFW headers: ${glfw_SOURCE_DIR}/include")
    include_directories(${glfw_SOURCE_DIR}/include)
    set(EXTERNAL_LIBRARIES ${EXTERNAL_LIBRARIES} glfw)

    set(WINDOW_ARG " --window")
else()
    set(WINDOW_ARG "")
endif()
#message(STATUS "${WINDOW_ARG}")

#No, Microsoft, the standard library is *not* deprecated.
add_definitions("-D_CRT_SECURE_NO_WARNINGS")

file(GLOB_RECURSE SOURCE_FILES
	${CMAKE_SOURCE_DIR}/src/*.hpp*
	${CMAKE_SOURCE_DIR}/src/*.h*
	${CMAKE_SOURCE_DIR}/src/*.cpp*
	${CMAKE_SOURCE_DIR}/src/*.c*
)

add_executable(${PROJECT_NAME} ${SOURCE_FILES})

foreach(_source_file IN ITEMS ${SOURCE_FILES})
	get_filename_component(_source_path "${_source_file}" PATH)
	string(REPLACE "${CMAKE_SOURCE_DIR}" "" _group_path "${_source_path}")
	string(REPLACE "/" "\\" _group_path "${_group_path}")
	source_group("${_group_path}" FILES "${_source_file}")
endforeach()

#set_property( TARGET ${PROJECT_NAME} PROPERTY VS_DEBUGGER_COMMAND "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}${PROJECT_NAME}" )
set_property( TARGET ${PROJECT_NAME} PROPERTY VS_DEBUGGER_COMMAND_ARGUMENTS
	"--scene=cornell-srgb -w=512 -h=512 -spp=64 --output=output.png${WINDOW_ARG}"
)
set_property( TARGET ${PROJECT_NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY
	"${CMAKE_SOURCE_DIR}"
)

target_link_libraries(${PROJECT_NAME} PRIVATE 
	glm::glm
	${EXTERNAL_LIBRARIES})