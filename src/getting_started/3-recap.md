# Recap

Let's do a summary of what we have so far.

- A cmake file used to setup the project (the root CMakeLists):

```cmake
cmake_minimum_required(VERSION 3.21)

project(qml-template LANGUAGES C CXX VERSION 1.0.0)

set(BUILD_SHARED_LIBS ON)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

set(CMAKE_CXX_STANDARD 20) #Qt6 requires at least cpp17
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 6.8 REQUIRED COMPONENTS Core Qml Quick Widgets)
if(Qt6_FOUND)
    message(STATUS "Qt found, version ${Qt6_VERSION}")
    qt_standard_project_setup(REQUIRES 6.8)
endif()


add_subdirectory(src)
```

- A cmake file used to build the application (the app CMakeLists):

```cmake
set(_exeName "qmlApp")

set(_qmlFiles
    app/Main.qml)

set(_qmlResources)

set(_cppSources
    main.cpp)

set(_links
    Qt::Core
    Qt::Qml
    Qt::Quick
    Qt::Widgets)

qt_add_executable(${_exeName} ${_cppSources})

qt_add_qml_module(${_exeName}
    URI ${_exeName}
    QML_FILES ${_qmlFiles}
    RESOURCES ${_qmlResources}
)

target_link_libraries(${_exeName} PRIVATE ${_links})
```

- A main c++ file containing the logic the initialize and start the app:

```c++
#include <QApplication>
#include <QQmlApplicationEngine>

int main(int argc, char* argv[])
{
    QApplication app(argc, argv);

    QApplication::setApplicationName("Simple QML app");
    QApplication::setApplicationVersion("1.0.0");
    QApplication::setOrganizationName("SURGAR");
    QApplication::setOrganizationDomain("surgar-surgery.com");

    // Init the qml engine
    QQmlApplicationEngine engine;

    // load main qml file
    engine.loadFromModule("qmlApp/app", "Main");

    const bool errorCode = app.exec();

    return errorCode;
}

```

- A main QML file containing the content of the first window:

```qml
import QtQuick
import QtQuick.Window
import QtQuick.Controls

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
}

```

```admonish
if you don't want to copy/paste all the code, you can clone this [template](https://github.com/BastienHerrerosSurgAR/qml-template/).
```