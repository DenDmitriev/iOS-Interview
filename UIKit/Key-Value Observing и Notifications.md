# Key-Value Observing и Notifications

## KVO
Позволяет объектам подписаться на изменение конкретных свойств объекта. При KVO - обозреватель в курсе об обозреваемом объекте

```swift
import UIKit

class MyObjectToObserve: NSObject {
    @objc dynamic var myDate = Date()
    func updateDate() {
        myDate = Date()
    }
}

class MyObserver: NSObject {
    @objc var objectToObserve: MyObjectToObserve
    var observation: NSKeyValueObservation?
    
    init(object: MyObjectToObserve) {
        objectToObserve = object
        super.init()
        
        observation = observe(\.objectToObserve.myDate) { object, change in
            print("Observed a change to \(object.objectToObserve).myDate, updated to: \(object.objectToObserve.myDate.description)")
        }
    }
}
 
let observed = MyObjectToObserve()
let observer = MyObserver(object: observed)
 
observed.updateDate()
// Observed a change to <__lldb_expr_302.MyObjectToObserve: 0x600002028020>.myDate, updated to: 2023-10-31 07:47:16 +0000
```

## Notifications
Определяет зависимость типа «один ко многим» между объектами таким об разом, что при изменении состояния одного объекта все зависящие от него оповещаются об этом. В iOS представлен в виде NotificationCenter. При Notifications - обозреватель в курсе лишь о названии нотификации, и абсолютно не имеет понятия кто ее отослал.

Исходя из определений используй, то что тебе нужно:
Если например тебе нужно в разных местах обрабатывать разлогин пользователя, то используй Notifications. 
Если тебе нужно отслеживать изменение имени в профайле, то KVO.
