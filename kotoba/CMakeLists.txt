# YDLidar Kotoba codec (protocol framing / scan-sample records).
# No USB and no C++ FFI. The host owns the port.

set(KOTOBA_DIR ${CMAKE_CURRENT_SOURCE_DIR})

if(NOT KOTOBA_EXECUTABLE)
  find_program(KOTOBA_EXECUTABLE kotoba)
endif()

if(KOTOBA_EXECUTABLE)
  add_custom_target(ydlidar_kotoba
    COMMAND ${KOTOBA_DIR}/scripts/ci.sh
    WORKING_DIRECTORY ${KOTOBA_DIR}
    COMMENT "Compile kotoba/ to wasm and run fixture tests"
    VERBATIM)
  add_test(NAME ydlidar_kotoba_fixtures
           COMMAND ${KOTOBA_DIR}/scripts/ci.sh
           WORKING_DIRECTORY ${KOTOBA_DIR})
else()
  message(STATUS "kotoba CLI not found; skip kotoba codec target")
endif()

file(GLOB KOTOBA_EXAMPLES "${KOTOBA_DIR}/examples/*.kotoba")
foreach(child ${KOTOBA_EXAMPLES})
  install(FILES ${child} DESTINATION share/YDLIDAR_SDK/kotoba/examples)
endforeach()
install(FILES ${KOTOBA_DIR}/src/ydlidar.kotoba DESTINATION share/YDLIDAR_SDK/kotoba)
install(FILES ${KOTOBA_DIR}/README.md DESTINATION share/YDLIDAR_SDK/kotoba)
