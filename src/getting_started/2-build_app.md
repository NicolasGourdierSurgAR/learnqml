# Build an app

## Files

In order to build a minimal app, you will need 3 files:
- CMakeLists.txt
- main.cpp
- Main.qml

We will not go into the details of the `Main.qml` and `main.cpp` files for now. You can find examples in the first chapter and in the [template](https://github.com/BastienHerrerosSurgAR/qml-template/).

On the cmake side, Qt gives access to new functions used to create executables or libraries.

## Create target

For `qt_add_executable`, the function will:
- Create the exe target
- Link Qt::Core
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))
- Finalize the project ([doc](https://doc.qt.io/qt-6.8/qt-finalize-project.html))

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
- **RESOURCES** is a list of resources like icons for examples. These will be accessible via the qt resource manager (qrc:/qt/.../myIcon.svg)

````admonish example "Example"

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

````
