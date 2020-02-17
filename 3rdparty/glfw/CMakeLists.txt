# GLFW

set(INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/install)

set(GLFW_PREFIX ${CMAKE_BINARY_DIR}/glfw)
set(GLFW_LIBRARY ${INSTALL_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}glfw3${CMAKE_STATIC_LIBRARY_SUFFIX})
set(GLFW_INCLUDE_DIR ${INSTALL_PREFIX}/include)

execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory "${GLFW_INCLUDE_DIR}")

if(APPLE AND MOLECULAR_ENABLE_VULKAN)
	set(EXTERNAL_GLFW_OPTIONS
		CMAKE_COMMAND env VULKAN_SDK=${VULKANSDK_DIRECTORY} ${CMAKE_COMMAND}
		DEPENDS vulkan
	)
endif()

ExternalProject_Add(external_glfw
	URL https://github.com/glfw/glfw/archive/3.3.tar.gz
	URL_MD5 5be03812f5d109817e6558c3fab7bbe1
	PREFIX ${GLFW_PREFIX}
	${EXTERNAL_GLFW_OPTIONS}
	CMAKE_ARGS
	    ${EXTERNAL_CMAKE_ARGS}
		-DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PREFIX}
	BUILD_BYPRODUCTS ${GLFW_LIBRARY}
	EXCLUDE_FROM_ALL
)

add_library(glfw-imported STATIC IMPORTED GLOBAL)
set_target_properties(glfw-imported PROPERTIES IMPORTED_LOCATION ${GLFW_LIBRARY})
add_dependencies(glfw-imported external_glfw)

add_library(glfw INTERFACE)
target_link_libraries(glfw INTERFACE glfw-imported)

if(WIN32)
	target_link_libraries(glfw INTERFACE winmm ws2_32 dinput8 dxguid user32 gdi32)
elseif(APPLE)
	find_library(COCOA_LIBRARY Cocoa)
	find_library(IOKIT_LIBRARY IOKit)
	find_library(QUARTZCORE_LIBRARY QuartzCore)
	target_link_libraries(glfw INTERFACE "${COCOA_LIBRARY}" "${IOKIT_LIBRARY}" "${QUARTZCORE_LIBRARY}")
else()
	target_link_libraries(glfw INTERFACE Xrandr Xi dl X11 Xxf86vm Xinerama Xcursor)
endif()
target_include_directories(glfw INTERFACE "${GLFW_INCLUDE_DIR}")

