cmake_minimum_required(VERSION 3.10)

project(RayTracingInVulkan)

#set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_DEBUG_POSTFIX d)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)

foreach (OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
    string(TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${PROJECT_BINARY_DIR}/bin)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${PROJECT_BINARY_DIR}/bin)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${PROJECT_BINARY_DIR}/lib)
endforeach()

set (CMAKE_CXX_STANDARD 17)

if (WIN32)
	add_definitions(-DUNICODE -D_UNICODE)
	add_definitions(-DWIN32_LEAN_AND_MEAN)
endif ()

if (MSVC)
	foreach (flag_var CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
		if (${flag_var} MATCHES "/MD")
			string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
		endif()
	endforeach()
	
	add_compile_options("/MP")
endif()

if (UNIX)
	
	add_definitions(-DUNIX)
	add_compile_options("-Wall")
	add_compile_options("-fvisibility=hidden")
	
	find_package(Backtrace REQUIRED)

	string(COMPARE EQUAL ${Backtrace_HEADER} "backtrace.h" BacktraceRequired)

	if (BacktraceRequired)
		add_compile_definitions(BOOST_STACKTRACE_USE_BACKTRACE)
	endif (BacktraceRequired)

endif ()

find_package(Boost REQUIRED COMPONENTS exception program_options) 

if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()

if (WIN32)
    find_package(freetype CONFIG REQUIRED)
    find_package(glfw3 CONFIG REQUIRED)
    find_package(glm CONFIG REQUIRED)
    find_package(imgui CONFIG REQUIRED)
    find_package(tinyobjloader CONFIG REQUIRED)
    find_package(Vulkan REQUIRED)
    find_path(STB_INCLUDE_DIRS "stb.h")
else ()
    #list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    #set(VULKAN_SDK "/home/arwillis/bin/vulkansdk-linux-x86_64-1.2.176.1/1.2.176.1/x86_64")
    #list(APPEND CMAKE_MODULE_PATH "/home/arwillis/bin/vulkansdk-linux-x86_64-1.2.176.1/1.2.176.1/source/Vulkan-ExtensionLayer/cmake/")
    #list(APPEND CMAKE_MODULE_PATH "/home/arwillis/bin/vulkansdk-linux-x86_64-1.2.176.1/1.2.176.1/source/Vulkan-Tools/cmake/")
    #find_package(Vulkan REQUIRED)
    #if (VULKAN_FOUND)
    #    message(STATUS "Vulkan library version ${VK_MAJOR_VERSION}")
    #endif (VULKAN_FOUND)
    set(Vulkan_FOUND ON)
    set(Vulkan_GLSLANG_VALIDATOR "${CMAKE_CURRENT_BINARY_DIR}/bin/glslangValidator")
    #find_package(Freetype CONFIG REQUIRED)
    #set(Vulkan_LIBRARY "vulkan")
    #set(Vulkan_INCLUDE_DIRS "third_party/vulkan/include")
endif (WIN32)

add_definitions(-DIMGUI_DISABLE_OBSOLETE_FUNCTIONS)

IF (NOT Vulkan_FOUND)
    message(FATAL_ERROR "Could not find Vulkan library!")
ELSE()
    message(STATUS ${Vulkan_LIBRARY})
ENDIF()

find_program(Vulkan_GLSLANG_VALIDATOR 
	NAMES glslangValidator 
	HINTS ENV VULKAN_SDK 
	PATH_SUFFIXES bin)
	
if (NOT Vulkan_GLSLANG_VALIDATOR)
    message(FATAL_ERROR "glslangValidator not found!")
endif()

set(MAIN_PROJECT "RayTracer")
# Add third party libraries
add_subdirectory(third_party)
add_subdirectory(assets)
include_directories(    
    /usr/include/freetype2
    third_party/imgui
    third_party/glfw/include
    third_party/glm
    third_party/stb
    third_party/vulkan/include
    #$ENV{VULKAN_SDK}/include/vulkan
)
link_directories(
    ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
    ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}
    $ENV{VULKAN_SDK}/lib
)
set(STB_INCLUDE_DIRS "")
add_subdirectory(src)
set_property (DIRECTORY PROPERTY VS_STARTUP_PROJECT ${MAIN_PROJECT})
