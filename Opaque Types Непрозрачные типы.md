# Непрозрачные типы (Opaque Types)

Непрозрачный тип подразумевает, когда вызывающему абоненту не нужно указывать базовый тип возврата или тип параметра, и вызывающий абонент должен точно получить конкретный тип.

Давайте посмотрим на различные типы сценариев использования, с помощью которых мы можем решить проблемы с помощью непрозрачатого типа возврата:

Пример использования 1: Требования к самостоятельному или связанному типу 👇
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

