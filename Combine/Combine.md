# Combine
Это основа Apple для реактивного программирования в 2019 году. Это позволяет нам получать данные с течением времени, которые могут работать в качестве альтернативы шаблону делегатов или обратным вызовам в блоках замыкания.

В структуре Combine есть три основные концепции, которые нам нужно понять, прежде чем погрузиться в нее:
1. Издатель или Publisher
2. Подписчик или Subscriber
3. Подписка или Subscription

## Издатель или Publisher
Это протокол, который определяет связанный типа значений, которые будут публиковаться с течением времени и связанный тип ошибки для отправки сбоев.

> Связанные типы - это интерфейс издателя, который должен соответствовать подписчику для создания подписки.

Взгляните на протокол Publisher и одно из его наиболее важных расширений:
```swift
public protocol Publisher<Output, Failure> {
  // Тип значений, которые может производить издатель.
  associatedtype Output

  // Тип ошибки, которую может вызвать издатель, или Never, если издатель гарантированно не выдает ошибку.
  associatedtype Failure : Error

  // Реализация subscribe(_:) вызовет receive(subscriber:), чтобы привязать подписчика к издателю, т.е. создать подписку.
  func receive<S>(subscriber: S)
    where S: Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}

extension Publisher {
  // Подписчик вызывает subscribe(_:) у издателя, чтобы прикрепить к нему.
  public func subscribe<S>(_ subscriber: S)
    where S : Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}
```

Протокол имеет два связанных типа (associated types), которые должны быть определены:
- Output: Тип значений, которые будут отправлены. Это может быть String, Int, пользовательская структура и т. д.
- Failure: Тип ошибки, который мы хотим использовать, когда мы хотим выпустить событие сбоя.

## Подписчик или Subscriber
Это протокол, который определяет связанный типа значений, которые будут приходить с течением времени и связанный тип ошибки для получения сбоев.
Или простыми словами подписчик может получать события, которые издает издатель, и выполнять задачу, необходимую для этих событий.

```swift
public protocol Subscriber<Input, Failure>: CustomCombineIdentifierConvertible {
  // Тип значений, которые может получить подписчик.
  associatedtype Input

  // Тип ошибки, которую может получить подписчик; или Never, если подписчик не получит ошибку.
  associatedtype Failure: Error

  // Издатель вызывает receive(subscription:) на подписчика, чтобы дать ему подписку.
  func receive(subscription: Subscription)

  // Издатель вызывает receive(_:) на подписчика, чтобы отправить ему новое значение, которое он только что опубликовал.
  func receive(_ input: Self.Input) -> Subscribers.Demand

  // Издатель вызывает receive(completion:) подписчику, чтобы сообщить ему, что он закончил производить значения, либо из-за окончания, либо из-за ошибки.
  func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

## Подписка или Subscription
Это протокол, который связывает издателя и подписчика. Вот протокол Subscription:
```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
  func request(_ demand: Subscribers.Demand)
}
```
Методом request(_:) указывает, что подписчик готов получать значения с ограничениями. Например тут подписчик готов получить до трех значений при подписке:
```swift
subscription.request(.max(3))
```

Протокол соответсвует другому протоколу `Cancellable`
```swift
public protocol Cancellable {
  func cancel()
}
```
Метод `cancel()` требуется реалзиовать остановку публкации чтоб завершить подписку и разорвать сильные ссылки.

### AnyCancellable
AnyCancellable является классом который реализует протокол `Cancellable`. Используется для управления жизненным циклом подписок издателей, а именно имеет возможность их завершить методом `cancel()`.

## Как это работает
События, которые может опубликовать издатель для подпичсика: 
- значения
- ошибка
- завершение

<img width="654" alt="Снимок экрана 2024-02-27 в 16 52 09" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/4d96c7ec-c0bd-4cba-84e0-eed8b14d0477">

Прогуливаясь по этой диаграмме:
1. Подписчик подписывается на издателя.
2. Издатель создает подписку и предоставляет ее подписчику.
3. Подписчик запрашивает значения.
4. Издатель отправляет значения.
5. Издатель отправляет завершение.

## Операторы
Существуют методы, которые могут преобразовывать значения, которые выдает издатель. Мы можем связать столько операторов, сколько захотим. Например, если мы делаем сетевой запрос и получаем ошибку 500, мы можем зафиксировать эту ошибку, преобразовать ее в пользовательский тип ошибки и отправить это значение.
Вы можете проверить полный список операторов, которые есть у Combine, [здесь](https://developer.apple.com/documentation/combine/anypublisher-publisher-operators/).

## Объединим это в коде
Теперь, когда мы рассмотрели основы, давайте перейдем к некоторым примерам кода, чтобы увидеть, как на самом деле работает Combine.

Так как Publisher это всего лишь протокол нам нужен объект который реализует этот протокол. Мы можем воспользоваться некоторыми из встроенных издателей, которые предоставляет Combine, вместо того, чтобы создавать свои собственные. Давайте посмотрим на наиболее распространенные.
Мы будем использовать тип `AnyPublisher`, который является реализацией протокола Publisher.

### Just (Только)
Структура Just создает простого издателя, который выдаст одно значение для всех подписчиков, а затем завершит подписку.

<img width="291" alt="Снимок экрана 2024-02-27 в 17 37 43" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/b0014cfc-1d4d-4f56-b0f3-679a06ae4052">

```swift
// Издатель
let just = Just("Hello world!")

// Подписка
let cancellable: AnyCancellable?

cancellable = just
    .sink( // Подпиcание на издателя
        receiveCompletion: {
            print("Received completion", $0)
        },
        receiveValue: {
            print("Received value", $0)
        })
// Received value Hello world!
// Received completion finished
```
Метод `sink` выступает как подписчик и возвращает `AnyCancellable` и мы храним её в свойстве `cancellable` чтоб подписка существовала. 

### Empty (Пустой)
Структура Empty является еще одним удобным заполнителям издателем. Он немедленно завершается без излучения каких-либо значений.

<img width="291" alt="Снимок экрана 2024-02-27 в 18 09 23" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/3c98fa20-68c1-4ce3-8269-c8515f66283b">

```swift
let empty = Empty<String, Never>()

let cancellable: AnyCancellable?

cancellable = empty
    .sink(
    receiveCompletion: {
        print("Received completion", $0)
    },
    receiveValue: {
        print("Received value", $0)
    })
// Received completion finished
```

### Fail или Неудача
Издатель Fail немедленно отключит поток данных с указанной ошибкой. В основном он используется для тестирования и отладки.

```swift
let fail = Fail<Int, Error>(error: ErrorDomain.example)

let cancellable: AnyCancellable?

cancellable = fail
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("Received completion")
            case .failure(let failure):
                print("Received completion with error: \(failure.localizedDescription)")
            }
        },
        receiveValue: {
            print("Received value", $0)
        })
// Received completion with error: 👾 Example error
```
