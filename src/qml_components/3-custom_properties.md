# Custom properties

When you create a widget, you may want to add properties that when set, will modify the internal appearance of your widget.

Let's take the following widget (named `MyRect`) as an example for this section.

In the file `MyRect.qml`
```qml
import QtQuick
import QtQuick.Controls

Rectangle {
    id: root

    width: 100
    height: 50

    color: "red"
    radius: 5

    Label {
        id: myInnerLabel

        anchors.centerIn: parent
    }
}
```

In your app:

```qml
import myLib

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    Column {
        MyRect{}

        MyRect{}
    }
}
```

Here the `myInnerLabel` id is inaccessible from the app, so we can't customize its value when we instantiate the widget.

## New properties

To create a new property, you just need to define it in you widget

In the file `MyRect.qml`
```qml
import QtQuick
import QtQuick.Controls

Rectangle {
    id: root

    property string myTextProperty: "initial value"

    width: 100
    height: 50

    color: "red"
    radius: 5

    Label {
        id: myInnerLabel

        anchors.centerIn: parent

        text: root.myTextProperty
    }
}
```

In your app:

```qml
import myLib

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    Column {
        MyRect{
            myTextProperty: "1"
        }

        MyRect{}
    }
}
```

Here we have a property that is optional.

In the first case, the label will display `"1"`, and in the second it will display `"initial value"`

Here the issue is that if the text of the inner label is updated, the property is not. If this is not a behavior we want, we will need to use another strategy.

## Aliases

Aliases can be seen as just a forward of a property. When the user will access the alias from the widget, he will in fact directly access the property of the inner widget. Meaning that the updates can be done in both ways:
- The alias is updated so the inner property is also updated
- The inner property is updated so the alias is also updated

Which means that all the default signals (xxxChanged) will be triggered properly.

In the file `MyRect.qml`
```qml
import QtQuick
import QtQuick.Controls

Rectangle {
    id: root

    property alias myTextProperty: myInnerLabel.text

    width: 100
    height: 50

    color: "red"
    radius: 5

    Label {
        id: myInnerLabel

        anchors.centerIn: parent

        text: "initial value"
    }
}
```

In your app:

```qml
import myLib

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    Column {
        MyRect{
            myTextProperty: "1"
        }

        MyRect{}
    }
}
```

Here the property is still optional.

In the first case, the label will display `"1"`, and in the second it will display `"initial value"`.

But when `myInnerLabel.text` is updated, the `MyRect` widget will emit `myTextPropertyChanged` signal.

## Attributes

Until now, all the properties we created are optional and can be read/set. But Qt provide some keywords to add new behavior to the properties.

Here are some of the existing [attributes](https://doc.qt.io/qt-6.8/qtqml-syntax-objectattributes.html#property-attributes):

### [Readonly](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#read-only-properties)

These properties have a static value and cannot be assigned directly.

Nevertheless, if the property value is binded or an alias of another property, it's content can changed.
Which means that the property is not constant.

### [Required](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#required-properties)

These properties are properties that need to be set when we instantiate a widget.

**Note**: You cannot set a default value to a required property

**Note**: A required property cannot be an alias

### [Default](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#default-properties)

A default property is a property that will be filled when we instantiate a widget inside the current widget.

Let's use an example:

In the file `MyRect.qml`
```qml
import QtQuick
import QtQuick.Controls

Rectangle {
    id: root

    default property var myContent
}
```

In your app:

```qml
import myLib

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    MyRect{
        Label {
            text: "I'm inside the default property"
        }
    }

    MyRect{
        myContent: Label {
            text: "I'm inside the default property"
        }
    }
}
```

Here, the label will be stored inside our property. Also, both declaration works, the name of the property is completely optional.

You'll find this in lot's of controls from the Qt library, for example when you want to set the background or the content of a widget.

```admonish note "Note"
An object can only have one default property
```
