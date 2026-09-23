# Property binding

We have already used bindings in every chapter so far, without ever naming them. It is time to look at them properly, because they are the single idea that makes QML what it is.

## A binding is not an assignment

Here are two lines that look almost identical:

```qml
width: 200            // a value
width: parent.width   // a binding
```

The first one sets the width to 200, once and for all. The second one does **not** set the width to whatever `parent.width` happens to be at that moment. It establishes a *relationship*: from now on, this item is as wide as its parent, resize the parent and the child follows.

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
        // Always half the window, whatever happens to the window
        width: root.width / 2
        height: root.height

        color: "steelblue"
    }
}
```

Here, the rectangle keeps half the width of the window, and the full height of the window.

Any property can be bound, not just sizes, and the expression can be anything that produces a value of the right type:

```qml
color: root.enabled ? "steelblue" : "gray"
text: "You have " + root.messageCount + " messages"
visible: root.messageCount > 0
```

## How it works: change signals

You already know the mechanism from the [previous chapter](./signals_and_signal_handlers.md).

When you write a binding, the engine evaluates the expression once and watches which properties it reads along the way. For each of them it connects to the corresponding change signal. When any of those signals fires, the expression is evaluated again and the result is written to the property.

A binding is, in essence, a set of signal handlers that QML writes and maintains for you. `myVar: root.width + root.height` is roughly "connect to `root.onWidthChanged` and `root.onHeightChanged`, and on every emission recompute `root.width + root.height` and assign it".

```admonish info "No change signal, no binding"
This explains a limitation you will run into much later, when exposing C++ objects to QML. A property that has no change signal cannot notify anything, so nothing can be kept up to date from it. QML will read it once, and then never hear about it again. Properties declared in QML always have a change signal, which is why the question never comes up until C++ enters the picture.
```

## A binding can be a whole program

A binding does not have to be a one-liner. The value of a property can be a complete block of JavaScript, with conditions, loops, local variables and function calls:

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

    property int itemCount: 3
    property bool compact: false

    Rectangle {
        anchors.centerIn: parent
        height: 200
        color: "steelblue"

        width: {
            if (root.compact) {
                return 100;
            }

            var total = 0;
            for (var i = 0; i < root.itemCount; ++i) {
                total += 40;
            }

            return Math.min(total, root.width);
        }
    }
}
```

The rule does not change one bit. Every property this block reads while it runs becomes a dependency: here `root.compact`, `root.itemCount` and `root.width`. Change any one of them and **the entire block is evaluated again, from the top**. There is no partial re-evaluation, and no way to ask for one.

```admonish note "Multi-line bindings need a return"
A binding written as a single expression returns its value implicitly. As soon as you use braces, you are writing a JavaScript block and you must `return` explicitly, as above. Forgetting the `return` gives a property stuck at its default value, with no warning at all.
```

## Prefer a binding to a handler

It is always possible to get the same result with a change handler. It is almost always worse:

```qml
// Works, but reinvents what QML already does
onWidthChanged: label.width = root.width / 2

// Says it once, declaratively
width: root.width / 2
```

The difference is not about typing less. It is about being able to answer the question *"why does this property have this value?"*.

With a binding, the answer is always in the same place: the line where the property is declared.

With handlers, a property can be assigned from anywhere — a change handler, a button, a timer, a function called by something else — and the only way to find out what set it is to search the entire file, and the rest of the project:

## Binding loops

Since a binding re-evaluates when its dependencies change, and since evaluating it changes a property, it is entirely possible to build a circle.

```qml
Rectangle {
    id: outer

    width: inner.width + 20   // outer follows inner

    Rectangle {
        id: inner

        width: outer.width - 20   // and inner follows outer
    }
}
```

Each property waits for the other, forever. Qt notices this one and prints a message at runtime:

```text
QML Rectangle: Binding loop detected for property "width"
```

Then it breaks the cycle, leaving the property at whatever value it had reached. Your application keeps running, which is precisely why these are easy to ignore until something looks subtly wrong.

```admonish warning "Loops are rarely this obvious"
The two-line version above is the textbook case. Real ones hide much better. We will not detail them here for now, to avoid digging too early into advanced concept.
```

## Breaking a binding

A common source of error comes from breaking a property binding.

A property holds *either* a binding *or* a value. The moment you assign a value to a bound property from JavaScript, **the binding is destroyed**, permanently.

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
        id: box

        anchors.centerIn: parent
        width: root.width / 2   // a binding
        height: 100

        color: "steelblue"
    }

    Button {
        anchors.bottom: parent.bottom
        anchors.horizontalCenter: parent.horizontalCenter
        text: "Break it"

        onClicked: box.width = 150
    }
}
```

Resize the window and the rectangle follows, as expected. Now click the button once, and resize again: the rectangle is stuck at 150 because the binding is gone.

This is the mechanism behind a whole family of bugs that are can be hard to track down. Juste keep in mind for now that this can be a potential source of errors.

```admonish tip "Log broken bindings automatically"
Qt can tell you every time this happens. There is a logging category for it, `qt.qml.binding.removal`, which is off by default. Turn it on when running your application:

    QT_LOGGING_RULES="qt.qml.binding.removal.info=true" ./myApplication

Or, have it permanently while developing, from your `main.cpp`:

    QLoggingCategory::setFilterRules(QStringLiteral("qt.qml.binding.removal.info=true"));

Every binding destroyed by an imperative assignment is then reported, pointing at the code responsible. It is noisy on a code base that was not written with this in mind, which is itself rather informative.
```


## Creating bindings at runtime

`Qt.binding()` is the general answer whenever a binding has to be created from JavaScript rather than written in the file. It takes a function, and returns something you assign exactly like a value:

```qml
Component.onCompleted: {
    box.width = Qt.binding(function() { return root.width / 3 })
}
```