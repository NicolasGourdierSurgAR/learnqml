## Exercice 1
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