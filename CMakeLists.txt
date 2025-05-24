cmake_minimum_required(VERSION 3.7.2)
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})
project(physics VERSION ${PROJECT_VERSION})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CPACK_PROJECT_NAME physics)
<<<<<<< HEAD
=======
# Пытаемся найти предустановленный SFML/
find_package(SFML COMPONENTS graphics)
>>>>>>> edcd883aeebdff16e9e3b8e0980e0a5c6e01fae9

# =============================================
# Указываем CMake, где искать SFML в MSYS2
set(SFML_DIR "C:/msys64/mingw64/lib/cmake/SFML")  # Путь для MSYS2
# =============================================

# Пытаемся найти предустановленный SFML
find_package(SFML COMPONENTS graphics system window REQUIRED)  # Добавлены system и window

# скачивание SFML
if (SFML_FOUND)
    message(STATUS "The SFML installed in the system will be used")
else()
    # Если SFML не найден - выкачиваем и собираем из исходников
    message(STATUS "Get SFML from the source code")
    include(FetchContent)
    FetchContent_Declare(SFML
            GIT_REPOSITORY https://github.com/SFML/SFML.git
            GIT_TAG 2.6.x)
    set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    set(WARNINGS_AS_ERRORS FALSE CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(SFML)
endif()

# для использования M_PI
if (MSVC)
    add_definitions(-D_USE_MATH_DEFINES)
endif()

# добавление исполняемого файла
add_executable(physics
        main.cpp World.cpp Ball.cpp Physics.cpp Color.cpp Painter.cpp Dust.cpp
        black_box/Application.cpp black_box/View.cpp black_box/PainterImpl.cpp)

# связывание с SFML
target_link_libraries(physics PRIVATE sfml-graphics sfml-system sfml-window)  # Добавлены sfml-system и sfml-window

if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std:c++17")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
endif()

target_compile_options(physics PRIVATE
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>)

# установка исполняемого файла
install(TARGETS physics DESTINATION bin)

# настройка CPack
set(CPACK_GENERATOR "ZIP;DEB")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_NAME "physics")
set(CPACK_PACKAGE_DESCRIPTION "Описание вашего проекта")
set(CPACK_PACKAGE_VENDOR "Ваше имя или компания")
set(CPACK_PACKAGE_CONTACT "Ваш контактный email")
include(CPack)