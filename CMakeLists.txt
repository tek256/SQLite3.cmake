#--------------------------------------------
# SQLite build script for amalgamation
#--------------------------------------------
cmake_minimum_required(VERSION 3.21)
project(SQLite3 VERSION 3.40.1 LANGUAGES C)

set(THREADS_PREFER_PTHREAD_FLAG YES)
find_package(Threads REQUIRED)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

option(SQLITE_BUILD_STATIC  "Build SQLite static library" ON)
option(SQLITE_BUILD_SHARED  "Build SQLite shared library" ON)

option(SQLITE_MEMDEBUG "Build SQLite memory debug features" OFF)
option(SQLITE_DEBUG    "Build SQLite debug features" OFF)
option(SQLITE_RTREE    "Build R*Tree index extension" OFF)

file(STRINGS "${PROJECT_SOURCE_DIR}/src/sqlite3.h" _sqlite_api_h_VER_STRING REGEX ".*#define[ ]+SQLITE_VERSION[ ]+")

string(REGEX MATCH "[0-9\\.]+" SQLITE_VER_STRING ${_sqlite_api_h_VER_STRING})
string(REGEX MATCHALL "[0-9]+" _sqlite_ver_LIST "${SQLITE_VER_STRING}")
list(LENGTH _sqlite_ver_LIST _sqlite_list_len)
list(GET _sqlite_ver_LIST 0 SQLITE_VER_MAJOR)
list(GET _sqlite_ver_LIST 1 SQLITE_VER_MINOR)
list(GET _sqlite_ver_LIST 2 SQLITE_VER_PATCH)
if(_sqlite_list_len EQUAL 4)
    list(GET _sqlite_ver_LIST 3 SQLITE_VER_PATCHLEVEL)
    message("Patch level: ${SQLITE_VER_PATCHLEVEL}")
endif()

message(VERBOSE "FOUND: SQLite version = ${SQLITE_VER_STRING}")

set(SQLITE_DYN_NAME "${PROJECT_NAME}")
set(SQLITE_STATIC_NAME "${SQLITE_DYN_NAME}-static")

# add include path for project
include_directories(
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
# adding compilation flags depending on options
add_compile_definitions(
  $<$<BOOL:${SQLITE_DEBUG}>:SQLITE_DEBUG>
  $<$<BOOL:${SQLITE_MEMDEBUG}>:SQLITE_MEMDEBUG>
  $<$<BOOL:${SQLITE_RTREE}>:SQLITE_RTREE>
  $<$<PLATFORM_ID:WindowsStore>:SQLITE_OS_WINRT>
)


set(SRC_LIB_FILE ${PROJECT_SOURCE_DIR}/src/sqlite3.c)
set(SRC_SHELL_FILE ${PROJECT_SOURCE_DIR}/src/shell.c)
file(GLOB HEADER_FILES ${PROJECT_SOURCE_DIR}/src/*.h)

if (SQLITE_BUILD_STATIC)
    # build static library
    add_library(${SQLITE_STATIC_NAME} STATIC ${SRC_LIB_FILE})
    add_library(SQLite3::Static ALIAS ${SQLITE_STATIC_NAME})
    set_property(TARGET ${SQLITE_STATIC_NAME} PROPERTY EXPORT_NAME Static)
endif()

if (SQLITE_BUILD_SHARED)
    # build dynamic library
    add_library(${SQLITE_DYN_NAME} SHARED ${SRC_LIB_FILE})
    add_library(${PROJECT_NAME}::Dynamic ALIAS ${SQLITE_DYN_NAME})
    set_property(TARGET ${SQLITE_DYN_NAME} PROPERTY EXPORT_NAME Dynamic)
    if (WIN32)
      set_property(TARGET ${SQLITE_DYN_NAME}
        PROPERTY
          DEFINE_SYMBOL "SQLITE_API=__declspec(dllexport)")
    endif()
endif()

# build shell executable
add_executable(shell ${SRC_SHELL_FILE})

target_link_libraries(shell
  PRIVATE
    $<IF:$<BOOL:${SQLITE_BUILD_SHARED}>,SQLite3::Dynamic,SQLite3::Static>
    Threads::Threads
    ${CMAKE_DL_LIBS})

# installation
list(APPEND TO_INSTALL shell)
if(SQLITE_BUILD_SHARED)
    list(APPEND TO_INSTALL ${SQLITE_DYN_NAME})
endif()
if(SQLITE_BUILD_STATIC)
    list(APPEND TO_INSTALL ${SQLITE_STATIC_NAME})
endif()

install(TARGETS ${TO_INSTALL} EXPORT ${PROJECT_NAME}Targets)
install(EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME}
  NAMESPACE SQLite3::)
  install(FILES ${HEADER_FILES} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
  ${PROJECT_NAME}Config.cmake
  NO_SET_AND_CHECK_MACRO
  INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME})

write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake
  COMPATIBILITY SameMajorVersion)

install(FILES
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME})
