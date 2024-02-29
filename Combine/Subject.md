# Subject

## Что такое Subject?
Издатель, который предоставляет метод для внешних вызовов для публикации элементов. Подробнее Subject - это издатель, которого вы можете использовать для "введения" значений в поток, используя его метод send(_:). Это может быть полезно для адаптации существующего императивного кода к модели Combine. 

Если оценивать его реализацию то Subject - это протокол, который соответсвует протоколу издателя Publisher так же как и Just, Future или другие дефолтные реализации издателя.

Давайте посмотрим на его содержание:
```swift
public protocol Subject<Output, Failure> : AnyObject, Publisher {

     /// Отправляет значение подписчику.
     ///
     /// - Parameter value: значение для отправки.
     func send(_ value: Self.Output)

     /// Отправляет подписчику сигнал завершения.
     ///
     /// - Parameter completion: экземпляр `Completion`, который указывает, завершилась ли публикация нормально или произошла ошибка.
     func send(completion: Subscribers.Completion<Self.Failure>)

     /// Отправляет подписку подписчику.
     ///
     /// Этот вызов предоставляет ``Субъекту`` возможность установить спрос на любые новые восходящие подписки.
     ///
     /// - Parameter subscription: экземпляр подписки, через который подписчик может запрашивать элементы.
     func send(subscription: Subscription)
}
```
Интерфейс протокола имеет возможности:
- `send(_:)` Отправляет значение абоненту.
- `send(completion:)` Посылает сигнал завершения подписчику.
- `send(subscription:)` Отправляет подписку подписчику.

Имеет два связанных типа: Output, Failure, которые наследуются от Publisher.

В Combine уже реализованы два типа subject, которые готовы к работе:
- PassThroughSubject, который выдает новые значения подписчикам без хранения.
- CurrentValueSubject, который хранит последнее значение и публикует его для подписчиков всякий раз, когда это значение меняется.

# Смотрим работу в коде
Добавьте этот новый пример на свой playground:
```swift
// Определите пользовательский тип ошибки.
enum MyError: Error {
    case test
}

// Определите пользовательского подписчика, который получает строки и ошибки MyError.
final class StringSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = MyError
    
    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }
    
    func receive(_ input: String) -> Subscribers.Demand {
        print("Received value", input)
        // Скорректируйте требование в зависимости от полученного текста.
        return input == "World" ? .max(1) : .none
    }
    
    func receive(completion: Subscribers.Completion<MyError>) {
        print("Received completion", completion)
    }
}

// Создайте экземпляр пользовательского подписчика.
let subscriber = StringSubscriber()
```

Возврат .max(1) в receive(_:), когда входной сигнал "World" приводит к тому, что новый максимум будет установлен в 3 (изначальный максимум плюс 1).

```swift
// Создает экземпляр PassthroughSubject типа String и пользовательского типа ошибки MyError, который вы определили.
let subject = PassthroughSubject<String, MyError>()

// Подписчик подписывается на subject.
subject.subscribe(subscriber)

// Создает еще одну подписку с помощью sink.
let subscription = subject
    .sink(
        receiveCompletion: { completion in
            print("Received completion (sink)", completion)
        },
        receiveValue: { value in
            print("Received value (sink)", value)
        }
    )
```

PassthroughSubject позволяют публиковать новые значения по требованию. Они с радостью передают данные и событие завершения. Как и в случае с любым издателем, вы должны заранее объявить тип данных и ошибок, которые он может выдавать; подписчики должны сопоставить эти типы с его типами ввода и сбоев, чтобы подписаться на этот Subject.

Теперь, когда вы PassthroughSubject, которая может отправлять значения и подписки для их получения, пришло время отправить некоторые значения. Добавьте следующий код к вашему примеру:
```swift
subject.send("Hello")
subject.send("World")
```
Запустив playground, вы увидите:
```swift
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
```
Каждый подписчик получает значения по мере их публикации.

Добавьте следующий код:
```swift
// Отмените вторую подписку.
subscription.cancel()

// Отправьте другое значение
subject.send("Still there?")
```
Как вы и ожидали, только первый подписчик получает значение. Это происходит потому, что вы ранее отменили подписку второго подписчика sink:
```swift
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
Received value Still there?
```

Добавьте этот код к примеру:
```swift
subject.send(completion: .finished)
subject.send("How about another one?")
```

Запустим playground. Второй подписчик не получает "How about another one?" значение, потому что он получил событие завершения непосредственно перед отправкой этого значения. Первый подписчик не получает событие завершения или значение, потому что его подписка была ранее отменена.

```swift
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
Received value Still there?
Received completion finished
```
Добавьте следующий код непосредственно перед строкой, которая отправляет событие завершения.
```swift
subject.send(completion: .failure(MyError.test))
```

Запустив playground, вы увидите следующее, напечатанное на консоли:

