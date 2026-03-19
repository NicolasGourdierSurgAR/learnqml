# Your first application

If the previous chapter went well, you should already have a working QML application. Let's start here by dissecting the two main files.

## Main.qml

Here is an exemple of a an "Hello world" in QML:
```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    title: "Hello World"
    visible: true
    width: 400
    height: 400
    color: "#202020"

    Text {
        x: 10
        y: 10
        text: "Hello world"
        color: "white"
    }
}
```

As in many programming language each QML file start by the imports. Qt provide several modules that can be imported to use different items in a QML file. Of course, you can create your own module, but this will be the subject of a later chapter.

The main qml modules to know are:
- QtQuick: The essential primitives such as **Rectangle**, **Text** or **Image** and a few *Positioner* like **Column** or **Row** (more on that incoming chapter)
- QtQuick.Controls: A large set of common controls like **Button**, **Slider**, **ProgressBar** (The completed list can be found [here](https://doc.qt.io/qt-6/qtquickcontrols-index.html#controls))
- QtQuick.Layouts: A set of QML types used to arrange and resize items.

In our exemple, we imported **QtQuick** to use **Text** and **QtQuick.Controls** to use **ApplicationWindow**.

Next, we've got an object at the root, here it is **ApplicationWindow**. Each QML file must have one single object at its root.

```admonish info "Item vs Object"
Note that an **Item** and an **Object** in QML are two different things: an Item is a visual primitive whereas an object is not visual.
```

-- the properties

```admonish warning "The visible property"
Do not forget to set the `visible` property to true. For some reason its default is `false`, hence by default the window will not be visible.
```