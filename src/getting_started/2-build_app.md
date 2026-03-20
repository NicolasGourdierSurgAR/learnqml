# Build an app

## Files

In order to build a minimal app, you will need 3 files:
- CMakeLists.txt
- main.cpp
- Main.qml

We will not go into the details of the `Main.qml` and `main.cpp` files for now. You can find examples in the following chapters and in the [template](https://github.com/BastienHerrerosSurgAR/qml-template/).

On the cmake side, Qt gives access to new functions used to create executables or libraries.

## Create target

When you build a QML app in cmake, you have to use `qt_add_executable` instead of `add_executable`.

This cmake function takes the same parameters as the original function.

```admonish info "More details"
For `qt_add_executable`, the function will:
- Create the exe target
- Link it Qt::Core
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))
- Finalize the project ([doc](https://doc.qt.io/qt-6.8/qt-finalize-project.html))
```

You will then need to link Qt to this target using `target_add_library`. The minimal targets you need are:
- Qt::Core
- Qt::Qml
- Qt::Quick
- Qt::Widgets

````admonish example "Example"

This gives us the following code in the app CMakeLists:

```cmake
set(_exeName "qmlApp")

set(_cppSources
    main.cpp)

set(_links
    Qt::Core
    Qt::Qml
    Qt::Quick
    Qt::Widgets)

qt_add_executable(${_exeName} ${_cppSources})

target_link_libraries(${_exeName} PRIVATE ${_links})
```

````

## Add qml

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
- **RESOURCES** is a list of resources like icons for examples. These will be accessible via the qt resource manager (more on that on a later chapter)

````admonish example "Example"

This gives us the following code in the app CMakeLists:

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

````

## Main code file

Now that we have the cmake setup, we can finally have a look at the c++ and QML files.

## Main QML file

The `Main.qml` file is the first page that will be open by the app.

```admonish note "Note"
The main QML file can be named however you want, the only requirement is that the file start with an upper case letter.

In this course, it will be named `Main.qml` and stored in `src/qmlApp/app`
```

In this file, we will define the main parameter of the window define its content.

For now, we will no go further into details about these parameters.

````admonish example "Example"

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"
}

```

````

## Main c++ file

As you know, all c++ projects must have a main file containing the main function.

This main funtion will be used to initialized the QML application and start it.

To do that, you will need two things:
- A `QApplication` that will contain the main information about the app like its name for example.
- A `QQmlApplicationEngine` that will be used to load the main QML page.

In order to load the first page you will need to call the `loadFromModule` function from the `QQmlApplicationEngine`.

It take two arguments:
- The relative path to the folder containing the file
- The name of the file

````admonish example "Example"

```c++
#include <QApplication>
#include <QQmlApplicationEngine>

int main(int argc, char* argv[])
{
    QApplication app(argc, argv);

    // Init the qml engine
    QQmlApplicationEngine engine;

    // load main qml file
    engine.loadFromModule("qmlApp/app", "Main");

    const bool errorCode = app.exec();

    return errorCode;
}

```

````