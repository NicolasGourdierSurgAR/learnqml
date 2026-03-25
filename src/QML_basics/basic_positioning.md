## Basic positioning

There is different way to define the position and the size of items in QML, some are simpler than others, some are more automatic than others. Let's start  with the basics.

### x, y, width, height

In the previous exemple you noticed we used *x*, *y*, *width* and *height*. These are pretty straight forward, they define the position of an Item **relative to its parent**. The (0,0) point is in the top left corner. The following qml exemple should speak for itself.
```qml
ApplicationWindow {
    id: root

    title: "Hello World"
    visible: true
    width: 400
    height: 400
    color: "#202020"

    Rectangle {
        x: 100
        y: 20
        height: 100
        width: 100
        color: "blue"

        Rectangle {
            x: 25
            y: 25
            height: 50
            width: 50
            color: "red"
        }
    }
}
```
This produce the following window:

![image](./images/NestedRectangle.png)

```admonish note "The color property"
Note that we used the *color* property two different ways here. The first one was by giving an hexadecimal value (`"#202020"`), the second and third ones were by giving a svg color name (`"red"`). You can find the full documentation for color [here](https://doc.qt.io/qt-6/qml-color.html).
```

### Exercice 1
Even though it does not seem much, this is a good time to start practicing simple exercices. The proposal here, is to reproduce the following window in QML.  An exemple of how to achieve it will be given right afterward. Feel free to experiment and to try own ideas.
Window to reproduce:

![image](./images/Exercice1.png)

````admonish abstract "Exercice solution"
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

    Rectangle {
        x: 0
        y: 0
        height: 400
        width: 200
        color: "red"
    }
    Rectangle {
        x: 200
        y: 0
        height: 400
        width: 200
        color: "green"
    }
}
```
````


## Anchors
You probably already though that is a quite poor way to set the position and size of item, and that will not suffice for the majority of the use cases. Indeed, this is clearly not *responsive*. The first alternative is the **anchors**. The **anchors** allow you to bind an anchor line of an item to an anchors line of an other item (parent of sibling).
```admonish example
```
Here ...

There are 7 anchors lines: left, horizontalCenter, right, top, verticalCenter, bottom and baseline. Baseline is rarely used so we will not detail it here. You can see the location of the 6 first anchor lines in the following picture (taken from the [QT website](https://doc.qt.io/qt-6/qtquick-positioning-anchors.html)).

```admonish warning "The effect of the anchors"
Using the anchors will affect the x,y, with and height properties depending on the anchors used. It means that setting an anchors to the
```