# Implicit sizes

## Default values

After the first examples, you might wonder what happens when you leave out some of these properties. Let's take the following example, where the **Rectangle** has a size but no position (from now on, we will not always write the imports, for the sake of brevity):

```qml
ApplicationWindow {
    id: root

    width: 200
    height: 200

    title: "Hello world"
    visible: true
    
    Rectangle {
        width: 50
        height: 50

        color: "red"
    }
}
```

Let's run it:

![image](./images/NoXAndYRect.png)

You guessed it: every property has a default value. Here, x and y default to 0. For a **Rectangle**, width and height also default to 0, but this is not the case for every item. You will understand why by the end of this page.

So, if we take the last example and remove the width and height, the red **Rectangle** should disappear (or rather, it should have a width and height of 0):
```qml
ApplicationWindow {
    id: root

    width: 200
    height: 200

    title: "Hello world"
    visible: true
    
    Rectangle {
        color: "red"
    }
}
```

This produces the following output:

![image](./images/NoSizeRect.png)

## Implicit sizes

In QML, every item has an implicit size (`implicitWidth` and `implicitHeight`) and an actual size (`width` and `height`). This distinction is important to grasp: it is a very useful tool to let items size themselves automatically, but it is also a common source of errors when not fully understood.

The `implicitWidth` of an item is its **natural** width, the one it would like to have, and the `implicitHeight` is, of course, its natural height. Each type should define an implicit size that makes sense in its context. For example, the implicit width of a `Text` is exactly the width its content needs to be fully displayed, depending on its font size, boldness, letter spacing, etc. Similarly, the implicit width of a `Button` is roughly the width of its text, plus its icon if it has one, plus the spacing between the icon and the text, plus some padding around all that.

We can actually show this by logging the `implicitWidth` of a `Text` once it is created:

```qml
Text {
    text: "aa"

    Component.onCompleted: console.log(implicitWidth)
}
```

gives the output:

```text
15.40625
```

whereas:

```qml
Text {
    text: "aaaaaaa"

    Component.onCompleted: console.log(implicitWidth)
}
```

gives the output:

```text
53.921875
```

This shows that the implicit width of a `Text` depends on its content. (The exact values depend on the font used on your system, so do not worry if yours differ.)

Here is the key point: as long as you do not set an item's `width` explicitly, its `width` follows its `implicitWidth` (and likewise for `height` and `implicitHeight`). So the default width of a **Rectangle** is not really "0": it is its implicit width, which happens to be 0 for a **Rectangle**. This is also why the **Text** of our very first "Hello world" example was displayed correctly even though we never gave it a size: it simply took its implicit size.

When you *use* an item, you should almost never modify its implicit size. The implicit size is meant to be defined once, when *designing* the item. When you want to change the size of an item you use, set its plain `width` and `height`.

When you design your own reusable items later in this course, choosing an appropriate implicit size, and making sure the item still behaves well when made smaller or larger than that, will be crucial to make them truly reusable.
