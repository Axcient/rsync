project(rsync)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/rsync
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure --disable-md2man CPPFLAGS=-I/opt/homebrew/include LDFLAGS=-L/opt/homebrew/lib
    COMMAND make
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Configuring and building rsync"
    VERBATIM
)

add_custom_target(
    rsync_build ALL
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/rsync
)

add_executable(rsync IMPORTED GLOBAL)
set_target_properties(rsync PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/rsync)

# install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/rsync DESTINATION ${CMAKE_BINARY_DIR}/bin)

# include("../../cmake/paths_and_names.cmake") # Adjust the path as per your directory structure

# message(STATUS "Agent install path: ${AGENT_INSTALL_PATH}")

# install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/rsync COMPONENT agent DESTINATION "${AGENT_INSTALL_PATH}"
# 		PERMISSIONS
# 			OWNER_WRITE OWNER_READ OWNER_EXECUTE
# 			GROUP_READ GROUP_EXECUTE
# 			WORLD_READ WORLD_EXECUTE)