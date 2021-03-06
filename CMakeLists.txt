cmake_minimum_required(VERSION 3.17 FATAL_ERROR)

project(paniview C)

include(CTest)

# Generate PE private build string
set(BUILDSTRING "Built by: $ENV{USER} on <$ENV{NAME}>")

find_package(Git)
if(GIT_FOUND)
  execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
    RESULT_VARIABLE GIT_COMMITHASH_RESULT
    RESULT_VARIABLE GIT_COMMITHASH_OUTPUT
    OUTPUT_QUIET
    ERROR_QUIET
    OUTPUT_STRIP_TRAILING_WHITESPACE)

  # If git command succeed
  if(GIT_COMMITHASH_RESULT EQUAL 0)
    execute_process(
      COMMAND ${GIT_EXECUTABLE} config user.name
      RESULT_VARIABLE GIT_USERNAME_RESULT
      OUTPUT_VARIABLE GIT_USERNAME_OUTPUT
      OUTPUT_QUIET
      ERROR_QUIET
      OUTPUT_STRIP_TRAILING_WHITESPACE)

    execute_process(
      COMMAND ${GIT_EXECUTABLE} config user.email
      RESULT_VARIABLE GIT_EMAIL_RESULT
      OUTPUT_VARIABLE GIT_EMAIL_OUTPUT
      OUTPUT_QUIET
      ERROR_QUIET
      OUTPUT_STRIP_TRAILING_WHITESPACE)

    set(BUILDSTRING "Built by: ${GIT_USERNAME_OUTPUT} <${GIT_EMAIL_OUTPUT}>")
  else()
    message("[VersionInfo::PrivateBuildString] No Git repository.")
  endif()

endif()

configure_file(version.h.in version.h)

add_executable(${PROJECT_NAME} WIN32 paniview.c dlnklist.c paniview.rc)
target_compile_definitions(${PROJECT_NAME} PRIVATE _UNICODE UNICODE)
target_link_options(${PROJECT_NAME} PRIVATE -municode)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Werror -pedantic)
target_link_libraries(${PROJECT_NAME} PRIVATE comctl32 pathcch d2d1)

if(BUILD_TESTING)
  add_subdirectory(test)
endif()
