# Actor

## Что такое Data Race?
Гонка данных происходит, когда два или более потоков в одном процессе одновременно обращаются к одному и тому же местоположению памяти, и по крайней мере один из доступов используется для записи, и потоки не используют никаких эксклюзивных замков для управления своим доступом к этой памяти.

# Что такое Race Condition?
Условие гонки возникает, когда два потока одновременно получают доступ к общей переменной. Первый поток читает переменную, а второй поток считывает то же значение из переменной. Это означает, что условие гонки возникает, когда время или порядок событий влияет на правильность фрагмента кода.

```swift
class Account {
    var balance: Double
    var transactions: [String] = []
    
    init(balance: Double) {
        self.balance = balance
    }
    
    func withdraw(amount: Double) {
        
        if balance >= amount {
            let processingTime = UInt32.random(in: 0...3)
            transactions.append("[Withdraw] Processing for \(amount) \(processingTime) seconds")
            sleep(processingTime)
            transactions.append("Withdrawing \(amount) from account")
            
            self.balance -= amount
            transactions.append("Balance is \(balance)")
        }
    }
}
```

```swift
var account= Account(balance: 500)
DispatchQueue.concurrentPerform(iterations: 2) { _ in 
    self.bankAccountVM.withdraw(500)
}
// Balance is 0.0
// Balance is -500.0
```

В многопоточном приложении несколько потоков вызывают различные выводы средств для одной и той же учетной записи. Заказ одних и тех же двух выводов непредсказуем и может привести к двум различным результатам, основанным на порядке исполнения, который заканчивается более странными результатами балансов. Синхронизация не существует.

До Swift актер решил это с помощью механизма синхронизации, который может быть DispatchQueue, DispatchSemaphore и Lock.
```swift
// using DispatchQueue
let lockQueue = DispatchQueue(label: "my.serial.lock.queue")
DispatchQueue.concurrentPerform(iterations: 2) { _ in
    lockQueue.sync {
        self.bankAccountVM.withdraw(500)
    }
}
// using DispatchSemaphore
let semaphore = DispatchSemaphore(value: 1)
DispatchQueue.concurrentPerform(iterations: 2) { _ in
    semaphore.wait()
    self.bankAccountVM.withdraw(500)
    semaphore.signal()
}
// using lock
var lock = os_unfair_lock_s()
DispatchQueue.concurrentPerform(iterations: 2) { _ in
    os_unfair_lock_lock(&lock)
    self.bankAccountVM.withdraw(500)
    os_unfair_lock_unlock(&lock)
}
```

# Actor
Actor - это конкретный номинальный тип в Swift для решения проблемы синхронизации данных. Они являются эталонными типами и могут иметь методы и свойства, а также могут реализовывать протоколы, но они не поддерживают наследование, что делает их инициализаторы намного проще - нет необходимости в удобных `convenience initializers`, `overriding`, `final` ключевом слове.

Актеры создаются с использованием ключевого слова new actor. Они могут защитить внутреннее состояние с помощью изоляции данных, гарантируя, что только один поток будет иметь доступ к базовой структуре данных в данный момент времени. Все субъекты неявно соответствуют новому протоколу Actor; ни один другой конкретный тип не может использовать его. Актеры решают проблему data race, вводя изоляцию актера. Сохраненные свойства вообще не могут быть записаны из-за пределов объекта актера.

```swift
enum ARBankError: Error {
    case insufficientFunds(Double)
}
actor ARBankAccount {
    let accountNumber: Int
    var balance: Double
    var details: String {
        "Account holder Number: \(accountNumber)"
    }
    
    init(accountNumber: Int, balance: Double) {
        self.accountNumber = accountNumber
        self.balance = balance
    }
    
    func deposit(_ amount: Double) async {
        balance += amount
    }
    
    func transfer(amount: Double, to other: ARBankAccount) async throws  {
        if amount > balance {
            throw ARBankError.insufficientFunds(amount)
        }
        balance -= amount
        await other.deposit(amount)
        
        print("Account: \(balance), Other Account: \(await other.balance)")
    }
}
```

