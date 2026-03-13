# Getting started

This chapter is here to help you create a basic project for Qt 6.8 using cmake.

You will be able to complete this project throughout the course.

A template can be found [here](https://github.com/BastienHerrerosSurgAR/qml-template/).

## Minimal CMake

The version 6 of Qt requires at least c++17, in the following example we will ask for c++20 because why not.

### Find Qt

As any other third parties, we will need to first find it using a `find_package`.

If Qt is not found, you will have to set the `Qt6_DIR` manually. usually the path is `/path/to/qt/dir/{qt_version}/{compiler}/lib/cmake/Qt6`.

With `qt_version` the full version (ex: 6.8.6) and `compiler` the name of the compiler used (ex: gcc_64)

### Setup project

If the library is found, we will need to call `qt_standard_project_setup` in order to set the default values of some cmake variables (see the full list [here](https://doc.qt.io/qt-6.8/qt-standard-project-setup.html#description))

Thanks to this function, we no longer need to set `CMAKE_AUTOMOC` or `CMAKE_AUTOUIC` as we did when using Qt5.

**Note**: this function will also include `GNUInstallDirs`

### Example

The minimal cmake required to build a qml app is the following:

```cmake
cmake_minimum_required(VERSION 3.21)

project(qml-template LANGUAGES C CXX VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 20) #Qt6 requires at least cpp17
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 6.8 REQUIRED COMPONENTS Core Qml Quick Widgets)
if(Qt6_FOUND)
    message(STATUS "Qt found, version ${Qt6_VERSION}")
    qt_standard_project_setup(REQUIRES 6.8)
endif()
```

## Build an app

### Files

In order to build a minimal app, you will need 3 files:
- CMakeLists.txt
- main.cpp
- Main.qml

We will not go into the details of the `Main.qml` and `main.cpp` files for now. You can find examples in the first chapter and in the [template](https://github.com/BastienHerrerosSurgAR/qml-template/).

On the cmake side, Qt gives access to new functions used to create executables or libraries.

### Create target

For `qt_add_executable`, the function will:
- Create the exe target
- Link Qt::Core
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))
- Finalize the project ([doc](https://doc.qt.io/qt-6.8/qt-finalize-project.html))

### Add qml

In order to add qml files, we will need to use a new function: `qt_add_qml_module`.

We will no go through all the parameters so here is the [doc](https://doc.qt.io/qt-6.8/qt-add-qml-module.html).

Here are the parameters for a minimal case:

```cmake
qt_add_qml_module(${_exeName}
    URI ${_exeName}
    QML_FILES ${_qmlFiles}
    RESOURCES ${_qmlResources}
)
```

- **The first parameter** is the name of the target
- **URI** is the name that will be used to import the widgets into the qml
- **QML_FILES** is a list of qml files
- **RESOURCES** is a list of resources like icons for examples. These will be accessible via the qt resource manager (qrc:/qt/.../myIcon.svg)

### Example


```cmake
set(_exeName "qmlApp")

set(_qmlFiles
    app/Main.qml)

set(_qmlResources)

set(_cppSources
    main.cpp)

qt_add_executable(${_exeName} ${_cppSources})

qt_add_qml_module(${_exeName}
    URI ${_exeName}
    QML_FILES ${_qmlFiles}
    RESOURCES ${_qmlResources}
)
```

## Use a library

If you want to be able to reuse qml widgets between multiple apps, ou will have to use a library.

### Create target

Here, we won't need any c++ files (unless you want to create qml widgets in c++ of course, but we will see that later)

For `qt_add_library`, the function will:
- Create the lib target
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))

### Add qml

In order to add qml files, we will need to use a new function: `qt_add_qml_module`.

We will no go through all the parameters so here is the [doc](https://doc.qt.io/qt-6.8/qt-add-qml-module.html).

Here are the parameters for a minimal case:

```cmake
qt_add_qml_module(${_libName}
    URI ${_libName}
    QML_FILES ${_qmlFiles}
    RESOURCES ${_qmlResources}
    NO_PLUGIN)
```

- **The first parameter** is the name of the target
- **URI** is the name that will be used to import the widgets into the qml
- **QML_FILES** is a list of qml files
- **RESOURCES** is a list of resources like icons for examples. These will be accessible via the qt resource manager (qrc:/qt/.../myIcon.svg)

By default, for a library, the `qt_add_qml_module` will create another target named `myLibplugin` (if the original lib is names `myLib`) (see [doc](https://doc.qt.io/qt-6.8/qt-add-qml-module.html#targets-and-plugin-targets))

The original target will contain all the source code and the plugin target will contains the qml import logic, so that the widgets from the lib can be used in the app.

## Link to app

If you only want to use the code, you have to link the original library (ex: `myLib`). But if you also want to use the qml, you will have to also link the plugin library (ex: `myLibplugin`).

This can be a bit of a bother when you plan to always use them both.
That why we can add the `NO_PLUGIN` parameters. This will merge both targets, allowing you to only link the original target.

### Example

```cmake
set(_libName uiLib)

set(_qmlFiles
    widgets/Hello.qml
    widgets/SimpleIcon.qml)

set(_qmlResources
    assets/icon.svg)

set(_publicLinks
    Qt::Core
    Qt::Qml
    Qt::Quick
    Qt::Widgets)

qt_add_library(${_libName} STATIC)

qt_add_qml_module(${_libName}
    URI ${_libName}
    QML_FILES ${_qmlFiles}
    RESOURCES ${_qmlResources}
    NO_PLUGIN)

target_link_libraries(${_libName} PUBLIC ${_publicLinks})
```