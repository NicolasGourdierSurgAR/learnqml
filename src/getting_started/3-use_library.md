# Use a library

If you want to be able to reuse qml widgets between multiple apps, ou will have to use a library.

## Create target

Here, we won't need any c++ files (unless you want to create qml widgets in c++ of course, but we will see that later)

For `qt_add_library`, the function will:
- Create the lib target
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))

## Add qml

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
