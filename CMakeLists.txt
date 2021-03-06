option(FORMAT_CHECK_CMAKE "Enable CMake formatting.")

find_program(CLANG_FORMAT_PROGRAM clang-format)

if (CLANG_FORMAT_PROGRAM)
  set(CLANG_FORMAT_COMMAND python ${CMAKE_CURRENT_LIST_DIR}/git-clang-format.py --binary=${CLANG_FORMAT_PROGRAM})
  set(GIT_EMPTY_TREE_HASH 4b825dc642cb6eb9a060e54bf8d69288fbee4904)

  add_custom_target(clang-format
    COMMAND ${CLANG_FORMAT_COMMAND} --diff ${GIT_EMPTY_TREE_HASH}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

  add_custom_target(check-clang-format
    COMMAND ${CLANG_FORMAT_COMMAND} --ci ${GIT_EMPTY_TREE_HASH}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

  add_custom_target(fix-clang-format
    COMMAND ${CLANG_FORMAT_COMMAND} ${GIT_EMPTY_TREE_HASH} -f
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

  list(APPEND FORMAT_TARGETS clang-format)
  list(APPEND CHECK_FORMAT_TARGETS check-clang-format)
  list(APPEND FIX_FORMAT_TARGETS fix-clang-format)
else()
  message(STATUS "clang-format not found: adding dummy Format.cmake targets")

  set(CLANG_FORMAT_NOT_FOUND_COMMAND_ARGS
    COMMAND ${CMAKE_COMMAND} -E echo "cannot run Format.cmake: clang-format not found"
    COMMAND ${CMAKE_COMMAND} -E false
  )

  add_custom_target(clang-format ${CLANG_FORMAT_NOT_FOUND_COMMAND_ARGS})
  add_custom_target(check-clang-format ${CLANG_FORMAT_NOT_FOUND_COMMAND_ARGS})
  add_custom_target(fix-clang-format ${CLANG_FORMAT_NOT_FOUND_COMMAND_ARGS})
endif()

find_program(GIT_PROGRAM git)
find_program(CMAKE_FORMAT_PROGRAM cmake-format)

if (GIT_PROGRAM AND CMAKE_FORMAT_PROGRAM)
  function (add_cmake_format_target name)
    add_custom_target(${name}
      COMMAND ${CMAKE_COMMAND} -DGIT_PROGRAM=${GIT_PROGRAM} -DCMAKE_FORMAT_PROGRAM=${CMAKE_FORMAT_PROGRAM}
        -DCMAKE_FORMAT_TARGET=${name} -DBINARY_DIR="${CMAKE_CURRENT_BINARY_DIR}"
        -P ${CMAKE_CURRENT_LIST_DIR}/cmake-format.cmake
      WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )
  endfunction()

  add_cmake_format_target(cmake-format)
  add_cmake_format_target(check-cmake-format)
  add_cmake_format_target(fix-cmake-format)

  if (FORMAT_CHECK_CMAKE)
    list(APPEND FORMAT_TARGETS cmake-format)
    list(APPEND CHECK_FORMAT_TARGETS check-cmake-format)
    list(APPEND FIX_FORMAT_TARGETS fix-cmake-format)
  endif()
else()
  message(STATUS "cmake-format or git not found: adding dummy Format.cmake targets")

  set(CMAKE_FORMAT_NOT_FOUND_COMMAND_ARGS
    COMMAND ${CMAKE_COMMAND} -E echo "cannot run Format.cmake: cmake-format or git not found"
    COMMAND ${CMAKE_COMMAND} -E false
  )

  add_custom_target(cmake-format ${CMAKE_FORMAT_NOT_FOUND_COMMAND_ARGS})
  add_custom_target(check-cmake-format ${CMAKE_FORMAT_NOT_FOUND_COMMAND_ARGS})
  add_custom_target(fix-cmake-format ${CMAKE_FORMAT_NOT_FOUND_COMMAND_ARGS})
endif()

add_custom_target(format)
add_custom_target(check-format)
add_custom_target(fix-format)

add_dependencies(format ${FORMAT_TARGETS})
add_dependencies(check-format ${CHECK_FORMAT_TARGETS})
add_dependencies(fix-format ${FIX_FORMAT_TARGETS})
