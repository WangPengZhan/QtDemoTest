cmake_minimum_required(VERSION 3.15)

project(QtDemoTest
    VERSION 1.0.0 
    DESCRIPTION "A demo for qt application."
    HOMEPAGE_URL "https://demo.app/"
    LANGUAGES CXX)

set(PROJECT_COMPANY_NAME "demo studio")
set(PROJECT_COPYRIGHT "Copyright © iveswang and other contributors")

set(CMAKE_INCLUDE_CURRENT_DIR ON)

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag(-Wno-builtin-macro-redefined SUPPORT_CXX_WNO_BUILTIN_MACRO_REDEFINED)
if (SUPPORT_CXX_WNO_BUILTIN_MACRO_REDEFINED)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-builtin-macro-redefined")
else ()
    message("CXXCompiler can't support Wno-builtin-macro-redefined")
endif (SUPPORT_CXX_WNO_BUILTIN_MACRO_REDEFINED)

# MSVC并行编译 cmake -j xxx 对于MSVC没有作用
add_compile_options($<$<CXX_COMPILER_ID:MSVC>:/Gm->) #高版本已被废弃，但是低版本的Gm会影响并行
cmake_host_system_information(RESULT CPU_NUMBER_OF_LOGICAL_CORES QUERY NUMBER_OF_LOGICAL_CORES)
add_compile_options($<$<CXX_COMPILER_ID:MSVC>:/MP${CPU_NUMBER_OF_LOGICAL_CORES}>)

add_compile_definitions(UNICODE)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 20)

cmake_policy(SET CMP0087 NEW)
cmake_policy(SET CMP0079 NEW)

# qt
set(QT_COMPONENTS
    Core
    Gui
    Widgets
    Svg
)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets Svg)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS ${QT_COMPONENTS} LinguistTools)


include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

file(GLOB_RECURSE QTS "src/*.ts")
file(GLOB_RECURSE QRC "/resource/*.qrc")
file(GLOB_RECURSE SOURCES "src/*.cpp" "src/*.h")
file(GLOB_RECURSE UI "src/*.ui")


if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE release CACHE STRING "Build Type" FORCE)
endif ()

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

add_executable(${PROJECT_NAME}
    ${SOURCES}
    ${QT_MOC}
    ${UI}
    ${UIS}
    ${QRC}
)
include(cmake/appicon.cmake)
appicon(${PROJECT_NAME} RESOURCE_FILES)

include(cmake/appversion.cmake)
appversion()

target_sources(${PROJECT_NAME} 
    PRIVATE  
        ${RESOURCE_FILES}
)

if (WIN32)
    set_target_properties(${PROJECT_NAME} PROPERTIES WIN32_EXECUTABLE TRUE)
endif()

if (${QT_VERSION_MAJOR} EQUAL 5)
    qt5_add_translation(${PROJECT_NAME} TS_FILES ${QTS})
else ()
    qt_add_translations(${PROJECT_NAME} TS_FILES ${QTS})
endif ()

foreach (COMPONENTS ${QT_COMPONENTS})
    set(QT_LIBS ${QT_LIBS} Qt${QT_VERSION_MAJOR}::${COMPONENTS})
endforeach ()


target_link_libraries(${PROJECT_NAME}
    PRIVATE
    ${QT_LIBS}
)
include(cmake/qtdeploy.cmake)
deployqt(${PROJECT_NAME})

include(InstallRequiredSystemLibraries)
install(DIRECTORY $<TARGET_FILE_DIR:${PROJECT_NAME}>/ DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR})

include(cmake/wix.cmake)
generator_wix()

