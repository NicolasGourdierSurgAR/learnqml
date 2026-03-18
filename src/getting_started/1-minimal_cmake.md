# Minimal CMake

The version 6 of Qt requires at least c++17, in the following example we will ask for c++20 because why not.

## Find Qt

As any other third parties, we will need to first find it using a `find_package`.

If Qt is not found, you will have to set the `Qt6_DIR` manually. usually the path is `/path/to/qt/dir/{qt_version}/{compiler}/lib/cmake/Qt6`.

With `qt_version` the full version (ex: 6.8.6) and `compiler` the name of the compiler used (ex: gcc_64)

## Setup project

If the library is found, we will need to call `qt_standard_project_setup` in order to set the default values of some cmake variables (see the full list [here](https://doc.qt.io/qt-6.8/qt-standard-project-setup.html#description))

Thanks to this function, we no longer need to set `CMAKE_AUTOMOC` or `CMAKE_AUTOUIC` as we did when using Qt5.

```admonish
This function will also include `GNUInstallDirs`
```

````admonish example "Example"
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
````
