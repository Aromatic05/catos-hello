cmake_minimum_required(VERSION 4.0.0)

project(CatOS-Hello VERSION 0.1 LANGUAGES CXX)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(APP_TARGET catos-hello)
set(APP_I18N_PREFIX catos-hello)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 查找 Qt 6
find_package(Qt6 REQUIRED COMPONENTS Widgets LinguistTools Network)

# 定义源文件
set(PROJECT_SOURCES
    main.cpp
    mainwindow.cpp
    mainwindow.h
    mainwindow.ui
    AssistantTab.cpp AssistantTab.h GeneralNewsTab.cpp GeneralNewsTab.h InstallAppsTab.cpp InstallAppsTab.h PostInstallGuideTab.cpp PostInstallGuideTab.h TipsTab.cpp TipsTab.h
    MirrorListWindow.h MirrorListWindow.cpp RepoListWindow.h RepoListWindow.cpp
)

# 创建可执行文件
qt_add_executable(${APP_TARGET}
    ${PROJECT_SOURCES}
)

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(${APP_TARGET} PRIVATE -mno-avx512f)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(${APP_TARGET} PRIVATE /arch:AVX2)
endif()

# 定义 .ts 文件
set(TS_FILES
    ${CMAKE_SOURCE_DIR}/translations/${APP_I18N_PREFIX}_zh_CN.ts
    ${CMAKE_SOURCE_DIR}/translations/${APP_I18N_PREFIX}_fr.ts
    ${CMAKE_SOURCE_DIR}/translations/${APP_I18N_PREFIX}_de.ts
)

# 添加 lupdate 作为构建前的目标
add_custom_target(update_translations
    COMMAND lupdate ${PROJECT_SOURCES} -ts ${TS_FILES}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

# 让 lupdate 在每次构建前自动触发
add_dependencies(${APP_TARGET} update_translations)

# 生成 .qm 文件
qt_add_lrelease(${APP_TARGET}
    TS_FILES ${TS_FILES}
    QM_FILES_OUTPUT_VARIABLE QM_FILES
)

# 动态生成 translations.qrc
file(WRITE ${CMAKE_BINARY_DIR}/translations.qrc
[[<RCC>
    <qresource prefix="/translations">
        <file>catos-hello_zh_CN.qm</file>
        <file>catos-hello_fr.qm</file>
        <file>catos-hello_de.qm</file>
    </qresource>
</RCC>]])

# 添加资源文件
set(RESOURCES
    ${CMAKE_BINARY_DIR}/translations.qrc
)

target_compile_options(${APP_TARGET} PRIVATE -fsanitize=address)
target_link_options(${APP_TARGET} PRIVATE -fsanitize=address)

qt_add_resources(APP_RESOURCES_RCC ${RESOURCES})
target_sources(${APP_TARGET} PRIVATE ${APP_RESOURCES_RCC})

# 链接 Qt 库
target_link_libraries(${APP_TARGET} PRIVATE Qt6::Widgets Qt6::Network)

# 测试支持
option(BUILD_TESTS "Build tests" OFF)
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()