> Если вы пишете код внутри метода актера, вы можете прочитать другие свойства этого актера и вызвать его синхронные методы без использования await, но если вы пытаетесь использовать эти данные извне актера await требуется даже для синхронных свойств и методов.

По умолчанию каждый метод актера становится изолированным, что означает, что вам уже придется быть в контексте актера или использовать ожидание, чтобы дождаться утвержденного доступа к данным, содержащимся в актерере. Методы Actor изолированы по умолчанию, но явно не помечены.

## Параметры актера изолированы

Использование изолированного ключевого слова для параметров может быть использовано с меньшим количеством кода для решения конкретных задач. В приведенном выше примере кода был введен метод внесения депозита для изменения баланса другого банковского счета.

Мы могли бы избавиться от этого дополнительного метода, отметив параметр как изолированный и обновив баланс другого банковского счета.
```swift

func transfer(amount: Double, to other: isolated ARBankAccount) async throws {
    if amount > balance {
        throw ARBankError.insufficientFunds(amount)
    }
    balance -= amount
    other.balance += amount
    print(“Account: \(balance), Other Account: \(other.balance)”)
}
```

## Неизолированное ключевое слово в actor
Маркирование методов или свойств как неизолированных может быть использовано для отказа от изоляции актеров по умолчанию. Отказ может быть полезен в случаях доступа к неизменяемым значениям или при соблюдении требований протокола.

1. Доступ к неизменяемым значениям из вычисляемого свойства

Номер счета неизменяем, поэтому доступ безопасен из неизолированной среды. Компилятор достаточно умен, чтобы распознать это состояние, поэтому нет необходимости помечать этот параметр как неизолированный явно.

Однако, если вычисляемое свойство получает доступ к неизменяемому свойству, оно покажет ошибку компиляции, как показано ниже.
![1*sRYr-MVIN6ktQVaX2ccxTQ](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/f6ac8838-53a0-4eea-8f22-9d5e896c4246)

Здесь мы должны явно пометить его как неизолированный, чтобы удалить ошибку.

```swift
actor ARBankAccount {
    let accountNumber: Int
    var balance: Double
    nonisolated var details: String {
        "Account holder Number: \(accountNumber)"
    }
    init(accountNumber: Int, balance: Double) {
        self.accountNumber = accountNumber
        self.balance = balance
    }
}
```

2. Соответствие протокола неизолированным

Добавление соответствия протоколу, в котором мы уверены, что сможем получить доступ только к неизменяемому состоянию. Ниже приведен пример, где мы могли бы заменить свойство details протоколом CustomStringConvertible.

![1*rvlZTI8YYZI_z5m0cGZsCA](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/6139b8b5-23ae-4ae9-b1ec-4c82621d3302)

Здесь мы можем решить это снова, явно используя ключевое слово nonisolated.

```swift

extension ARBankAccount: CustomStringConvertible {
    nonisolated var description: String {
        "Account holder number: \(accountNumber)"
    }
}
```

## MainActor
MainActor - это глобальный актер, который позволяет нам выполнять код в основной очереди. Используя ключевое слово @MainActor global actor, вы можете использовать для маркировки свойств и методов, к которым должен быть доступ только в основном потоке. Это глобальная обертка актера вокруг базовой структуры MainActor, что полезно, потому что она имеет статический метод run(), который позволяет нам планировать работу для запуска.
```swift

import Foundation

class ARController {
    func save() -> Bool {
        guard Thread.isMainThread else {
            print("Func Save is not in main thread...")
            return false
        }
        print("Saving old data...")
        return true
    }
}

class AROtherController {
    @MainActor func save() {
        print("Saving new data...")
    }
}

let obj = ARController()
DispatchQueue.concurrentPerform(iterations: 2) { _ in
    obj.save()
}
// Saving old data...
// Func Save is not in main thread...


let newObj = AROtherController()
await newObj.save()
// Saving new data...
```
> Написание свойств извне актера не допускается, с ожиданием или без него.

Вы можете проверить демо-проект на [GitHub](https://github.com/ashokrwt31/Actor-POC) для Swift Actor.


# Источники
- [How does Swift Actor prevent data race?](https://medium.com/@ashokrwt/how-does-swift-actor-prevent-data-race-b6f484e7eb8c)
