cmake_minimum_required(VERSION 3.11)
project(spdlog_utests CXX)

if(NOT TARGET spdlog)
    # Stand-alone build
    find_package(spdlog REQUIRED)
endif()

include(../cmake/utils.cmake)

find_package(PkgConfig)
if(PkgConfig_FOUND)
    pkg_check_modules(systemd libsystemd)
endif()

find_package(Catch2 3 QUIET)
if(Catch2_FOUND)
    message(STATUS "Packaged version of Catch will be used.")
else()
    message(STATUS "Bundled version of Catch will be downloaded and used.")
    include(FetchContent)
    FetchContent_Declare(
        Catch2 GIT_REPOSITORY https://github.com/catchorg/Catch2.git
        GIT_TAG 53d0d913a422d356b23dd927547febdf69ee9081 # v3.5.0
    )
    FetchContent_MakeAvailable(Catch2)
endif()

set(SPDLOG_UTESTS_SOURCES
    test_file_helper.cpp
    test_file_logging.cpp
    test_daily_logger.cpp
    test_misc.cpp
    test_eventlog.cpp
    test_pattern_formatter.cpp
    test_async.cpp
    test_registry.cpp
    test_macros.cpp
    utils.cpp
    main.cpp
    test_mpmc_q.cpp
    test_dup_filter.cpp
    test_fmt_helper.cpp
    test_stdout_api.cpp
    test_backtrace.cpp
    test_create_dir.cpp
    test_custom_callbacks.cpp
    test_cfg.cpp
    test_time_point.cpp
    test_stopwatch.cpp
    test_circular_q.cpp
    test_bin_to_hex.cpp
    test_ringbuffer.cpp)

if(NOT SPDLOG_NO_EXCEPTIONS)
    list(APPEND SPDLOG_UTESTS_SOURCES test_errors.cpp)
endif()

if(systemd_FOUND)
    list(APPEND SPDLOG_UTESTS_SOURCES test_systemd.cpp)
endif()

enable_testing()

function(spdlog_prepare_test test_target spdlog_lib)
    add_executable(${test_target} ${SPDLOG_UTESTS_SOURCES})
    spdlog_enable_warnings(${test_target})
    target_link_libraries(${test_target} PRIVATE ${spdlog_lib})
    if(systemd_FOUND)
        target_link_libraries(${test_target} PRIVATE ${systemd_LIBRARIES})
    endif()
    target_link_libraries(${test_target} PRIVATE Catch2::Catch2WithMain)
    if(SPDLOG_SANITIZE_ADDRESS)
        spdlog_enable_addr_sanitizer(${test_target})
    elseif(SPDLOG_SANITIZE_THREAD)
        spdlog_enable_thread_sanitizer(${test_target})
    endif()
    add_test(NAME ${test_target} COMMAND ${test_target})
    set_tests_properties(${test_target} PROPERTIES RUN_SERIAL ON)
endfunction()

# The compiled library tests
if(SPDLOG_BUILD_TESTS OR SPDLOG_BUILD_ALL)
    spdlog_prepare_test(spdlog-utests spdlog::spdlog)
endif()

# The header-only library version tests
if(SPDLOG_BUILD_TESTS_HO OR SPDLOG_BUILD_ALL)
    spdlog_prepare_test(spdlog-utests-ho spdlog::spdlog_header_only)
endif()
