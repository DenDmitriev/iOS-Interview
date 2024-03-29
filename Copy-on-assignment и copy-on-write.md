# Copy-on-write (COW)

Механизм в Swift, который копирует поведение ссылочных типов на типы значений до первых изменений. 

## Что нам это дает?
Это избавляет нашу программу от лишних копирований, тем самым улучшая производительность.

## Как работает?
Как известно основным отличием ссылочных типов от типов значений является то, что первые передаются по ссылкам, в то время как вторые копируются. У этого есть ряд своих плюсов таких как то, что типы значений работают быстрее, потому что хранятся на стеке, а не в куче, используют статическую диспетчеризацию и прочее. 
Однако можно задаться вопросом “а зачем копировать данные, если мы их не меняем?”. Действительно и COW как раз-таки отвечает на этот вопрос и говорит, что это не обязательно.

Проведем небольшой эксперимент:
```swift
func address(_ o: UnsafeRawPointer, _ name: String) {
  print(name, "address: \(Int(bitPattern: o))")
}

var arr1 = [1, 2, 3]
var arr2 = arr1

address(arr1, "arr1") // arr1 address: 105553156825120
address(arr2, "arr2") // arr2 address: 105553156825120
// Обе переменные ссылаются на одну область памяти, хотя мы и работаем с типом значений и данные первой переменной должны были бы копироваться во вторую.

// А теперь изменим второй массив и посмотрим что будет.
arr2.append(4)

address(arr1, "arr1")  // arr1 address: 105553156825120
address(arr2, "arr2") // arr2 address: 105553133600496
// Как можно заметить теперь ссылки ведут на разные области в памяти.
```

## Copy-on-write в кастномном типе

Существует вариант интеграции COW в собственный тип данных от [Apple GitHub](https://github.com/apple/swift/blob/658be5b40fe151cc87a49f5f24b2f20b79dd4813/docs/OptimizationTips.rst#advice-use-copy-on-write-semantics-for-large-values). 
В этом случае создается некий класс-враппер, являющийся оболочкой для хранимого значения.
```swift
final class Ref<T> {
  var val: T
  init(_ v: T) { val = v }
}

struct Box<T> {
  var ref: Ref<T>
  init(_ x: T) { ref = Ref(x) }

  var value: T {
    get { return ref.val }
    set {
      if !isKnownUniquelyReferenced(&ref) {
        ref = Ref(newValue)
        return
      }
      ref.val = newValue
    }
  }
}
```

# Copy-on-assignment (COA)
Механизм, который по умолчанию используется при копировании значений структур. Как только значение некоторой структуры копируется из одного параметра в другой, создается его полная копия, которая размещается в новой области памяти.

```swift
var source = 3
// Int - структура
// копируем экземпляр структуры
var copy = source

// выводим адреса в памяти
withUnsafePointer(to: &source) {
print($0) // текущий адрес, например 0x00000001155e5090
}

withUnsafePointer(to: &copy) {
print($0) // другое значение, например 0x00000001155e5098
}
```

## Источники
 - [Copy-on-write](https://habr.com/ru/articles/673372/)
 - [УПРАВЛЕНИЕ ПАМЯТЬЮ В SWIFT](https://swiftme.ru/upravlenie-pamyatyu-v-swift-8281/)
