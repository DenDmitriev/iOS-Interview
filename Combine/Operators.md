# Операторы издателей
В Combine методы, которые выполняют операцию со значениями, поступающими от издателя, называются операторами.

Каждый оператор комбина фактически возвращает тип издателя. Вообще говоря, этот издатель получает вышестоящие значения, манипулирует данными, а затем отправляет эти данные ниже по течению. 

## `collect()`
Оператор сбора предоставляет удобный способ преобразования потока отдельных значений от издателя в массив этих значений.

<img width="728" alt="Снимок экрана 2024-02-29 в 15 32 04" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/8b9181c7-5746-4df9-8429-beec807f30b5">

Как показано на этой мраморной диаграмме, collect будет буферизовать поток отдельных значений в массив этих значений, как только вышестоящему издателю завершит. Затем он будет излучать этот массив вниз по течению.

Посомтрим этот пример:
```swift
var subscriptions = Set<AnyCancellable>()

["A", "B", "C", "D", "E"].publisher
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
```
Каждое значение выдается и печатается индивидуально, за которым следует завершение:
```swift
// A
// B
// C
// D
// E
// finished
```
Теперь вставьте использование сбора перед раковиной. Ваш код должен выглядеть следующим образом:
```swift
["A", "B", "C", "D", "E"].publisher
    .collect()
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
```

sink получает один выпущенный собранный массив, за которым следует завершение::
```swift
["A", "B", "C", "D", "E"]
finished
```
> Примечание: Будьте осторожны при работе с collect() и другими операторами буферизации, которые не требуют указания количества или лимита. Они будут использовать неограниченный объем памяти для хранения полученных значений.

Замените collect() строку:
```swift
.collect(2)
```
Консоль выведет:
```swift
["A", "B"]
["C", "D"]
["E"]
finished
```

## Семейство map

### `map(_:)`
Оператор преобразования значений издателя.
Первое, о чем вы узнаете, это карта, которая работает так же, как стандартный `map` Swift, за исключением того, что она работает на значениях, выпущенных издателем. На диаграмме map принимает замыкание, которое умножает каждое значение на 2.

<img width="749" alt="Снимок экрана 2024-02-29 в 15 41 16" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/93017122-f3f7-42f9-af9a-fa0d453dd14f">

Добавьте этот новый пример в playground:
```swift
var subscriptions = Set<AnyCancellable>()

let formatter = NumberFormatter()
formatter.numberStyle = .spellOut
formatter.locale = Locale(identifier: "RU-ru")

[123, 4, 56].publisher
    .map { formatter.string(from: NSNumber(integerLiteral: $0)) ?? "" }
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

Используйте map, передавая замыкание, которое получает значения свыше от издателя и возвращает результат использования форматора для возврата прописанных чисел.

```swift
сто двадцать три
четыре
пятьдесят шесть
```

### `map<T>(_:)`

Семейство операторов map также включает в себя три версии, которые могут сопоставляться в одно, два или три свойства значения с помощью ключевых путей. Их подписи следующие:
- `map<T>(_:)` Оператор трансформирует данный в значение по ключевому пути.
- `map<T0, T1>(_:_:)` Оператор, который публикует значения двух ключевых путей в виде кортежа.
- `map<T0, T1, T2>(_:_:_:)`  Оператор, который публикует значения трех ключевых путей в виде кортежа.

`T` представляет тип значений, найденных на заданных ключевых путях.

В следующем примере вы будете использовать оператор
```swift
let publisher = PassthroughSubject<CGSize, Never>()

publisher
    .map(\.width, \.height) // Сопоставьте свойства width и height, используя их ключевые пути.
    .sink(receiveValue: { width, height in
        let aspectRatio = ((width / height) * 100).rounded() / 100
        print("Aspect ratio = \(aspectRatio)")
    })
    .store(in: &subscriptions)

publisher.send(CGSize(width: 1280, height: 720))
publisher.send(CGSize(width: 400, height: 300))
```
В этом примере вы используете версию map, которая сопоставляется в два свойства с помощью ключевых путей.

Консоль выведет:
```swift
Aspect ratio = 1.78
Aspect ratio = 1.33
```

### `tryMap(_:)`
Несколько операторов, включая map, имеют аналоговый оператор try, который будет принимать замыкание, которое может выдавать ошибку. Если вы выдаете ошибку, она будет выдавать эту ошибку ниже по течению. Добавьте этот пример в playground:
```swift
var subscriptions = Set<AnyCancellable>()

