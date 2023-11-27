project(rsync)

if(${TARGET_ARCH} STREQUAL "ARM")
    set(CPPFLAGS "-I/opt/homebrew/include")
    set(LDFLAGS "-L/opt/homebrew/lib")
elseif(${TARGET_ARCH} STREQUAL "AMD")
    set(CPPFLAGS "-I/usr/local/include")
    set(LDFLAGS "-L/usr/local/lib")
else()
    message(FATAL_ERROR "Unsupported architecture: ${TARGET_ARCH}")
endif()

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/rsync
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure --disable-md2man ${CPPFLAGS} ${LDFLAGS}
    COMMAND make
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Configuring and building rsync for ${TARGET_ARCH}"
    VERBATIM
)

add_custom_target(
    rsync_build ALL
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/rsync
)

add_executable(rsync IMPORTED GLOBAL)
set_target_properties(rsync PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/rsync)
