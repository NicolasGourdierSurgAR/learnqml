# Your first application

If the previous chapter went well, you should already have a working QML application. Let's start here by dissecting the main QML file.

## Main.qml

Here is an example of a "Hello world" in QML (it might differ a bit from the example you got in the last chapter):
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
### The imports
As in many programming languages, each QML file starts with its imports. Qt provides several modules that you can import to use their types in a QML file. You can also create your own modules, but this will be the subject of a later chapter.

The main QML modules to know are:
- **QtQuick**: the essential primitives such as **Rectangle**, **Text** or **Image**, and a few *positioners* like **Column** or **Row** (more on those at the end of this chapter)
- **QtQuick.Controls**: a large set of common controls like **Button**, **Slider** or **ProgressBar** (the complete list can be found [here](https://doc.qt.io/qt-6/qtquickcontrols-index.html#controls))
- **QtQuick.Layouts**: a set of QML types used to arrange and resize items.

In our example, we imported **QtQuick** to use **Text** and **QtQuick.Controls** to use **ApplicationWindow**.

### The root object
Next comes the root object, here an **ApplicationWindow**. Each QML file must have exactly one object at its root.

```admonish info "Item vs Object"
Note that an **Item** and an **Object** in QML are two different things. An **Object** (the actual type being **QtObject**) is the base type for everything. An **Item** (which inherits from QtObject) is the base type for every **visual** object: it holds all the common attributes such as x, y, width, height, etc.
```

### The attributes
Every object has a set of attributes. Some of them exist by default and can be customized (like the `text` property of a **Button**), and you can also add as many of your own as you want to any QML object.

There are several kinds of attributes. We will not dig into all of them in this chapter, but here is the complete list:
- the **id** attribute
- **properties**
- **signals**
- **signal handlers**
- **functions**
- **attached properties** and **attached signal handlers**
- **enumerations**
- **child objects**
- **inline components**

````admonish tip "Organizing your files"
We highly recommend organizing the content of your files consistently from the start, to make things easier to find and modify. This pays off especially once files grow larger. Everyone has their own way of organizing a file, and most of them are valid. Here is what we propose (you will probably not understand all of the following yet, and that is not a problem):
```qml
RootItem {
    id: root // always name the root item "root"

    // The signals
    signal mySignal()

    // The enums
    enum MyEnum { First = 0, Second = 1 }

    // Custom properties
    property int myCustomProperty: 10

    // The properties relative to size or positioning
    x: 5
    width: 200

    // The other properties
    color: "red"

    // The signal handlers
    onWidthChanged: console.log("width")

    // The child objects
    Rectangle {

    }

    // The functions
    function myFunction() {

    }

    // The inline components
    component MyLocalLabel : Label {
        color: "chartreuse"
    }
}
```
````

Now, back to our Main.qml example. Here, we set a few properties on the root object:
- title: the title of the window
- visible: whether the window is displayed
- width: its width
- height: you guessed it, its height
- color: the background color

Then we added an item (specifically a **Text**) as a child of the root object, and set some of its properties (namely x, y, text and color).

Note that the **id** attribute is not mandatory (we set it for the ApplicationWindow but not for the Text).

If all went well, you should obtain the following window:
![image](./images/HelloWorld.png)


```admonish warning "The visible property"
Do not forget to set the `visible` property to true. Its default is `false`, so the window will not be visible unless you set it explicitly.
```

## Referring to other objects

Before moving on, two notions deserve a closer look, because you will use them constantly starting with the very next page.

The **id** attribute gives an object a name that the rest of the file can use to refer to it. It is not a property, and it is not a string: it is an identifier. It must begin with a lowercase letter or an underscore, and it must be unique within the file. Once an object has an id, any other object in the same file can read its properties:

```qml
Rectangle {
    id: background

    width: 200
}

Text {
    text: "I am as wide as the background"
    width: background.width
}
```

Every visual item also has a **parent**: the item it is declared inside. The `parent` property lets you refer to it without having to name it, which is convenient because an item usually cares about *its parent's size*, not about its parent's name. In our Main.qml, the **Text** is declared inside the **ApplicationWindow**.

```admonish note "The parent of a window's children"
Strictly speaking, a window is not an **Item**, so it cannot be the `parent` of an item. The children you declare in an **ApplicationWindow** are actually placed in its `contentItem`, an invisible item that covers the window's content area. In practice this changes nothing for you: writing `parent.width` in a direct child of the window gives you the width of that area, which is what you want.
```

```admonish info "Properties are bound, not assigned"
In the snippet above, `width: background.width` does not copy the value of `background.width` once and for all. It creates a **binding**: from then on, whenever `background.width` changes, the width of the **Text** changes with it, by itself, with no extra code.

This is the single most important idea in QML, and it holds for any property whose value is an expression involving other properties. We will come back to bindings in detail in a later chapter. For now, read `a: b.c` as "*a follows b.c*" rather than "*a is set to b.c*".
```
