cmake_minimum_required(VERSION 3.15)

find_package(Threads)

if (CMAKE_USE_OPENSSL)
    add_library(test_server STATIC
                abstractServer.cpp
                httpServer.cpp
                httpsServer.cpp)
else ()
    add_library(test_server STATIC
                abstractServer.cpp
                httpServer.cpp)
endif()
target_link_libraries(test_server PRIVATE Threads::Threads cpr::cpr GTest::GTest
                                             PUBLIC mongoose)

macro(add_cpr_test _TEST_NAME)
    add_executable(${_TEST_NAME}_tests ${_TEST_NAME}_tests.cpp)
    target_link_libraries(${_TEST_NAME}_tests PRIVATE
        test_server
        GTest::GTest
        cpr::cpr
        CURL::libcurl)
    add_test(NAME cpr_${_TEST_NAME}_tests COMMAND ${_TEST_NAME}_tests)
    # Group under the "tests" project folder in IDEs such as Visual Studio.
    set_property(TARGET ${_TEST_NAME}_tests PROPERTY FOLDER "tests")
    if(WIN32 AND BUILD_SHARED_LIBS)
        add_custom_command(TARGET ${_TEST_NAME}_tests POST_BUILD
                           COMMAND ${CMAKE_COMMAND} -E copy
                                $<TARGET_FILE:libcurl> $<TARGET_FILE_DIR:${_TEST_NAME}_tests>)
        add_custom_command(TARGET ${_TEST_NAME}_tests POST_BUILD
                           COMMAND ${CMAKE_COMMAND} -E copy
                                $<TARGET_FILE:cpr> $<TARGET_FILE_DIR:${_TEST_NAME}_tests>)
    endif()
endmacro()

add_cpr_test(get)
add_cpr_test(post)
add_cpr_test(session)
add_cpr_test(async)
add_cpr_test(proxy)
add_cpr_test(head)
add_cpr_test(delete)
add_cpr_test(put)
add_cpr_test(callback)
add_cpr_test(raw_body)
add_cpr_test(options)
add_cpr_test(patch)
add_cpr_test(error)
add_cpr_test(alternating)
add_cpr_test(util)
add_cpr_test(structures)

if (CMAKE_USE_OPENSSL)
    add_cpr_test(ssl)
    # Install all ssl keys and certs:
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/data/server.key" "${CMAKE_BINARY_DIR}/bin/server.key"  COPYONLY)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/data/server.cer" "${CMAKE_BINARY_DIR}/bin/server.cer"  COPYONLY)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/data/client.key" "${CMAKE_BINARY_DIR}/bin/client.key"  COPYONLY)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/data/client.cer" "${CMAKE_BINARY_DIR}/bin/client.cer"  COPYONLY)
endif()
