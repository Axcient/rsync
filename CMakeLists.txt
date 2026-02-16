project(rsync)

message(STATUS "arch info: ${CMAKE_HOST_SYSTEM_PROCESSOR}")

set(RSYNC_CONFIG_OPTS --disable-md2man --with-included-popt --with-included-zlib --disable-xxhash --disable-zstd --disable-lz4 --disable-openssl)
list(JOIN RSYNC_CONFIG_OPTS " " RSYNC_CONFIG_OPTS_STR)
if(APPLE)
	if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "arm64")
		set(CPP_FLAGS "-I${OPENSSL_ROOT_DIR}/include -I/opt/homebrew/include")
		set(LD_FLAGS "-L${OPENSSL_ROOT_DIR}/lib -L/opt/homebrew/lib")
	elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "x86_64")
		set(CPP_FLAGS "-I${OPENSSL_ROOT_DIR}/include -I/usr/include")
		set(LD_FLAGS "-L${OPENSSL_ROOT_DIR}/lib -L/usr/lib")
	else()
		message(FATAL_ERROR "Unsupported architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
	endif()
	add_custom_target(
		rsync_build ALL
		COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure ${RSYNC_CONFIG_OPTS} CPPFLAGS=${CPP_FLAGS} LDFLAGS=${LD_FLAGS}
		COMMAND make -j
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT "Configuring and building rsync for ${CMAKE_HOST_SYSTEM_PROCESSOR}"
		VERBATIM
	)
elseif(NOT WIN32)
    add_custom_target(
	    rsync_build ALL
	    COMMAND bash -c "test -e rsync || \
		${CMAKE_CURRENT_SOURCE_DIR}/configure ${RSYNC_CONFIG_OPTS_STR} \
		&& make -j"
	    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
	    COMMENT "Configuring and building rsync for ${CMAKE_HOST_SYSTEM_PROCESSOR}"
	    VERBATIM
    )
elseif(EXISTS ${CYGWIN_PATH})
	execute_process(
		  COMMAND ${CMAKE_COMMAND} -E env "PATH=${CYGWIN_PATH}/bin;$ENV{PATH}" ${CYGWIN_PATH}/bin/cygpath.exe "${CMAKE_CURRENT_SOURCE_DIR}"
		  OUTPUT_VARIABLE RSYNC_CYGWIN_PATH
		  OUTPUT_STRIP_TRAILING_WHITESPACE
		)
	message(INFO " Compile rsync by cygwin ${CMAKE_CURRENT_SOURCE_DIR} -> ${RSYNC_CYGWIN_PATH}")

	add_custom_target(
		rsync_build ALL
		COMMAND ${CMAKE_COMMAND} -E env "PATH=${CYGWIN_PATH}/bin;$ENV{PATH}" ${CYGWIN_PATH}/bin/bash.exe -c " \
		        test -e rsync.exe || \
				${RSYNC_CYGWIN_PATH}/configure ${RSYNC_CONFIG_OPTS_STR} \
				&& make -j"
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT "Configuring and building rsync for ${CMAKE_HOST_SYSTEM_PROCESSOR}"
		USES_TERMINAL
		VERBATIM
	)
else()
	message(INFO " Skipping build rsync")
	add_custom_target(
		rsync_build ALL
		COMMAND ${CMAKE_COMMAND} -E echo 'Skip build rsync'
		USES_TERMINAL
	)
endif()

add_executable(rsync IMPORTED GLOBAL)
set_target_properties(rsync PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/rsync)
