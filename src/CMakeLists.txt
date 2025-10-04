cmake_minimum_required(VERSION 3.5)
project(lzjd)
FIND_PACKAGE( Boost 1.50 COMPONENTS program_options filesystem )
INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIR} )

set(CMAKE_CXX_FLAGS "-Wall -std=c++11")

add_executable(${PROJECT_NAME} ../src/LZJD.cpp ../src/MurmurHash3.cpp main.cpp)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})

add_library(lzjd.shared SHARED ../src/LZJD.cpp ../src/MurmurHash3.cpp)
target_link_libraries(lzjd.shared)

add_library(lzjd.static STATIC ../src/LZJD.cpp ../src/MurmurHash3.cpp)

execute_process(COMMAND pg_config --includedir-server RESULT_VARIABLE PG_INCLUDESRV_RESULT OUTPUT_VARIABLE PG_INCLUDESRV_DIR OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND pg_config --includedir RESULT_VARIABLE PG_INCLUDE_RESULT OUTPUT_VARIABLE PG_INCLUDE_DIR OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND pg_config --libdir RESULT_VARIABLE PG_LIBDIR_RESULT OUTPUT_VARIABLE PG_LIB_DIR OUTPUT_STRIP_TRAILING_WHITESPACE)

if(${PG_INCLUDESRV_RESULT} EQUAL 0)
  MESSAGE( STATUS "PG Server Include: ${PG_INCLUDESRV_DIR}" )
  MESSAGE( STATUS "PG Include: ${PG_INCLUDE_DIR}" )

  add_library(lzjd_psql SHARED ../src/pglzjd.c ../src/pg_lzjd_helper.cpp ../src/LZJD.cpp ../src/MurmurHash3.cpp)
  target_include_directories(lzjd_psql PUBLIC ${PG_INCLUDESRV_DIR})
  set_target_properties(lzjd_psql PROPERTIES PREFIX "")
  set_target_properties(lzjd_psql PROPERTIES OUTPUT_NAME "lzjd_psql")
  if(APPLE)
    target_link_options(lzjd_psql PUBLIC "-Wl,-undefined,dynamic_lookup")
    target_link_libraries(lzjd_psql PUBLIC libpgcommon.a libpgport.a libpgfeutils.a)
    target_link_directories(lzjd_psql PUBLIC ${PG_LIB_DIR})
  endif()
endif()

set(CMAKE_BUILD_TYPE Release)

