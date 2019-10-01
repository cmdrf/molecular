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
)
set_target_properties(external_glfw PROPERTIES EXCLUDE_FROM_ALL TRUE)

add_library(glfw STATIC IMPORTED GLOBAL)
set_target_properties(glfw PROPERTIES IMPORTED_LOCATION ${GLFW_LIBRARY})

if(WIN32)
	set_target_properties(glfw PROPERTIES INTERFACE_LINK_LIBRARIES "winmm;ws2_32;dinput8;dxguid;user32;gdi32")
elseif(APPLE)
	find_library(COCOA_LIBRARY Cocoa)
	find_library(IOKIT_LIBRARY IOKit)
	find_library(QUARTZCORE_LIBRARY QuartzCore)
	set_target_properties(glfw PROPERTIES INTERFACE_LINK_LIBRARIES "${COCOA_LIBRARY};${IOKIT_LIBRARY};${QUARTZCORE_LIBRARY}")
else()
	set_target_properties(glfw PROPERTIES INTERFACE_LINK_LIBRARIES "Xrandr;Xi;dl;X11;Xxf86vm;Xinerama;Xcursor")
endif()
set_target_properties(glfw PROPERTIES INTERFACE_INCLUDE_DIRS "${GLFW_INCLUDE_DIR}")

add_dependencies(glfw external_glfw)
