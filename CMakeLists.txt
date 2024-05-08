cmake_minimum_required(VERSION 3.28)
project(bridge)

set(CMAKE_CXX_STANDARD 17)

function(download_file url filename)
    message(STATUS "Download to ${filename} ...")
    file(DOWNLOAD ${url} ${filename})
endfunction()

function(download_file_with_hash url filename hash_type hash)
    message(STATUS "Download to ${filename} ...")
    if (${hash_type}!="NULL")
        file(DOWNLOAD ${url} ${filename} EXPECTED_HASH ${hash_type}=${hash})
    else ()
        file(DOWNLOAD ${url} ${filename})
    endif ()

endfunction()

function(extract_file filename extract_dir)
    message(STATUS "Extract to ${extract_dir} ...")

    set(temp_dir ${CMAKE_BINARY_DIR}/tmp_for_extract.dir)
    if(EXISTS ${temp_dir})
        file(REMOVE_RECURSE ${temp_dir})
    endif()
    file(MAKE_DIRECTORY ${temp_dir})
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar -xf ${filename}
            WORKING_DIRECTORY ${temp_dir})

    file(GLOB contents "${temp_dir}/*")
    list(LENGTH contents n)
    if(NOT n EQUAL 1 OR NOT IS_DIRECTORY "${contents}")
        set(contents "${temp_dir}")
    endif()

    get_filename_component(contents ${contents} ABSOLUTE)
    file(INSTALL "${contents}/" DESTINATION ${extract_dir})

    file(REMOVE_RECURSE ${temp_dir})
endfunction()

function(download_and_extract)
    set(options REMOVE_EXTRACT_DIR_IF_EXISTS)
    set(oneValueArgs DESTINATION RENAME)
    set(multiValueArgs)
    set(oneValueArgs URL FILENAME HASH_TYPE HASH EXTRACT_DIR)
    cmake_parse_arguments(DAE "${options}" "${oneValueArgs}" "${multiValueArgs}"
            ${ARGN})
    if(NOT DEFINED DAE_URL)
        message(FATAL_ERROR "Missing URL")
    endif()
    if(NOT DEFINED DAE_FILENAME)
        message(FATAL_ERROR "Missing FILENAME")
    endif()
    if(NOT DEFINED DAE_HASH_TYPE)
        message(FATAL_ERROR "Missing HASH_TYPE")
    endif()
    if(NOT DEFINED DAE_HASH)
        message(FATAL_ERROR "Missing HASH")
    endif()
    if(NOT DEFINED DAE_EXTRACT_DIR)
        message(FATAL_ERROR "Missing EXTRACT_DIR")
    endif()

    if(EXISTS ${DAE_EXTRACT_DIR})
        if(DAE_REMOVE_EXTRACT_DIR_IF_EXISTS)
            message(STATUS "${DAE_EXTRACT_DIR} already exists, removing...")
            file(REMOVE_RECURSE ${DAE_EXTRACT_DIR})
        else()
            message(
                    STATUS "${DAE_EXTRACT_DIR} already exists, skip download & extract")
            return()
        endif()
    endif()

    if(EXISTS ${DAE_FILENAME})
        file(${DAE_HASH_TYPE} ${DAE_FILENAME} _ACTUAL_CHKSUM)

        if(NOT (${_EXPECT_HASH} STREQUAL ${_ACTUAL_CHKSUM}))
            message(STATUS "Expect ${DAE_HASH_TYPE}=${_EXPECT_HASH}")
            message(STATUS "Actual ${DAE_HASH_TYPE}=${_ACTUAL_CHKSUM}")
            message(WARNING "File hash mismatch, remove & retry ...")
            file(REMOVE ${DAE_FILENAME})
            download_file_with_hash(${DAE_URL} ${DAE_FILENAME} ${DAE_HASH_TYPE}
                    ${_EXPECT_HASH})
        else()
            message(STATUS "Using exists local file ${DAE_FILENAME}")
        endif()
    else()
        download_file_with_hash(${DAE_URL} ${DAE_FILENAME} ${DAE_HASH_TYPE} ${_EXPECT_HASH})
    endif()
    extract_file(${DAE_FILENAME} ${DAE_EXTRACT_DIR})
endfunction()

download_and_extract(
        URL "https://github.com/FFmpeg/FFmpeg/archive/refs/heads/master.zip"
        FILENAME "FFmpeg.zip"
        HASH_TYPE "NULL"
        HASH "asdf"
        EXTRACT_DIR "./extern")

add_subdirectory(core)
add_subdirectory(play)

add_executable(bridge main.cpp)
target_link_libraries(bridge PRIVATE core)
target_link_libraries(bridge PRIVATE play)