```swift
Received value (sink) Hello
Received value Hello
Received value (sink) World
Received value World
Received value Still there?
Received completion failure(__lldb_expr_534.MyError.test)
```

Ошибка получена первым подписчиком. Но событие завершения, которое было отправлено после ошибки, не приходит. Это показывает, что как только издатель отправляет одно событие завершения - будь то нормальное завершение или ошибка - это понимается, как завершение подписки! 

Передача значений с помощью PassthroughSubject является одним из способов связать императивный код с декларативным миром Combine. Иногда, однако, вы также можете захотеть посмотреть на текущее значение издателя в вашем императивном коде - для этого у вас есть уметко названная тема: CurrentValueSubject.

Вместо того, чтобы хранить каждую подписку в качестве значения, вы можете хранить несколько подписок в коллекции `AnyCancellable`. Затем коллекция автоматически отменит каждую добавленную к ней подписку, когда коллекция будет деинициализирована.

Добавьте этот новый пример в playground:
```swift
// Создайте коллекцию subscriptions.
var subscriptions = Set<AnyCancellable>()

// Создайте издателя CurrentValueSubject типа Int и Never.
// Он будет публиковать целые числа и никогда не будет публиковать ошибку.
// CurrentValueSubject требует начальное значение, зададим значение 0.
let subject = CurrentValueSubject<Int, Never>(0)

// Создайте подписку на издателя CurrentValueSubject и распечатайте значения, полученные от него.
subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
     // Храните подписку в наборе подписок, который передается в качестве параметра inout,
     // чтобы тот же набор обновлялся вместо копии.
```

Вы должны инициализировать текущие значения с начальным значением; новые подписчики немедленно получают это значение или последнее значение, опубликованное этим субъектом. Запустите playground, чтобы увидеть это в действии:
```swift
// 0
```

Теперь добавьте этот код, чтобы отправить два новых значения:

```swift
subject.send(1)
subject.send(2)
// 1
// 2
```

В отличие от PassthroughSubject субъекта, вы можете в любое время получить текущее значения издателя. Добавьте следующий код, чтобы распечатать текущее значение темы:

```swift
print(subject.value)
```
Как вы могли сделать вывод из имени CurrentValueSubject, вы можете получить его текущее значение, получив доступ к его свойству value. Запустите игровую площадку, и вы увидите 2, напечатанные во второй раз.
```swift
// 2
```
Вызов send(_:) на субъекте текущего значения является одним из способов отправки нового значения. Другой способ - назначить новое значение его свойству `value`. Ого, мы просто пошли все императивно здесь или как? Добавьте этот код:
```swift
subject.value = 3
print(subject.value)
```
Запустите playground. Вы увидите 3 выведенное дважды - один раз получателем подписчиком и один раз после печати значения темы после добавления этого значения к теме.

Затем, в конце этого примера, создайте новую подписку на текущую тему значения:
```swift
subject
  .sink(receiveValue: { print("Second subscription:", $0) })
  .store(in: &subscriptions)
```
Здесь вы создаете подписку и печатаете полученные значения. Вы также храните эту подписку в наборе subscriptions.

## Как завершить подписку?
### Автоматически
Вы прочитали минуту назад, что набор подписок автоматически отменит добавленные к нему подписки, но как вы можете это проверить? Вы можете использовать оператор print(), который будет регистрировать все события публикации в консоли.

Поместите код в блок кода `Task`. Вставьте оператор print() в обе подписки, между темой и раковиной. Начало каждой подписки должно выглядеть следующим образом:
```swift
Task {
// код инициализации подписок и подписка
subject
  .print()
  .sink...
```
Запустите playground еще раз, и вы увидите следующий вывод для всего примера:
```swift
receive subscription: (CurrentValueSubject)
request unlimited
receive value: (0)
0
receive value: (1)
1
receive value: (2)
2
2
receive value: (3)
3
3
receive subscription: (CurrentValueSubject)
request unlimited
receive value: (3)
Second subscription: 3
receive cancel
receive cancel
```
Каждое событие печатается вместе со значениями, напечатанными в обработчиках подписки, и когда вы печатаете значения темы. События отмены получения печатаются, потому что набор подписок был определен в рамках этого примера, поэтому он отменяет подписки, которые он содержит при выхода из блока контекста функции Task.

### Вручную
Итак, вам может быть интересно, можете ли вы также назначить событие завершения свойству value? Попробуйте, добавив этот код:
```swift
subject.value = .finished
```
Нет! Это приводит к ошибке. Свойство значения CurrentValueSubject предназначено именно для этого: значений. События завершения по-прежнему должны быть отправлены с помощью send(_:). Измените ошибочную строку кода на следующую:
```swift
subject.send(completion: .finished)
```
 На этот раз вы увидите следующий вывод внизу:
```swift
receive finished
receive finished
```
 Обе подписки получают событие завершения вместо события отмены. Они закончены и больше не нуждаются в отмене.
