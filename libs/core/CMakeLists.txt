set(MODULE_NAME core)

include(${PROJECT_SOURCE_DIR}/cmake/Util.cmake)

file(GLOB_RECURSE SRC_FILES "include/*.*" "src/*.*")
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${SRC_FILES})

add_library(${MODULE_NAME} ${SRC_FILES})

if(USE_SDL_ENGINE)
	find_package(SDL2 CONFIG REQUIRED)
	find_package(sdl2-net CONFIG REQUIRED)
endif()
if(DEBUG_UI)
	find_package(imgui CONFIG REQUIRED)
endif()
find_package(STB MODULE REQUIRED)
if(LINUX)
	find_package(GTK2 2.4 REQUIRED)
	target_include_directories(${MODULE_NAME} PRIVATE ${GTK2_INCLUDE_DIRS})
	target_link_libraries(${MODULE_NAME} PRIVATE ${GTK2_LIBRARIES})
endif()

# Make a wrapper module for STB
add_library(STB INTERFACE)
target_include_directories(STB INTERFACE ${STB_INCLUDE_PATH})

target_include_directories(${MODULE_NAME}
	PUBLIC "include"
)

target_link_libraries(${MODULE_NAME}
	PUBLIC
		$<$<BOOL:${DEBUG_UI}>:imgui::imgui>
	PRIVATE
		STB
		noc
		$<$<BOOL:${LINUX}>:linenoise>
		$<$<BOOL:${USE_SDL_ENGINE}>:SDL2::SDL2-static>
		$<$<BOOL:${USE_SDL_ENGINE}>:SDL2::SDL2_net>
)

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	if("x${CMAKE_CXX_SIMULATE_ID}" STREQUAL "xMSVC") # clang-cl
		# ...
	else()
		target_link_libraries(${MODULE_NAME} PUBLIC
			stdc++fs # required for filesystem
		) 
	endif()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
		target_link_libraries(${MODULE_NAME} PUBLIC
			stdc++fs # required for filesystem
		) 
endif()
