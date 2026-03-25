# How to define a qml component

## Inline code

The first and easiest way to create a qlm component is to just write it where you need it.

**Pros**:
- Easy to write
- Easy to find in the code

**Cons**:
- The code have to be copied if you want to use it elsewhere
- If the widget is large it may make the code difficult to read

````admonish example "Example"

Here we will create a rectangle that has certain fixed properties

```qml

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    Rectangle {
        id: myWidget

        width: 100
        height: 50

        color: "red"
        radius: 5
    }
}

```

````

## Components

The first possible solution to factorize qml code is to use components.

A component is a qml widget that works as a template. You define it in some place and then you can instantiate it multiple times.

**Pros**:
- Easy to write
- Can be used multiple times
- The component can access ids or properties of the current file

**Cons**:
- The component is only available in the current file
- If the widget is large it may make the code difficult to read

````admonish example "Example"
Here we will create a component named `MyWidget` that can be used multiple times

```qml

import myLib

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    Column {
        MyWidget{}

        MyWidget{}
    }

    component MyWidget: Rectangle {
        id: myWidget

        width: 100
        height: 50

        color: "red"
        radius: 5
    }
}

```

**Warning**: Here the id `myWidget` is only accessible inside the component

````


## Multiple files

If the component is too large, or if you want to be able to use it in other apps, you can move it inside a separate file.

```admonish info "Info"
The name of the file will be the name of the widget. It has to start with an upper case letter.
```

**Pros**:
- Can be used multiple times through multiple files/apps

**Cons**:
- Code might be difficult to update (since you'll have to update multiple files)

````admonish example "Example"
Here we will create a component names `MyWidget` in a file.

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
ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    Column {
        MyWidget{}

        MyWidget{}
    }
}
```

````

```admonish info "Advanced"
If the widget is in library (for example `myLib`), you will need to add `import myLib` at the beginning of the file (see [getting started chapter](./getting_started/3-use_library.md.md) to learn how to create a library)
```

````admonish error "Pitfalls"
Most of the time, when we move a widget out of a file, we might forget to remove properties related to the file it was in.

This can be hard to detect at first since the widget will first be used in the page it was removed from, where the properties still exists.

But when use in a completely different file, the widget will be broken.

So when creating a widget in a separated file, make sure that all the properties/ids used inside are available in the widget.

For example let's say that our code was like this:

```qml
ApplicationWindow {
    id: root

    property color myRectColor: "red"

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    Rectangle {
        id: myWidget

        width: 100
        height: 50

        color: root.myRectColor
        radius: 5
    }
}
```
Here, we create a rectangle that has the color defined in the root object.

Now we want to move out the rectangle in another file.

So we have `MyWidget.qml`:

```qml
Rectangle {
    id: myWidget

    width: 100
    height: 50

    color: root.myRectColor
    radius: 5
}
```

And the `app`:
```qml
ApplicationWindow {
    id: root

    property color myRectColor: "red"

    title: "QML app with the property"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    MyWidget {
        id: myWidget
    }
}
```

For now, everything works as intended.

But let's imagine that now you use it inside another app:

```qml
ApplicationWindow {
    id: root

    title: "QML app without the property"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    MyWidget {
        id: myWidget
    }
}
```

This app does not define `myRectColor`, so the widget cannot access it anymore.

Here the example is very simple, so it's easy to see the error. But on larger widgets it's easy to forget to remove a call to a parent or other properties.

````

## C++ widgets

We will not go into details here but you can also create widgets from the c++.

Usually you just have to create a class that inherits from QQuick widgets (for example QQuickPaintedItem for a canvas)

To make it available into the qml you have to:
- Add the source/header to you exe/lib via cmake
- In you "main.cpp", you have to register the type so that the qml can instantiate it
- Import it into the qml file


````admonish example "Example"
Here we will create a component named `MyCustomCppCanvas` in c++.

Let's take the following widget


```cpp
namespace libs::ui
{
    class MyCustomCppCanvas : public QQuickPaintedItem
    {
        ...
    }
}
```

First we add it to the CMakeLists the same way you do it for sources/headers

```cmake
set(_sources main.cpp
             MyCustomCppCanvas.hpp
             MyCustomCppCanvas.cpp
             ...
)

qt_add_executable(myExe ${_sources} ...)
```

Then we register it


```cpp
qmlRegisterType<libs::ui::MyCustomCppCanvas>("myCustomImport", 1.0, 0, "MyCustomCppCanvas");
```

- The **first parameter** is the import that will be used in the qml file
- The **second and third parameters** are the version of the widget
- The **last parameter** is the qml name

And lastly we can use it in a qml file

```qml
import myCustomImport

ApplicationWindow {
    id: root

    title: "Simple QML app"
    visible: true
    width: 1280
    height: 720
    color: "grey"

    MyCustomCppCanvas {
        ...
    }
}
```

````