Just("Папка с таким именем не существует")
    .tryMap { try FileManager.default.contentsOfDirectory(atPath: $0) }
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
    .store(in: &subscriptions)
```

Обратите внимание, что вам все еще нужно использовать ключевое слово try при вызове метода map.

Запустите playground и обратите внимание, что `tryMap` выводит событие завершения сбоя с соответствующей ошибкой "папка не существует" (вывод сокращенно):
```swift
failure(Error Domain=NSCocoaErrorDomain Code=260 "The folder “Папка с таким именем не существует” doesn’t exist."
```

### `flatMap(maxPublishers:_:)`
Преобразует все элементы от вышестоявшего издателя в нового издателя до максимального количества указанных вами издателей.
```swift
var subscriptions = Set<AnyCancellable>()

public struct ServerStatus {
    public let code: Int
}

var serverPublisher = PassthroughSubject<ServerStatus, URLError>()

serverPublisher
    .flatMap { status -> URLSession.DataTaskPublisher in
        let url = URL(string:"https://http.cat/\(status.code)")!
        return URLSession.shared.dataTaskPublisher(for: url)
    }
    .map(\.data)
    .sink { completion in
        print(completion)
    } receiveValue: { data in
        guard let image = NSImage(data: data) else { return }
        // code to update some view
    }
    .store(in: &subscriptions)


serverPublisher.send(ServerStatus(code: 100))
serverPublisher.send(ServerStatus(code: 200))
serverPublisher.send(ServerStatus(code: 403))
```
<img width="174" alt="Снимок экрана 2024-02-29 в 16 46 17" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/9d5bf98a-ce09-4c7b-927f-208bb3a1a1ae">
<img width="174" alt="Снимок экрана 2024-02-29 в 16 46 36" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/70e8e2f2-cf0d-419c-a6bf-b727ec3c11df">
<img width="174" alt="Снимок экрана 2024-02-29 в 16 46 48" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/fa1bc911-642a-4a9d-9ccd-a9ccb117359b">

Вспомните определение из предыдущего: flatMap сглаживает выходные данные всех полученных издателей в одного издателя. Это может создать проблему с памятью, потому что это будет буферизировать столько издателей, сколько вы отправите его для обновления одного издателя, который он излучает ниже по течению.

Чтобы понять, как управлять этим, взгляните на эту мраморную диаграмму flatMap:

<img width="725" alt="Снимок экрана 2024-02-29 в 16 49 27" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/c76f0cf1-4a5e-4031-94e2-492874feee11">

На диаграмме flatMap получает трех издателей: P1, P2 и P3. Каждый из этих издателей имеет свойство value, которое также является издателем. flatMap выдает значения издателей из P1 и P2, но игнорирует P3, потому что maxPublishers установлен в 2.

## `replaceNil(with:)`
Как показано на диаграмме, replaceNil получит необязательные значения и заменит nilы указанным вами значением:

<img width="732" alt="Снимок экрана 2024-02-29 в 16 51 10" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/69ea2d50-051d-4422-96dd-bcb52b432f1c">


Ранее в примере карты вы работали с методом Foundation Formatter.string(for:). Он создает необязательную строку, и вы использовали оператор нуль-коалессинга (??) Заменить значение nil на значение, не значаемое nol. Combine также включает в себя оператор, который вы можете использовать, когда хотите всегда предоставлять ценность.

Используйте replaceNil(with:), чтобы заменить значения nil, полученные от вышестоящего издателя, новым значением, не относящимся к нулю.
```swift
var subscriptions = Set<AnyCancellable>()

["A", nil, "C"].publisher
    .eraseToAnyPublisher()
    .replaceNil(with: "-")
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
// A
// -
// C
```

## `scan(_:_:)`
scan предоставит текущее значение, испускаемое вышестоящим издателем издателей для замыкыания, вместе с последним значением, возвращаемым этим замыканием.

На следующей диаграмме сканирование начинается с сохранения начального значения 0. По мере того, как он получает каждое значение от издателя, он добавляет его к ранее сохраненному значению, а затем сохраняет и выдает результат:

<img width="741" alt="Снимок экрана 2024-02-29 в 16 57 51" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/c9160721-2d59-41ff-9c6c-360fd0067428">

```swift
var subscriptions = Set<AnyCancellable>()

[1, 2, 3].publisher
    .scan(0) { latest, current in
        latest + current
    }
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// 1
// 3
// 6
```

## Фильтрующие Операторы
