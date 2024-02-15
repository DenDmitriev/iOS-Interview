# Непрозрачные типы (Opaque Types)
Непрозрачные типы (opaque types) являются важной особенностью языка Swift. С помощью ключевого слова some, обозначающего непрозрачный тип, вы можете скрыть конкретный тип возвращаемого значения для вычисляемого свойства или функции. И это позволит вам писать гибкий, лаконичный и надежный код. То есть непрозрачный тип подразумевает, когда вызывающему абоненту не нужно указывать базовый тип возврата или тип параметра, и вызывающий абонент должен точно получить конкретный тип.

Давайте посмотрим на различные типы сценариев использования, с помощью которых мы можем решить проблемы с помощью непрозрачатого типа возврата:

## Пример использования 1: Требования к самостоятельному или связанному типу 👇
```swift
protocol Product {
    associatedtype Kind
    var kind: Kind { get }
}

struct Mobile: Product {
    typealias Kind = Int
    var kind: Kind
    let name: String
}

struct TV: Product {
    typealias Kind = String
    var kind: Kind
    let name: String
}

func product() -> Product { // Use of protocol 'Product' as a type must be written 'any Product'
    Mobile(kind: 1, name: "iPhone")
}
```
Хотя Mobile подтверждает протокол Product, вы не можете установить Product в качестве типа возврата из-за его общего поведения, он требует особого обращения. Посмотрим, как мы можем решить проблему 👇

Решение:
```swift
func product() -> some Product {
    Mobile(kind: 1, name: "iPhone")
}
```

Здесь `some Product` подразумевают, что product() вернет конкретный тип, который должен подтвердить протокол Product. Именно это и является обратной общей концепцией.

Но функция не может возвращать несколько типов, если возвращаемым типом является OTR.
```swift
enum ProductType {
    case mobile, tv
}

func product(type: ProductType) -> some Product {
    switch type {
    case .mobile:
        Mobile(kind: 1, name: "iPhone") // Branches have mismatching types 'Mobile' and 'TV'
    case .tv:
        TV(kind: "LED", name: "Samsung")
    }
}
```

Использование непрозрачного типа возврата очень специфично, как вы можете видеть.

но мы можем решить это следующим образом

```swift
func product(type: ProductType) -> any Product {
    switch type {
    case .mobile:
        Mobile(kind: 1, name: "iPhone")
    case .tv:
        TV(kind: "LED", name: "Samsung")
    }
}
```

или спомощью универсальных шаблонов
```swift
func product<T: Product>(type: ProductType) -> T {
    // ...
}
```

Значит непрозрачные типы позволяют не укзывать связанный тип возвращаемого протокола в функции, если возрвщаемы объект реализует этот протокол.
