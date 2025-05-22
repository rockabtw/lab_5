cmake_minimum_required(VERSION 3.10)  # Обновлено до 3.10 как рекомендует предупреждение
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})
project(physics VERSION ${PROJECT_VERSION})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CPACK_PROJECT_NAME physics)

# Включение математических констант
if (MSVC)
    add_definitions(-D_USE_MATH_DEFINES)
endif()

# Поиск SFML с явным указанием версии
find_package(SFML 2.6 COMPONENTS graphics system window REQUIRED QUIET)

if(SFML_FOUND)
    message(STATUS "Found SFML ${SFML_VERSION} at ${SFML_DIR}")
else()
    message(STATUS "SFML not found, building from source")
    
    include(FetchContent)
    FetchContent_Declare(
        SFML
        GIT_REPOSITORY https://github.com/SFML/SFML.git
        GIT_TAG 2.6.1  # Конкретная стабильная версия
    )
    
    # Настройки для сборки SFML
    set(SFML_BUILD_WINDOW ON CACHE BOOL "" FORCE)
    set(SFML_BUILD_GRAPHICS ON CACHE BOOL "" FORCE)
    set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    set(SFML_DEPENDENCIES_INSTALL_DIR "${CMAKE_BINARY_DIR}/deps" CACHE PATH "" FORCE)
    set(WARNINGS_AS_ERRORS OFF CACHE BOOL "" FORCE)
    
    FetchContent_MakeAvailable(SFML)
endif()

add_executable(physics
    main.cpp World.cpp Ball.cpp Physics.cpp Color.cpp Painter.cpp Dust.cpp
    black_box/Application.cpp black_box/View.cpp black_box/PainterImpl.cpp)

# Подключение SFML с проверкой
if(TARGET sfml-graphics)
    target_link_libraries(physics PRIVATE sfml-graphics sfml-window sfml-system)
elseif(SFML_FOUND)
    target_link_libraries(physics PRIVATE SFML::Graphics SFML::Window SFML::System)
else()
    message(FATAL_ERROR "SFML graphics component not available")
endif()

# Настройки компилятора
target_compile_options(physics PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>)

# Установка
install(TARGETS physics DESTINATION bin)

# Настройка CPack
set(CPACK_GENERATOR "ZIP;DEB")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_NAME "physics")
include(CPack)