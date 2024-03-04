# Combine
Это основа Apple для реактивного программирования созданная в 2019 году. Это позволяет нам получать данные с течением времени, которые могут работать в качестве альтернативы шаблону делегатов или обратным вызовам в блоках замыкания.

В структуре Combine есть три основные концепции, которые нам нужно понять, прежде чем погрузиться в нее:
1. [Издатель или Publisher](#издатель-или-publisher)
2. [Подписчик (Subscriber) который имеет подписку (Subscription) на издателя](#подписчик-или-subscriber)
3. [Операторы](#операторы)

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

### Подписка или Subscription
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
`sink` это метод экземпляра протокола Publisher. Метод `sink` выступает как подписчик и возвращает `AnyCancellable` и мы храним её в свойстве `cancellable` чтоб подписка существовала. 

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

### Fail (Неудача)
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
### Будущее (Future)
Это класс, который может выдавать или не выдавать значение и заканчиваться или выдавать ошибку. Это полезно для асинхронного кода, например, для вызовов API. Нам нужно выполнить обещание (Promise) с каким-то результатом (Result<Output, Error>), он может быть успешным или неудачным. Если обещание выполнено, это обещание отправляется абоненту.

Например сделаем асинхронный запрос для получения изображений кошек:
```swift
enum ErrorCat: Error, LocalizedError {
    case error(description: String)
    
    var errorDescription: String? {
        switch self {
        case .error(let description):
            description
        }
    }
}

struct Cat: Decodable {
    let url: URL
}

let future = Future<[Cat], ErrorCat> { promise in
    let request = URLRequest(url: URL(string: "https://api.thecatapi.com/v1/images/search?limit=3")!)
    Task {
        do {
            let (data, _) = try await URLSession.shared.data(for: request)
            let cats = try JSONDecoder().decode([Cat].self, from: data)
            promise(.success(cats))
        } catch {
            promise(.failure(.error(description: error.localizedDescription)))
        }
    }
}

let cancellable: AnyCancellable?

cancellable = future
    .sink(receiveCompletion: { completion in
        switch completion {
        case .finished:
            print("Finished")
        case .failure(let failure):
            print("Finished with failure: \(failure.localizedDescription)")
        }
    }, receiveValue: { cats in
        cats.forEach { cat in
            print(cat.url)
        }
    })
// Консоль:
// Если не будет интернета то: Finished with failure: A server with the specified hostname could not be found.
// В случае удачи:
// https://cdn2.thecatapi.com/images/28j.jpg
// https://cdn2.thecatapi.com/images/4h4.gif
// https://cdn2.thecatapi.com/images/5bv.jpg
// ...
// Finished
```

### Издатель из последовательности
Массив данных можно рассматривать как последовательность событий с течением времени. Поэтому последовательность является отличным источником для простого издателя.
`Sequence` имеет расширение со свойством `publisher` c типом `Publishers.Sequence<Self, Never>`которе объеденяет элементы последовательности в поток публикующихся данных.
```swift
extension Sequence {
    public var publisher: Publishers.Sequence<Self, Never> { get }
}
```

Представьте, что нам нужен поток данных, который излучает целые числа с течением времени. Выпущенные данные могут выглядеть примерно так.

<img width="513" alt="Снимок экрана 2024-02-28 в 09 00 56" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/fd0a7e3a-1dc4-4fc7-8145-aea91aada5ea">

Array соответствует Sequence. Последовательность имеет удобное свойство, называемое издателем `publisher`, которое создает издателя, который излучает элементы из исходной последовательности. Давайте создадим издателя используя это свойства и попробуем сравнить с издателем создвнным через `Publishers.Sequence<Elements, Failure>`

```swift
let sequence = [1, 2, 3, 4, 5].publisher
let publishers = Publishers.Sequence<[Int], Never>(sequence: [1, 2, 3, 4, 5])
print(type(of: sequence)) // Sequence<Array<Int>, Never>
print(type(of: publishers)) // Sequence<Array<Int>, Never>
```
Типы издателей оказались идентичными. То есть свойство `publisher` у последовательности это быстрый способ создания издателя. Теперь подпишемся на издателя:
```swift
sequence
    .sink { value in
        print(value)
    }
// 1
// 2
// 3
// 4
// 5
```
или чтобы вы могли управлять памятью и жизненным циклом подписок издателей. Вы можете хранить ссылку на раковину вот так.
```swift
let cancellable: AnyCancellable?
cancellable = sequence
    .sink { value in
        print(value)
    }
// 1
// 2
// 3
// 4
// 5
```

### Предметы (Subjects)
Это подкласс протокола Publisher, который может выдавать значения по требованию подписчикам, вызывая метод send(_:) В Combine уже реализованы два типа subject, которые готовы к работе:
- PassThroughSubject, который выдает новые значения подписчикам без хранения.
- CurrentValueSubject, который хранит последнее значение и публикует его для подписчиков всякий раз, когда это значение меняется.

Представьте, что мы работаем над приложением, чтобы увидеть результаты в прямом эфире и объявления о футбольном матче.
```swift
import Combine
import Foundation

enum LiveScoreError: Error, LocalizedError {
    case badWeather
    
    public var errorDescription: String? {
        switch self {
        case .badWeather:
            return "Игра была приостановлена из-за плохих погодных условий"
        }
    }
}

struct LiveScore<T: Subject> where T.Failure == LiveScoreError {
    
    public var publisher: T
    
    public init(publisher: T) {
        self.publisher = publisher
    }
    
    /// Отправка новости
    public func sendAnnouncement(_ announcement: T.Output) {
        publisher.send(announcement)
    }
    
    /// Завершение матча
    public func matchEnded() {
        publisher.send(completion: .finished)
    }
    
    /// Приостановка матча
    public func matchSuspended(withReason reason: LiveScoreError) {
        publisher.send(completion: .failure(reason))
    }
}
```
Довольно простая конструкция, которая построена с определенной темой. Как только мы создадим экземпляр, мы сможем выпустить новую новость, вызвав метод `sendAnnouncement`. Кроме того, мы можем сказать подписчикам, что подписка закончилась, или мы можем выпустить пользовательскую ошибку, если что-то было не так, используя методы `matchEnded` и `matchSuspended`.

#### PassThroughSubject (Пройти через предмет)
Что в основном делает PassThroughSubject, так это выдает новые значения подписчикам без хранения какого-либо состояния или самих значений.

```swift
let passThroughSubject = PassthroughSubject<String, LiveScoreError>()
let liveScore = LiveScore(publisher: passThroughSubject)
liveScore.publisher
    .sink { completion in
        switch completion {
        case .finished:
            print("Матч завершен!")
        case .failure(let failure):
            print(failure.localizedDescription)
        }
    } receiveValue: { news in
        print(news)
    }

liveScore.sendAnnouncement("12' - ⚽️ Манчестер Юнайтед (Роналду)")
liveScore.sendAnnouncement("15' - 🟨 Тоттенхэм (Эрик Дайер)")
liveScore.sendAnnouncement("34' - 🟨 Манчестер Юнайтед (Сантос)")
liveScore.sendAnnouncement("35' - ⚽️ Тоттенхэм (Кин)")
liveScore.sendAnnouncement("38' - ⚽️ Манчестер Юнайтед (Роналду)")
liveScore.sendAnnouncement("Половина тайма")
liveScore.sendAnnouncement("72' - ⚽️ Тоттенхэм (Магуайр - Автогол)")
liveScore.sendAnnouncement("81' - ⚽️ Манчестер Юнайтед (Роналду)")
liveScore.sendAnnouncement("85' - 🟨 Манчестер Юнайтед (Погба)")
liveScore.matchEnded()

// 12' - ⚽️ Манчестер Юнайтед (Роналду)
// 15' - 🟨 Тоттенхэм (Эрик Дайер)
// 34' - 🟨 Манчестер Юнайтед (Сантос)
// 35' - ⚽️ Тоттенхэм (Кин)
// 38' - ⚽️ Манчестер Юнайтед (Роналду)
// Половина тайма
// 72' - ⚽️ Тоттенхэм (Магуайр - Автогол)
// 81' - ⚽️ Манчестер Юнайтед (Роналду)
// 85' - 🟨 Манчестер Юнайтед (Погба)
// Матч завершен!
```

#### CurrentValueSubject (Текущее значение предмета)
Это очень похоже на PassThroughSubject с ключевой разницей, CurrentValueSubject хранит фактическое значение и публикует его для подписчиков всякий раз, когда это значение меняется. Поскольку он хранит текущее значение, любой новый подписчик получит это значение в момент подписки.

Другое отличие заключается в том, что CurrentValueSubject имеет начальное значение, с которым нам нужно инициализировать Subject.

```swift
let currentValueSubject = CurrentValueSubject<String, LiveScoreError>("Матч начался!")
let liveScore = LiveScore(publisher: currentValueSubject)
liveScore.publisher
    .sink { completion in
        switch completion {
        case .finished:
            print("Матч завершен!")
        case .failure(let failure):
            print(failure.localizedDescription)
        }
    } receiveValue: { news in
        print(news)
    }

liveScore.sendAnnouncement("1' - ⚽️ Реал Мадрид (Бензема)")
liveScore.sendAnnouncement("15' - 🟨 Атл. Мадрид (Суарес)")
liveScore.sendAnnouncement("36' - 🟥 Атл. Мадрид (Корреа)")
liveScore.sendAnnouncement("Половина тайма")
liveScore.matchSuspended(withReason: .badWeather)

// Матч начался!
// 1' - ⚽️ Реал Мадрид (Бензема)
// 15' - 🟨 Атл. Мадрид (Суарес)
// 36' - 🟥 Атл. Мадрид (Корреа)
// Половина тайма
// Игра была приостановлена из-за плохих погодных условий
```


### @Published (Обвертка публикатора)
Используя обертку @Published в любом свойстве, Combine автоматически создаст Publisher для этого свойства, и он будет выдавать значение всякий раз, когда значение свойства изменится. Мы можем получить доступ к издателю с помощью оператора $.
Единственное ограничение, которое у нас есть, заключается в том, что @Published можно использовать в классах, а не в структурах.

```swift
class TrafficLight {
    public enum Light: String {
        case green = "🟢"
        case yellow = "🟡"
        case red = "🔴"
    }

    @Published public var currentLight: Light

    public init(light: Light) {
        self.currentLight = light
    }
}

let trafficLight = TrafficLight(light: .red)
trafficLight.$currentLight
    .sink { newLight in
        print(newLight.rawValue)
    }

trafficLight.currentLight = .green
trafficLight.currentLight = .yellow
trafficLight.currentLight = .red

// 🔴
// 🟢
// 🟡
// 🔴
```

## Источники:
- [Introduction to Combine framework in Swift](https://blorenzop.medium.com/introduction-to-combine-framework-in-swift-4e50ccd6afe2)
- [Swift Combine Publishers: An Overview](https://www.mikegopsill.com/posts/combine-publishers/)
- [Publishers & Subscribers](https://www.kodeco.com/books/combine-asynchronous-programming-with-swift/v2.0/chapters/2-publishers-subscribers)
