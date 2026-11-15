cmake_minimum_required(VERSION 3.16)
project(physics2d LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Ядро физики — без внешних зависимостей
add_library(physcore STATIC src/physics.cpp)
target_include_directories(physcore PUBLIC src)

# Визуализация: SFML 2.5/2.6 — сначала ищем установленную, иначе скачиваем исходники
find_package(SFML 2.5 COMPONENTS graphics window system QUIET)
if(NOT SFML_FOUND)
  message(STATUS "SFML не найдена — скачиваю SFML 2.6.1 через FetchContent")
  include(FetchContent)
  set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
  set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
  set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
  FetchContent_Declare(SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG 2.6.1)
  FetchContent_MakeAvailable(SFML)
endif()

add_executable(physics2d src/main.cpp src/bench.cpp)
target_link_libraries(physics2d PRIVATE physcore sfml-graphics sfml-window sfml-system)

# Шрифт кладём рядом с exe
add_custom_command(TARGET physics2d POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/assets $<TARGET_FILE_DIR:physics2d>/assets)
