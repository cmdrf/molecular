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
	GIT_REPOSITORY https://github.com/glfw/glfw.git
	GIT_TAG 34d20b0f03df5926882611041354d558c0b364dd
#	UPDATE_COMMAND ""
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
target_include_directories(glfw INTERFACE "${GLFW_INCLUDE_DIR}")

add_dependencies(glfw external_glfw)
