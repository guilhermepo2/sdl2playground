cmake_minimum_required(VERSION 3.10)
project(sdl2playgorund)
add_subdirectory(thirdparty/glad)

add_executable(${PROJECT_NAME}
    src/Main.cpp
)

if(EXISTS SDL2_PATH)
    message("found SDL2_PATH")
    set(SDL2_FOUND true)
else()
    set(SDL2_FOUND false)
endif()

if(NOT ${SDL2_FOUND})
    message("downloading SDL2...")
        
    include(FetchContent)
    FetchContent_Declare(
        SDL2
        URL     https://www.libsdl.org/release/SDL2-devel-2.0.18-VC.zip
        URL_MD5 701b791e8d3f39d018acb2be4be4b0b5
    )
    FetchContent_MakeAvailable(SDL2)
    message("SDL2 SOURCE DIR: " ${sdl2_SOURCE_DIR})
    set(SDL2_PATH ${sdl2_SOURCE_DIR} CACHE INTERNAL "sdl2 root path")
endif()

target_link_libraries(${PROJECT_NAME} SDL2.lib glad)
target_include_directories(${PROJECT_NAME} 
    PUBLIC 
        ${SDL2_PATH}/include
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/thirdparty/glad/include>
    )
target_link_directories(${PROJECT_NAME} PUBLIC ${SDL2_PATH}/lib/x64)

if(EXISTS ${SDL2_PATH}/lib/x64/SDL2.dll)
    message("copying SDL2... " ${SDL2_PATH} "/lib/x64/SDL2/dll")
    add_custom_command(
        TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy ${SDL2_PATH}/lib/x64/SDL2.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>/../
    )
endif()

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/assets)
    add_custom_command(
        TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/assets $<TARGET_FILE_DIR:${PROJECT_NAME}>/../assets
    )
endif()

set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})