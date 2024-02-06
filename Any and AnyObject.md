# Any и AnyObject

- `Any` — может представлять экземпляр любого типа, используется для всех типов
- `AnyObject` — может представлять экземпляр любого класса, используется для типов Class

Проверим на деле эти два типа. Начнем с типа Any.

Для этого создадим массив с типом Any, и распечатаем его.

```swift
let anyArray: [Any] = ["Macbook", 1, 2]
print(anyArray)
// ["Macbook", 1, 2]
```

Как мы видим, Any позволяет работать с различными типами данных одновременно (String, Int).
Согласно документации, элементы (String и Int) в этом массиве являются структурами, которые являются типами значений, поэтому, теоретически, AnyObject не должен работать.

Чтобы проверить это, создадим идентичный массив, с типом AnyObject.
```swift
let anyObjectArray: [AnyObject] = ["Macbook", 1, 2] // Error: Type of expression is ambiguous without a type annotation
print(anyObjectArray)
/// Cannot convert value of type «Int» to expected element type «AnyObject»
/// Cannot convert value of type «Int» to expected element type «AnyObject»
/// Cannot convert value of type «String» to expected element type «AnyObject»
```

## Когда же использовать AnyObject?
Как говорится в документации Apple, AnyObject может быть использован для работы с объектами, которые являются производными от Class, но не имеют общего корневого класса.
Таким образом, AnyObject желательно использовать когда вы хотите ограничить протокол, чтобы его можно было использовать лишь с классами, а Any в остальных случаях.

Apple добавляет:
Используйте Any и AnyObject только тогда, когда вам явно нужно поведение и возможности, которые они предоставляют. Всегда лучше быть точным в отношении типов, которые вы ожидаете использовать в своем коде.

## Источники
- [Any и AnyObject в Swift. В чем их различие?](https://habr.com/ru/articles/483494/)
