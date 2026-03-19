# Use a library

```admonish info "Info"
This chapter is not needed to continue the course. So feel free to ignore this part.
```

If you want to go further into the creation of the project, we can also create a QML library.

Libraries can be used to shared QML widgets between multiple apps. We will see how later in the course.

```admonish info "Template"
The template with the library can be found [here](https://github.com/BastienHerrerosSurgAR/qml-template/tree/advanced/with_library)
```

## Create target

When you build a QML library in cmake, you have to use `qt_add_library` instead of `add_library`.

This cmake function takes the same parameters as the original function.

```admonish info "More details"
For `qt_add_library`, the function will:
- Create the lib target
- Finalize the target ([doc](https://doc.qt.io/qt-6.8/qt-finalize-target.html))
```

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
- **RESOURCES** is a list of resources like icons for examples. These will be accessible via the qt resource manager ( (more on that on a later chapter))

By default, for a library, the `qt_add_qml_module` will create another target named `myLibplugin` (if the original lib is names `myLib`) (see [doc](https://doc.qt.io/qt-6.8/qt-add-qml-module.html#targets-and-plugin-targets))

The original target will contain all the source code and the plugin target will contains the qml import logic, so that the widgets from the lib can be used in the app.

## Link to app

If you only want to use the code, you have to link the original library (ex: `myLib`). But if you also want to use the qml, you will have to also link the plugin library (ex: `myLibplugin`).

This can be a bit of a bother when you plan to always use them both.
That why we can add the `NO_PLUGIN` parameters. This will merge both targets, allowing you to only link the original target.

And finally, you can import the library into the qml using `import myLib`

````admonish example "Example"

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

In the app:

```qml
import uiLib

ApplicationWindow {
    id: root

    ...

    Hello {}
}
```

````