# Basic positioning

There are several ways to define the position and size of items in QML. Some are simpler than others, some are more automatic, and each has its own use cases. Let's start with the basics.

## x, y, width, height

You may have noticed that the previous example used *x*, *y*, *width* and *height*. These are pretty straightforward: *x* and *y* define the position of an item **relative to its parent**, while *width* and *height* define its size. The (0,0) point is the top-left corner of the parent. The following QML example should speak for itself.
```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    width: 400
    height: 400

    title: "Hello World"
    visible: true
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
This produces the following window:

![image](./images/NestedRectangle.png)

```admonish note "The color property"
Note that we set the *color* property in two different ways here: the first time with a hexadecimal value (`"#202020"`), the other two with an SVG color name (`"blue"`, `"red"`). You can find the full documentation for color [here](https://doc.qt.io/qt-6/qml-color.html).
```

Here is a second window, built with nothing more than the four properties we just saw:

![image](./images/TwoRectangles.png)

The code is given below, but it is folded away on purpose: you now know everything needed to write it yourself, so have a go at it first if you feel like it.

````admonish abstract "Show the code"
```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    width: 400
    height: 400

    title: "Hello World"
    visible: true
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
You have probably already guessed that fixed coordinates are a rather poor way to position and size items, and that they will not be enough for most use cases. Indeed, they are clearly not *responsive*: nothing moves when the window is resized. The first alternative is **anchors**.

Basically, **anchors** are a way to say "the left edge of this item is glued to the right edge of that item", "this item fills its parent" or "this item is centered in its parent". More precisely, they let you attach an anchor line of an item to an anchor line of another item (its parent or a sibling).

There are 7 anchor lines: left, horizontalCenter, right, top, verticalCenter, bottom and baseline. Baseline is rarely used, so we will not detail it here. You can see where the first six anchor lines are in the following picture (taken from the [Qt website](https://doc.qt.io/qt-6/qtquick-positioning-anchors.html)).

![image](./images/AnchorsLines.png)

```admonish warning "You can only anchor to a parent or a sibling"
For performance reasons, an item can only be anchored to its **direct parent** or to one of its **siblings**. Anchoring to a grandparent, to a cousin, or to any other item of the file is invalid: the anchor is ignored and Qt prints the runtime warning `Cannot anchor to an item that isn't a parent or sibling`.

This is by far the most common mistake made with anchors. When you hit it, the fix is usually to anchor in two steps through the common ancestor, or to restructure your items a little.
```

When two edges are anchored together, they touch by default. If you want to keep some space between them, use the `margins` group property: `anchors.margins` sets the same margin on every anchored edge at once, while `anchors.leftMargin`, `anchors.topMargin`, `anchors.rightMargin` and `anchors.bottomMargin` let you control each side individually.

```admonish note "Margins only apply to anchored edges"
A margin only has an effect on an edge that is actually anchored. As the Qt documentation puts it, *"anchor margins only apply to anchors; they are not a generic means of applying margins to an Item"*. Setting `anchors.margins` on an item positioned with `x` and `y` has no effect at all.
```

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    width: 400
    height: 400

    title: "Hello World"
    visible: true
    color: "#202020"

    Rectangle {
        // The same margin on every anchored edge
        anchors.fill: parent
        anchors.margins: 20

        color: "gray"

        Rectangle {
            // A different margin for each edge
            anchors.fill: parent
            anchors.topMargin: 10
            anchors.leftMargin: 40
            anchors.rightMargin: 40
            anchors.bottomMargin: 60

            color: "blue"
        }
    }
}
```
This produces the following window:

![image](./images/BasicAnchors.png)

### Anchoring shortcuts
Two combinations of anchors are so common that QML provides a shortcut for them:
- `anchors.fill` anchors all 4 sides at once, making the item exactly cover its target.
- `anchors.centerIn` anchors both the horizontal and vertical centers at once, centering the item on its target.

```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    width: 400
    height: 400

    title: "Hello World"
    visible: true
    color: "#202020"

    Rectangle {
        // Same as anchoring top, bottom, left and right to parent
        anchors.fill: parent

        color: "blue"

        Rectangle {
            // Same as anchoring horizontalCenter and verticalCenter to parent
            anchors.centerIn: parent
            width: 50
            height: 50

            color: "red"
        }
    }
}
```

With this new concept, we can go back to our two rectangles and make them responsive.
```qml
import QtQuick
import QtQuick.Controls

ApplicationWindow {
    id: root

    width: 400
    height: 400

    title: "Hello World"
    visible: true
    color: "#202020"

    Rectangle {
        id: rectangleLeft

        anchors.top: parent.top
        anchors.bottom: parent.bottom
        anchors.left: parent.left
        width: 200

        color: "red"
    }

    Rectangle {
        anchors.top: parent.top
        anchors.bottom: parent.bottom
        anchors.left: rectangleLeft.right
        anchors.right: parent.right

        color: "green"
    }
}
```
This gives the same window as before, with one big difference: try resizing it. The green rectangle now follows the right edge of the window, whereas the previous version kept two fixed 200-pixel-wide rectangles whatever the size of the window.

````admonish tip "Grouping properties"
When several properties belong to the same group, like the anchors we just saw, QML lets you set them either one by one with dots, or all together using braces. The following two snippets are strictly equivalent:
```qml
Rectangle {
    anchors.top: parent.top
    anchors.left: parent.left
    anchors.right: parent.right
}
```
```qml
Rectangle {
    anchors {
        top: parent.top
        left: parent.left
        right: parent.right
    }
}
```
The braces syntax is often preferred when you set several properties of the same group, as it avoids repeating the group name and keeps things visually together. This is not specific to **anchors**: other group properties such as **font** or **border** can be used the same way.
````

```admonish warning "Never mix anchors and explicit positioning"
Using anchors affects the x, y, width and height properties of an item, depending on which anchor lines are used. For example, anchoring both `left` and `right` will drive both `x` and `width`, while anchoring only `top` will drive `y`.

Because of this, anchors and absolute positioning **cannot be mixed for the same dimension**. If an item sets `x` and also sets `anchors.left`, or anchors its `left` and `right` edges but additionally sets a `width`, the result is *undefined*. There is no rule saying which of the two wins, so whatever you observe on your machine is not something you can rely on. The same goes for `y` and `height` together with `anchors.top`/`anchors.bottom`, or for `anchors.fill` together with `width` or `height`.

Pick one of the two approaches for a given dimension, and stick to it.
```


