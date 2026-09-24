# Property binding

We have already used bindings several times (every anchor is one, for instance), without looking at them closely. It is time to look at them properly, because they are the single idea that makes QML what it is.

## A binding is not an assignment

Here are two lines that look almost identical:

```qml
width: 200            // a value
width: parent.width   // a binding
```

The first one sets the width to 200, once and for all. The second one does **not** set the width to whatever `parent.width` happens to be at that moment. It establishes a *relationship*: from now on, this item is as wide as its parent. Resize the parent, and the child follows.

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

Here, the rectangle always keeps half the width and the full height of the window.

Any property can be bound, not just sizes, and the expression can be anything that produces a value of the right type. For example, assuming `root` declares a `property int messageCount`:

```qml
color: root.messageCount > 0 ? "steelblue" : "gray"
text: "You have " + root.messageCount + " messages"
visible: root.messageCount > 0
```

## How it works: change signals

You already know the mechanism from the [previous page](./signals_and_signal_handlers.md).

When you write a binding, the engine evaluates the expression once and watches which properties it reads along the way. For each of them it connects to the corresponding change signal. When any of those signals fires, the expression is evaluated again and the result is written to the property.

A binding is, in essence, a set of signal handlers that QML writes and maintains for you. `myVar: root.width + root.height` is roughly "connect to `root.widthChanged` and `root.heightChanged`, and on every emission recompute `root.width + root.height` and assign it".

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

## Prefer a binding to a handler

It is always possible to get the same result with a change handler. It is almost always worse:

```qml
// In root: works, but reinvents what QML already does
onWidthChanged: label.width = root.width / 2

// In label: says it once, declaratively
width: root.width / 2
```

The difference is not about typing less. It is about being able to answer the question *"why does this property have this value?"*.

With a binding, the answer is always in the same place: the line where the property is declared.

With handlers, a property can be assigned from anywhere: a change handler, a button, a timer, a function called by something else; and the only way to find out what set it is to search the entire file, and the rest of the project.

## Binding loops

Since a binding re-evaluates when its dependencies change, and since evaluating it changes a property, it is entirely possible to build a cycle.

```qml
Rectangle {
    id: outer

    width: inner.width + 20   // outer wants to be wider than inner

    Rectangle {
        id: inner

        width: outer.width + 20   // and inner wants to be wider than outer
    }
}
```

Each change triggers the other binding, which changes the first property again, forever. Qt notices this one and prints a message at runtime:

```text
QML Rectangle: Binding loop detected for property "width"
```

Then it breaks the cycle, leaving the property at whatever value it had reached. Your application keeps running, which is precisely why these are easy to ignore until something looks subtly wrong.

```admonish warning "Loops are rarely this obvious"
The two-line version above is the textbook case. Real ones hide much better, often spread across several items or files. We will not detail them here, to avoid digging into advanced concepts too early.
```

## Breaking a binding

A common source of errors is breaking a binding without noticing.

A property holds *either* a binding *or* a value. The moment you assign a value to a bound property from JavaScript (in a signal handler or a function, for instance), **the binding is destroyed**, permanently.

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

This is the mechanism behind a whole family of bugs that can be hard to track down. For now, just keep in mind that any imperative assignment may silently replace a binding.

```admonish tip "Log broken bindings automatically"
Qt can tell you every time this happens. There is a logging category for it, `qt.qml.binding.removal`, which is off by default. Turn it on when running your application:

    QT_LOGGING_RULES="qt.qml.binding.removal.info=true" ./myApplication

Or, have it permanently while developing, from your `main.cpp`:

    QLoggingCategory::setFilterRules(QStringLiteral("qt.qml.binding.removal.info=true"));

Every binding destroyed by an imperative assignment is then reported, pointing at the code responsible. It is noisy on a code base that was not written with this in mind, which is itself rather informative.
```


## Creating bindings at runtime

What if you *want* to set a binding from JavaScript, for example to restore one you broke, or to switch a property to a different relationship? `Qt.binding()` is the answer. It takes a function, and returns something you assign exactly like a value:

```qml
Button {
    text: "Follow a third of the window"

    onClicked: box.width = Qt.binding(function() { return root.width / 3 })
}
```

After clicking this button, `box.width` is bound again: resize the window and the rectangle follows, this time at a third of its width. The function is re-evaluated exactly like a binding written in the file, with the same dependency tracking.