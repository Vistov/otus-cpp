# --- 1. Основные настройки проекта ---
cmake_minimum_required(VERSION 3.20)
project(Eagle LANGUAGES CXX) # Замените 'Eagle' на имя вашей цели/проекта

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Принудительно указываем компилятор clang++ (если это нужно для локальной сборки)
# На GitHub Actions мы передаем их через командную строку
# set(CMAKE_CXX_COMPILER "clang++")
# set(CMAKE_C_COMPILER "clang")

# Режим Release по умолчанию
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()
message("Building in ${CMAKE_BUILD_TYPE} mode...")

# --- 2. Поиск зависимостей: OpenCV и FFmpeg ---

# Находим OpenCV
find_package(OpenCV REQUIRED)

# Находим компоненты FFmpeg через pkg-config
find_package(PkgConfig QUIET)
pkg_check_modules(AV_CODEC REQUIRED libavcodec)
pkg_check_modules(AV_FORMAT REQUIRED libavformat)
pkg_check_modules(AV_UTIL REQUIRED libavutil)
pkg_check_modules(SWSCALE REQUIRED libswscale)

# --- 3. Сборка исходных файлов ---
file(GLOB_RECURSE SOURCES src/*.cpp)
file(GLOB_RECURSE HEADERS include/*.hpp include/*.h)

add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# --- 4. Подключение библиотек к цели ---
target_include_directories(${PROJECT_NAME}
    PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include> # Для будущего использования
        # Пути для OpenCV
        ${OpenCV_INCLUDE_DIRS}
        # Пути для FFmpeg
        ${AV_CODEC_INCLUDE_DIRS}
        ${AV_FORMAT_INCLUDE_DIRS}
        ${AV_UTIL_INCLUDE_DIRS}
        ${SWSCALE_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}
    PRIVATE
        # Библиотеки OpenCV
        ${OpenCV_LIBS}
        # Библиотеки FFmpeg
        ${AV_CODEC_LIBRARIES}
        ${AV_FORMAT_LIBRARIES}
        ${AV_UTIL_LIBRARIES}
        ${SWSCALE_LIBRARIES}
)
