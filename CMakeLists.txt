cmake_minimum_required (VERSION 3.13)
cmake_policy(VERSION 3.13)

project(goma)

enable_language(C)
enable_language(CXX)
enable_language(Fortran)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)

set(PROJECT_NAME brkfix)


find_package(SEACASExodus REQUIRED)
MESSAGE(STATUS "SEACASExodus_DIR = ${SEACASExodus_DIR}")
find_package(SEACASChaco REQUIRED)
MESSAGE(STATUS "SEACASChaco_DIR = ${SEACASChaco_DIR}")

set(CMAKE_CXX_COMPILER ${SEACASExodus_CXX_COMPILER} )
set(CMAKE_C_COMPILER ${SEACASExodus_C_COMPILER} )
set(CMAKE_Fortran_COMPILER ${SEACASExodus_FORTRAN_COMPILER} )

# Set a default build type for single-configuration
# CMake generators if no build type is set.
IF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
set(BRKFIX_WARNINGS "-Wall -Wextra -Wno-unused-parameter" CACHE STRING "set warnings for goma C and CXX")
set(BRKFIX_DEBUG_FLAGS "-O0 -ggdb3" CACHE STRING "set debug flags for goma C and CXX")
set(BRKFIX_RELEASE_FLAGS "-O2 -DNDEBUG" CACHE STRING "set release flags for goma C and CXX")

# Select flags.
SET(CMAKE_CXX_FLAGS ${BRKFIX_WARNINGS})
SET(CMAKE_CXX_FLAGS_RELEASE "${BRKFIX_RELEASE_FLAGS}")
SET(CMAKE_CXX_FLAGS_DEBUG  "${BRKFIX_DEBUG_FLAGS}")
SET(CMAKE_C_FLAGS ${BRKFIX_WARNINGS})
SET(CMAKE_C_FLAGS_RELEASE "${BRKFIX_RELEASE_FLAGS}")
SET(CMAKE_C_FLAGS_DEBUG  "${BRKFIX_DEBUG_FLAGS}")

message(STATUS "CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
message(STATUS "CMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS_RELEASE=${CMAKE_CXX_FLAGS_RELEASE}")
message(STATUS "CMAKE_CXX_FLAGS_DEBUG=${CMAKE_CXX_FLAGS_DEBUG}")
message("\n")
set(BRKFIX_COMPILE_DEFINITIONS HAVE_CONFIG_H=1)
foreach(d ${BRKFIX_COMPILE_DEFINITIONS})
  message(STATUS "${d}")
endforeach()
message("\n")

set(BRKFIX_TPL_INCLUDES
  ${SEACASExodus_INCLUDE_DIRS}
  ${SEACASChaco_INCLUDE_DIRS}
  ${SEACASExodus_TPL_INCLUDE_DIRS}
  ${SEACASChaco_TPL_INCLUDE_DIRS}
)

set(BRKFIX_TPL_LIBRARY_DIRS
  ${SEACASExodus_LIBRARY_DIRS}
  ${SEACASExodus_TPL_LIBRARY_DIRS}
  ${SEACASChaco_LIBRARY_DIRS}
  ${SEACASChaco_TPL_LIBRARY_DIRS}
)

set(BRKFIX_TPL_LIBRARIES
  ${SEACASExodus_LIBRARIES}
  ${SEACASExodus_TPL_LIBRARIES}
  ${SEACASChaco_LIBRARIES}
  ${SEACASChaco_TPL_LIBRARIES}
)

set(BRKFIX_INCLUDES
  aalloc.h brkfix_types.h eh.h exo_struct.h mk_dm.h rd_dpi.h
  sam_perea.h wr_coords.h acconfig.h config.h emuck.h exo_utils.h
  nodesc.h rd_exo.h std.h wr_graph_file.h bbb.h dpi.h
  exo_conn.h map_names.h ppi.h rd_in.h utils.h
)

set(BRK_SOURCES
  brk.c rd_exo.c wr_exo.c rd_in.c wr_graph_file.c mk_dm.c wr_coords.c ppi.c
  sam_perea.c wr_dpi.c utils.c aalloc.c eh.c exo_utils.c exo_conn.c emuck.c
  rd_dpi.c
)

set(FIX_SOURCES
  fix.c rd_exo.c wr_exo.c rd_dpi.c wr_dpi.c utils.c
  aalloc.c eh.c bbb.c exo_utils.c
)


add_executable(brk ${BRK_SOURCES} ${BRKFIX_INCLUDES})
target_include_directories(brk PRIVATE ${CMAKE_SOURCE_DIR} ${BRKFIX_TPL_INCLUDES})
target_link_libraries(brk PUBLIC ${BRKFIX_TPL_LIBRARIES} m)
target_compile_definitions(brk PUBLIC ${BRKFIX_COMPILE_DEFINITIONS})

add_executable(fix ${FIX_SOURCES} ${BRKFIX_INCLUDES})
target_include_directories(fix PRIVATE ${CMAKE_SOURCE_DIR} ${BRKFIX_TPL_INCLUDES})
target_link_libraries(fix PUBLIC ${BRKFIX_TPL_LIBRARIES} m)
target_compile_definitions(fix PUBLIC ${BRKFIX_COMPILE_DEFINITIONS})

install(TARGETS brk fix
  RUNTIME DESTINATION bin
)
