# Signals and signal handlers

Properties describe *what an object is right now*: its width, its color, its text. Signals describe *what just happened to it*: a button was clicked, an animation finished, a value changed.

A **signal** is simply a notification that an object emits. On its own it does nothing at all: it is emitted, and if nobody is listening, nothing happens. To react to a signal you attach a **signal handler** to it, which is a piece of JavaScript that QML runs every time the signal is emitted.

## Reacting to a signal

The **Button** type from **QtQuick.Controls** emits a `clicked` signal every time the user clicks it. To react to it, we write a handler named `onClicked`:

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

    Button {
        anchors.centerIn: parent
        text: "Click me"

        onClicked: console.log("The button was clicked")
    }
}
```

Run it, click the button a few times, and watch the console of your application:

```text
qml: The button was clicked
qml: The button was clicked
qml: The button was clicked
```

`console.log()` is your first debugging tool in QML, and you will use it a lot.

The handler name is not something you choose: it is derived from the signal name by capitalizing its first letter and prefixing it with `on`. A `clicked` signal is handled by `onClicked`, a `pressAndHold` signal by `onPressAndHold`, a `valueChanged` signal by `onValueChanged`.

```admonish note "A handler is written like a property"
A handler is written with the same `name: value` syntax as a property. Its value can be a single expression, as above, or a whole block of JavaScript:

    onClicked: {
        console.log("The button was clicked")
        root.color = "#303030"
    }
```

## Declaring your own signal

You are not limited to the signals Qt provides. Any object can declare its own, with the `signal` keyword, and emit it by calling it like a function:

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

    // The declaration
    signal somethingHappened()

    // The handler
    onSomethingHappened: console.log("Something happened!")

    Button {
        anchors.centerIn: parent
        text: "Click me"

        // Emitting the signal is just calling it
        onClicked: root.somethingHappened()
    }
}
```

This looks like a pointless detour here, since the button could have logged the message itself. It stops being one as soon as your application grows: a signal lets an object announce that something happened **without knowing who cares**. The button does not need to know what happens next, and whoever reacts does not need to know which button emitted the signal. This is what keeps QML files from turning into a tangle of objects reaching into each other, and it is the reason custom components almost always expose their behavior as signals.

## Signals with parameters

A signal can carry values along with it. Declare them between the parentheses, with the name first and the type after a colon:

```qml
signal positionPicked(x: int, y: int)
```

To use those values, the handler must be written as a **function** that declares the parameters it wants. Two syntaxes are available for this, and they are equivalent:

```qml
// Arrow function
onPositionPicked: (x, y) => console.log("Picked position:", x, y)

// Anonymous function
onPositionPicked: function(x, y) { console.log("Picked position:", x, y) }
```

Both do exactly the same thing. Pick the one you find the most readable and try to stay consistent within a file.

Here is a complete example using one of each. It also introduces a **MouseArea**: an invisible item that covers a region of the window and reports what the mouse does over it. It is the simplest way to make anything in QML react to the mouse, and it emits a `clicked` signal of its own, carrying a `mouse` parameter that holds the position of the click. We will come back to it in detail in a later chapter, so for now just take it as a rectangle that tells you where it was clicked.

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

    signal positionPicked(x: int, y: int)

    // Anonymous function syntax
    onPositionPicked: function(x, y) {
        console.log("Picked position:", x, y)
    }

    MouseArea {
        anchors.fill: parent

        // Arrow syntax, unpacking the mouse parameter carried by clicked
        onClicked: (mouse) => root.positionPicked(mouse.x, mouse.y)
    }
}
```

Click anywhere in the window and you get:

```text
qml: Picked position: 143 87
```

The names of the parameters in your handler do not have to match the ones in the signal declaration: only their order matters. You may also drop the trailing parameters you do not need, though you cannot skip the leading ones: if you only want the second parameter, name the first one and ignore it.

```admonish warning "Qt 6: always declare the parameters of your handler"
In Qt 5, the parameters of a signal were *injected* into the handler, meaning you could use them directly as if they were variables appearing out of thin air:

    // Qt 5 style, do not write this
    onPositionPicked: console.log("Picked position:", x, y)

Qt 6 still accepts this form for backwards compatibility, but it is **deprecated** and will print a runtime warning as soon as an injected parameter is actually used. Always write the handler as a function that declares its parameters, using either of the two syntaxes above.

You will meet the old form constantly in tutorials, forum answers and older code bases, so it is worth recognizing it, and worth converting it whenever you touch it.
```

## Property change signals

Here is the part that makes QML so powerful: **every property automatically comes with its own signal**, emitted whenever its value changes. The signal is named after the property, followed by `Changed`, so the handler for a property `counter` is `onCounterChanged`.

You do not declare any of this. It is there for the properties you define yourself and for every property Qt provides:

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

    property int counter: 0

    // Our own property
    onCounterChanged: console.log("counter is now", root.counter)

    // A property we did not declare
    onWidthChanged: console.log("the window is now", root.width, "pixels wide")

    Button {
        anchors.centerIn: parent
        text: "Increment"

        onClicked: root.counter++
    }
}
```

Click the button, then resize the window, and both handlers report:

```text
qml: counter is now 1
qml: counter is now 2
qml: the window is now 412 pixels wide
qml: the window is now 427 pixels wide
```

Note that the signal is emitted when the value **changes**, not when it is assigned. Writing `root.counter = 0` while the counter already holds `0` emits nothing.

## Bonus: the Connections object

A handler must be declared inside the object that emits the signal. Most of the time that is exactly what you want, but not always: the object may be declared far away in the file, it may come from C++, or you may want to swap at runtime which object you are listening to.

**Connections** solves this. You give it a `target` and it hosts the handlers on its behalf:

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

    Button {
        id: button

        anchors.centerIn: parent
        text: "Click me"
    }

    Connections {
        target: button

        function onClicked() {
            console.log("Handled from outside the Button")
        }
    }
}
```

The two properties of **Connections** you will use most are:

- `target`: the object emitting the signals. If you leave it unset, it defaults to the parent of the **Connections**. Setting it to `null` disconnects everything, which is handy when the object you listen to appears and disappears.
- `enabled`: `true` by default, set it to `false` to temporarily stop reacting.

```admonish warning "Use the function syntax"
Inside a **Connections**, handlers must be declared with the `function` keyword:

    function onClicked() { ... }

The old form, written like a normal handler with a colon, is still accepted for backwards compatibility, but it is deprecated and Qt prints a warning when it meets it:

    onClicked: { ... } // do not write this inside a Connections

The two are easy to confuse because the colon form is the correct one *everywhere else*. The rule of thumb: inside a **Connections**, use `function`; directly inside the emitting object, use the colon.
```

```admonish tip "Connecting from JavaScript"
For completeness: from JavaScript, a signal can also be used as an object, with a `connect()` and a `disconnect()` method. It lets you wire things up at runtime, from inside a function:

    root.somethingHappened.connect(myFunction)
    root.somethingHappened.disconnect(myFunction)

You will need it far less often than handlers and **Connections**, but it is the right tool when the connection itself depends on something you only know while the application runs.
```

```admonish tip "Time to practice"
Signals are much easier to feel than to read about, and `console.log()` makes them visible. It is again a good time to experiment a bit. Here are a few things you can try: put a **Button** in a window and log its `clicked`, `pressed` and `released` signals to see in which order they arrive. Add a `property int counter` with an `onCounterChanged` handler and increment it from the button. Then add a second button that sets the counter to `5`, and check that clicking it twice in a row only logs once. Declare a signal of your own carrying a string, emit it from the button with different values, and log it from a **Connections** instead of a handler.
```
