# Qml components

In this chapter we will learn how to create and reuse qml components (also known as widgets).

## How to define a qml component

### Inline code

The first and easiest way to create a qlm component is to just write it where you need it.

Pros:
- Easy to write
- Easy to find in the code

Cons:
- The code have to be copied if you want to use it elsewhere
- If the widget is large it may make the code difficult to read

#### Example

Here we will create a rectangle that certain fixed properties

```qml
ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    Rectangle {
        id: myWidget

        width: 100
        height: 50

        color: "red"
        radius: 5
    }
}
```

### Components

The first possible solution to factorize qml code is to use components.

A component is a qml widget that works as a template. You define ir in some place and then you can instantiate it multiple times.

Pros:
- Easy to write
- Can be used multiple times
- The component can access ids or properties of the current file

Cons:
- The component is only available in the current file
- If the widget is large it may make the code difficult to read

#### Example


**Warning**: Here the id `myWidget` is only accessible inside the component

### Multiple files

If the component is too large, or if you want to be able to use it in other apps, you can move it inside a separate file.

Pros:
- Can be used multiple times through multiple files/apps

Cons:
- Code might be difficult to update (since you'll have to update multiple files)

#### Example

In `MyWidget.qml`:

```qml
import QtQuick

Rectangle {
    id: root

    width: 100
    height: 50

    color: "red"
    radius: 5
}
```

In you app:

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
        MyWidget{}

        MyWidget{}
    }
}
```

**Note**: The `import myLib` isnly needed if `MyWidget` is inside the library `myLib` (see [getting started chapter](./getting_started.md))

#### /!\ Pitfalls /!\

Most of the time, when we move a widget out of a file, we might forget to remove properties related to the file it was in.

This can be hard to detect at first since the widget will first be used in the page it was removed from, where the properties still exists.

But when use in a completely different file, the widget will be broken.

So when creating a widget in a separated file, make sure that all the properties/ids used inside are available in the widget

### C++ widgets

We will not go into details here but you can also create widgets from the c++.

Usually you just have to create a class that inherits from QQuick widgets (ex: QQuickPaintedItem for a canvas)

To make it available into the qml you have to:
- Add the source/header to you exe/lib via cmake
- In you "main.cpp", you have to register the type so that the qml can instantiate it
- Import it into the qml file


#### Example

Let's take the following widget

```cpp
namespace libs::ui
{
    class ImageDisplay : public QQuickPaintedItem
    {
        ...
    }
}
```

First we add it to the CMakeLists the same way you do it for sources/headers

```cmake

set(_sources main.cpp
             ImageDisplay.hpp
             ImageDisplay.cpp
             ...
)

qt_add_executable(myExe ${_sources} ...)

```

Then we register it


```cpp
qmlRegisterType<libs::ui::ImageDisplay>("myCustomImport", 1.0, 0, "ImageDisplay");
```

- The first parameter is the import that will be used in the qml file
- The second and third parameters are the version of the widget
- The last parameter is the qml name

And lastly we can use it in a qml file

```qml
import myCustomImport

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    visibility: Window.Maximized

    ImageDisplay {
        ...
    }
}
```

## Visual and non visual components

In qml, you can create and use widgets that does not have any visual appearance (Ex: `Row`, `Column`, `Item`, ...)

Most of these object are used to rearrange widgets on a page.

## Custom properties

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

### New properties

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

### Aliases

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

### Attributes

Until now, all the properties we created are optional and can be read/set. But Qt provide some keywords to add new behavior to the properties.

Here are some of the existing [attributes](https://doc.qt.io/qt-6.8/qtqml-syntax-objectattributes.html#property-attributes):

#### [Readonly](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#read-only-properties)

These properties have a static value and cannot be assigned directly.

Nevertheless, if the property value is binded or an alias of another property, it's content can changed.
Which means that the property is not constant.

#### [Required](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#required-properties)

These properties are properties that need to be set when we instantiate a widget.

**Note**: You cannot set a default value to a required property

**Note**: A required property cannot be an alias

#### [Default](https://doc.qt.io/qt-6/qtqml-syntax-objectattributes.html#default-properties)

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

**Note**: An object can only have one default property

## Existing components

## Exercise
