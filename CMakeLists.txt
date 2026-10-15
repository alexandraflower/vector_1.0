cmake_minimum_required(VERSION 3.16)

project(Vector3DProject
    VERSION 1.0.0
    LANGUAGES CXX
)

# Стандарт C++
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ---------------------------------------------------------
# Библиотека Vector3D
# ---------------------------------------------------------
add_library(Vector3D_lib STATIC 
    Vector3D.cpp 
    Vector3D.hpp
)

# Указываем, где искать заголовки (PUBLIC передаст путь исполняемым файлам и тестам)
target_include_directories(Vector3D_lib PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# Предупреждения компилятора вешаем на саму библиотеку
if (MSVC)
    target_compile_options(Vector3D_lib PRIVATE /W4)
else()
    target_compile_options(Vector3D_lib PRIVATE -Wall -Wextra -pedantic)
endif()

# ---------------------------------------------------------
# Основное приложение
# ---------------------------------------------------------
add_executable(vector_app main.cpp)
target_link_libraries(vector_app PRIVATE Vector3D_lib)

# ---------------------------------------------------------
# Тестирование (UnitTest++)
# ---------------------------------------------------------
enable_testing()

# Ищем системный UnitTest++
find_package(PkgConfig QUIET)
if (PKG_CONFIG_FOUND)
    pkg_check_modules(UNITTEST QUIET unittest++)
endif()

add_executable(run_tests tests.cpp)

if (UNITTEST_FOUND)
    target_include_directories(run_tests PRIVATE ${UNITTEST_INCLUDE_DIRS})
    target_link_libraries(run_tests PRIVATE Vector3D_lib ${UNITTEST_LIBRARIES})
else()
    # Запасной вариант, если find_package / pkg-config не нашел готовый таргет
    find_library(UNITTEST_LIB NAMES UnitTest++ unittest++)
    if (UNITTEST_LIB)
        target_link_libraries(run_tests PRIVATE Vector3D_lib ${UNITTEST_LIB})
    else()
        target_link_libraries(run_tests PRIVATE Vector3D_lib UnitTest++)
    endif()
endif()

add_test(NAME Vector3D_AllTests COMMAND run_tests)

option(ENABLE_COVERAGE "Enable code coverage testing" OFF)

if (ENABLE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(Vector3D_lib PRIVATE --coverage -O0 -g)
    target_link_options(Vector3D_lib PUBLIC --coverage)
endif()