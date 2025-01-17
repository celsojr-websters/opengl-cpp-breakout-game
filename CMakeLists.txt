cmake_minimum_required(VERSION 3.10)
project(OpenGLProject)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE) # Link time optimization

set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

include(FetchContent)

FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
    GIT_TAG 3.3.8
)
FetchContent_MakeAvailable(glfw)

FetchContent_Declare(
    glm
    GIT_REPOSITORY https://github.com/g-truc/glm.git
    GIT_TAG 0.9.9.8
)
FetchContent_MakeAvailable(glm)

FetchContent_Declare(
    freetype
    GIT_REPOSITORY https://gitlab.com/freetype/freetype.git
    GIT_TAG VER-2-13-0
)
FetchContent_MakeAvailable(freetype)

find_package(OpenGL REQUIRED)

file(GLOB SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB GLAD_FILES ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glad/src/glad.c)

add_executable(${PROJECT_NAME} ${SRC_FILES} ${GLAD_FILES})

if (MSVC)
    # Ensure SIMD optimizations take place
    target_compile_options(${PROJECT_NAME} PRIVATE /arch:AVX2)
    
    # Set subsystem to WINDOWS for a no-console application
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS")
endif()

target_compile_definitions(${PROJECT_NAME} PUBLIC RESOURCES_PATH="${CMAKE_CURRENT_SOURCE_DIR}/resources/")

set(IRRKLANG_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/lib/irrKlang.lib")
target_link_libraries(${PROJECT_NAME}
    PRIVATE
    glfw
    OpenGL::GL
    glm::glm
    freetype
    ${IRRKLANG_LIB_PATH}
)

target_include_directories(${PROJECT_NAME}
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/includes
    ${CMAKE_CURRENT_SOURCE_DIR}/vendor/glad/include
    ${glfw_SOURCE_DIR}/include
    ${glm_SOURCE_DIR}
)

set(DLL_DIR "${CMAKE_CURRENT_SOURCE_DIR}/dlls")
file(GLOB DLL_FILES "${DLL_DIR}/*.dll")

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory "${DLL_DIR}" $<TARGET_FILE_DIR:${PROJECT_NAME}>
)
