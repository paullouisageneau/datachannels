cmake_minimum_required(VERSION 3.7)
project(datachannel-wasm
	DESCRIPTION "C++ Wasm and Native Data Channels"
	VERSION 0.1.0
	LANGUAGES CXX)

if(CMAKE_SYSTEM_NAME MATCHES "Emscripten")
	set(WASM_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/wasm/src)
    set(DATACHANNELS_SRC
		${WASM_SRC_DIR}/channel.cpp
		${WASM_SRC_DIR}/datachannel.cpp
		${WASM_SRC_DIR}/peerconnection.cpp
		${WASM_SRC_DIR}/websocket.cpp)

	add_library(datachannel-wasm STATIC ${DATACHANNELS_SRC})
	set_target_properties(datachannel-wasm PROPERTIES
		VERSION ${PROJECT_VERSION}
		CXX_STANDARD 17)

	target_include_directories(datachannel-wasm PUBLIC
		${CMAKE_CURRENT_SOURCE_DIR}/wasm/include)
	target_include_directories(datachannel-wasm PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}/wasm/include/rtc)

	set(WASM_OPTS
		"SHELL:-s WASM=1"
		"SHELL:-s BINARYEN_METHOD=native-wasm")

	target_compile_options(datachannel-wasm PUBLIC ${WASM_OPTS})
	target_link_options(datachannel-wasm PUBLIC ${WASM_OPTS}
		"SHELL:--js-library ${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/webrtc.js"
		"SHELL:--js-library ${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/websocket.js")

else()
	add_library(datachannel-wasm INTERFACE)

	option(NO_WEBSOCKET "Disable WebSocket support in libdatachannel" OFF)
	option(NO_MEDIA "Disable media support in libdatachannel" ON)
	option(NO_EXAMPLES "Disable examples in libdatachannel" ON)

	add_subdirectory(deps/libdatachannel EXCLUDE_FROM_ALL)
	target_link_libraries(datachannel-wasm INTERFACE datachannel-static)
endif()

