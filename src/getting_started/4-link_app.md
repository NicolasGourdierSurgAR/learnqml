# Link to app

If you only want to use the code, you have to link the original library (ex: `myLib`). But if you also want to use the qml, you will have to also link the plugin library (ex: `myLibplugin`).

This can be a bit of a bother when you plan to always use them both.
That why we can add the `NO_PLUGIN` parameters. This will merge both targets, allowing you to only link the original target.

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

````