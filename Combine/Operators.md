# Операторы издателей
В Combine методы, которые выполняют операцию со значениями, поступающими от издателя, называются операторами.

Операторы Combine могут выполнять кучу различных задач, которые мы могли бы отлично выполнить с помощью императивных команд, но, как мы традиционно делали с методами, позвольте мне предоставить вам каждый вид операторов, которые у нас будут:
- [Операторы преобразования](#операторы-преобразования): создайте нового издателя, который берет выходные данные из вышестоящей линии и преобразует их в новые значения для публикации из нижестоящей. Мы могли бы сказать, что длина нашего выхода одинакова
- [Операторы фильтрации](#операторы-фильтрации) filter: создайте нового издателя, который принимает только некоторые значения, опубликованные из вышестоящим потоком. Мы могли бы сказать, что длина нашего нисходящего потока меньше, чем длина нашего выше.
- [Комбинирование операторов](#операторы-комбинирования): взять двух или более издателей и выдавать некоторые значения на основе выходных данных вышестоящего издателя.
- Операторы синхронизации: отвечают за доставку значений от вышестоящим издателем только при проверке некоторых условий.
- Операторы добавления: создайте нового издателя, который выдает некоторые дополнительные выходы, помимо вышестоячей.
- [Операторы метаданных](#операторы-метаданных): возвращают вывод с некоторой информацией о вышестоящим издателе
- [Операторы планирования](#издатели-планирования): контролируйте время испускаемых значений из вышепо течению

Каждый оператор комбина фактически возвращает тип издателя. Вообще говоря, этот издатель получает вышестоящие значения, манипулирует данными, а затем отправляет эти данные ниже по течению. 

# Операторы преобразования
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

## Reduce
Подобно scan, но мы не публикуем никаких промежуточных значений. Вместо этого мы просто публикуем окончательную накопленную стоимость. Он публикуется только после завершения вышестоящего издателя.
```swift
[1, 2, 3].publisher
    .reduce(0, +)
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// 6
```

<img width="684" alt="Снимок экрана 2024-02-29 в 17 28 48" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/babe5961-5818-41ef-8780-443755cb0f6b">


# Операторы фильтрации

## `filter(_:)`
Переиздает все элементы, которые соответствуют выражению в замыкании.
Давайте отфильтруем издателя числе от 1 до 10 по кратности 2м:
```swift
var subscriptions = Set<AnyCancellable>()

(1...5).publisher
    .filter({ $0.isMultiple(of: 2) })
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// 2
// 4
```

<img width="690" alt="Снимок экрана 2024-03-01 в 08 42 40" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/c374032c-2deb-4aed-aa3f-e789ed45d73d">

## `removeDuplicates()`
Публикует только те элементы, которые не равны предыдущему элементу. Важно напомнить вам, он удаляет только последовательные дубликаты.

```swift
var subscriptions = Set<AnyCancellable>()

[1, 2, 3, 3, 3, 1].publisher
    .removeDuplicates()
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// 1
// 2
// 3
// 1
```

<img width="734" alt="Снимок экрана 2024-03-01 в 08 47 45" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/8cbd8947-81c2-4d63-8d09-c4efe9839058">

## `ignoreOutput()`
Этот оператор сбрасывает все публикующиеся значения от вышестоящего издателя, выдавая только свое событие завершения.

<img width="728" alt="Снимок экрана 2024-03-01 в 08 51 08" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/66fef243-ffc1-4e3f-ade3-840f32a2c2d1">

## `first()` и `first(where:)`
Этот оператор публикует только первое выходное значение из издателя выше, без условий или с условием в замыкании.

```swift
var subscriptions = Set<AnyCancellable>()

(1...3).publisher
    .first(where: { $0 == 1 })
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)

// 1
```

<img width="728" alt="Снимок экрана 2024-03-01 в 08 53 15" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/19741372-45b7-40d4-b90d-524af6f20b6b">

## `last()` и `last(where:)
Этот оператор публикует только последнее выходное значение из издателя выше, без условий или с условием в замыкании.

```swift
var subscriptions = Set<AnyCancellable>()

(1...3).publisher
    .last(where: { $0 == 3 })
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// 3
```
<img width="709" alt="Снимок экрана 2024-03-01 в 08 57 51" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/94ef7305-19aa-4bb8-b6f7-64eab94b381a">

## `dropFirst()` и вариации
### dropFirst(_ count:)
Опускает указанное количество элементов перед повторной публикацией последующих элементов.

### drop(while:)
Опустить элементы от вышестоящего издателя до тех пор, пока данное закрытие не вернет false, прежде чем переиздать все оставшиеся элементы.

### drop(untilOutputFrom:)
Отбрасывает все элементы из вышестоящего издателя, пока какой-то другой издатель не начнет выдавать значения.

<img width="705" alt="Снимок экрана 2024-03-01 в 09 09 21" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/ad3693e3-5a64-46a0-8143-ba92c614a8c7">

## `prefix(_:)`
Это полная противоположность от drop. Вместо того, чтобы отбрасывать первую часть опубликованных значений из вышепо течения, мы рассматриваем только их:

<img width="704" alt="Снимок экрана 2024-03-01 в 09 10 53" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/cc000766-8c24-49bc-b09a-a84a9a6983ca">

# Операторы комбинирования
Операторы делают то, чего не делали предыдущие: они объединяют несколько вышестоящих издателей в один нисходящий. Иногда у вас могут быть разные источники асинхронных данных, и вам действительно нужен способ объединения их результатов в одном потоке. Фраза, которая возобновляет идею: "Я просто хочу результатов, независимо от того, откуда они берутся".

## Zip
Этот оператор берет двух или более издателей (до 4х) и возвращает их выходные данные в виде кортежа, что означает, что новое значение публикуется нисходящими только в том случае, если все вышестоящие издатели уже опубликовали значение.

```swift
var subscriptions = Set<AnyCancellable>()

let publisherA = PassthroughSubject<String, Never>()
let publisherB = PassthroughSubject<String, Never>()

Publishers.Zip(publisherA, publisherB)
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)

publisherA.send("A") // Нет публикаций, так как нет публикации от второго издателя publisherB

publisherB.send("Z") // ("A", "Z")

publisherB.send("Y")  // Нет публикаций, так как нет публикации от первого издателя publisherA

publisherA.send("B")  // ("B", "Y")
```

<img width="702" alt="Zip" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/bde8a5f8-e354-4cc0-8cd9-8aa643063b9e">


## CombineLatest
Работает аналогично zip, но в этом случае нисходящий издатель публикует значение каждый раз, когда это делает и нисходящий. Полученный кортеж будет состоять из элемента, который был только что издан вместе со всеми последними элементами из других:

```swift
var subscriptions = Set<AnyCancellable>()

let publisherA = PassthroughSubject<String, Never>()
let publisherB = PassthroughSubject<String, Never>()

Publishers.CombineLatest(publisherA, publisherB)
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)

publisherA.send("A") // Нет публикаций, ждет хотя бы одно значение от publisherB

publisherB.send("Z") // ("A", "Z")

publisherB.send("Y")  // ("A", "Y")

publisherA.send("B")  // ("B", "Y")
```

<img width="776" alt="CombineLatest" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/829d6aa0-46a6-4425-a785-3ee2883ee367">


Пример использования: Представьте, что у вас есть издатели, которые отслеживают значения из textFields. Вы хотите объединить их все, чтобы получить некоторый вывод в виде кортежа всего содержимого из текстовых полей. Если вы измените один из них, вывод должен быть проверен с текущим контентом от других, так как мы заинтересованы в текущих значениях.

## Merge
Этот оператор объединяет издателей, которые должны иметь один и тот же тип вывода, выдавая все значения, которые поступает из трех источников и ведут себя как будто это один издатель. Всякий раз, когда некоторые из издателей выдают данные, это значение публикуется ниже.

```swift
var subscriptions = Set<AnyCancellable>()

let publisherA = PassthroughSubject<String, Never>()
let publisherB = PassthroughSubject<String, Never>()

Publishers.Merge(publisherA, publisherB)
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)

publisherA.send("A") // A

publisherB.send("Z") // Z

publisherB.send("Y")  // Y

publisherA.send("B")  // B
```

<img width="702" alt="Merge" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/e94ae57c-3d88-49c2-81e4-9fb29821c2f7">


## switchToLatest
Этот оператор в основном сглаживает издателя издателей, которым мы хотим отслеживать значения, опубликованные с последнего отправленного издателя. Взгляните на этот код:

```swift
var subscriptions = Set<AnyCancellable>()

let publisherA = PassthroughSubject<String, Never>()
let publisherB = PassthroughSubject<String, Never>()

let subjects = PassthroughSubject<PassthroughSubject<String, Never>, Never>()

subjects
    .switchToLatest()
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)

publisherA.send("A") // Нет вывода, так как издателей на самом деле нет в подписке
publisherB.send("Z") // Нет вывода, так как издателей на самом деле нет в подписке

subjects.send(publisherA) // Подключаем издателя publisherA

publisherA.send("B")  // B
publisherB.send("Y")  // Нет вывода, так как издателя publisherB нет в подписке subjects

subjects.send(publisherB) // Подключаем издателя publisherB

publisherB.send("X")  // X

publisherA.send("C")  // Нет вывода, так как мы переключились на издателя publisherB
```

<img width="947" alt="switchToLatest" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/b4f5ec60-535a-4f02-9498-ecdc7fa53ba5">


Пример использования: Представьте, что у вас есть панель вкладок с тремя экранами, каждый из которых соответствует вкладке. У каждого из этих viewControllers есть издатель, который продолжает выдавать значения, но мы хотим отслеживать только значения, которые были опубликованы выбранной вкладкой. Мы должны отобразить значения в небольшом оповещении перед UITabBarController. Наиболее подходящим решением было бы полагаться на издателя, который выдает издателя из соответствующего ViewController, как только он появляется, а затем через оператор switchLatest мы отслеживаем значения, которые исходят с выбранного экрана.

# Дополнительные издатели
## prepend
Добавляет начальное значение перед всеми опубликованными значениями
```swift
var subscriptions = Set<AnyCancellable>()

["A", "B", "C", "D", "E"].publisher
    .prepend("Алфавит")
    .sink(receiveValue: { value in
        print(value)
    })
    .store(in: &subscriptions)
// Алфавит
// A
// B
// C
// D
// E
```

<img width="691" alt="Снимок экрана 2024-03-01 в 09 57 13" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/5a17c275-1663-48a2-b389-e74a0aa2002b">

## append
Добавьте новые значения в конце восходящего потока после завершения.
```var subscriptions = Set<AnyCancellable>()

["A", "B", "C", "D", "E"].publisher
    .append("Алфавит")
    .sink { completion in
        print(completion)
    } receiveValue: { value in
        print(value)
    }
    .store(in: &subscriptions)
// A
// B
// C
// D
// E
// Алфавит
// finished
```

<img width="694" alt="Снимок экрана 2024-03-01 в 09 57 27" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/8b4929b7-af47-444d-b7b3-81bc7264fadd">

# Операторы метаданных
## Count
Возвращает, сколько значений было опубликовано от издателя выше но игнорируя их. Публикует своезначение значение только тогда, когда заканчивается вышепоток.
```swift
var subscriptions = Set<AnyCancellable>()

let alphabet = "abcdefghijklmnopqrstuvwxyz"

alphabet.publisher
    .count()
    .sink { completion in
        print(completion)
    } receiveValue: { value in
        print(value)
    }
    .store(in: &subscriptions)
// 26
// finished
```

<img width="671" alt="Снимок экрана 2024-03-01 в 10 04 02" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/85943c60-cba6-4d86-8016-6d911cf579a8">

## `output(at:)`
Публикует элемент в данной позиции, как только он существует по индексу:
```swift
var subscriptions = Set<AnyCancellable>()

let alphabet = "abcdefghijklmnopqrstuvwxyz"

alphabet.publisher
    .output(at: alphabet.count - 1) // хотим последний элемент
    .sink { completion in
        print(completion)
    } receiveValue: { value in
        print(value)
    }
    .store(in: &subscriptions)
// z
// finished
```

<img width="714" alt="Снимок экрана 2024-03-01 в 10 08 02" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/25c2fd2d-ed38-4474-b04b-f4e136ff870a">


## `contains(_:)`
Публикует bool вниз по потоку, указывающий, содержат ли значения восходящего потока какое-то конкретное значение или есть какое-то значение, следующее за каким-то конкретным condition(contains(where:))
```swift
var subscriptions = Set<AnyCancellable>()

let alphabet = "abcdefghijklmnopqrstuvwxyz"

alphabet.publisher
    .contains("w") // проверяем на наличие буквы "w"
    .sink { value in
        print(value)
    }
    .store(in: &subscriptions)
// true
```

## `allSatisfy`
Публикует логическое определение того, соблюдают ли все значения из вышепотока какого-либо условия. Публикуется только после завершения upstream.
Например сделаем подписку на проверку влидности, как только данные не проходят проверку то закрываем доступ:
```swift
var subscriptions = Set<AnyCancellable>()
let validation = PassthroughSubject<Bool, Never>()

validation
    .allSatisfy({ $0 })
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { isValid in
        print(isValid)
    })
    .store(in: &subscriptions)

validation.send(true) // Нет публикаций
validation.send(true) // Нет публикаций
validation.send(true) // Нет публикаций
validation.send(false) // false и завершение
```

<img width="702" alt="allSatisfy" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/c5b58ec9-84d1-4bef-9d02-6ae608c756a0">

# Издатели Планирования
## Задержка `delay(for:tolerance:scheduler:options:)`
Начинает публикацию значений из вышепотока (или события завершения) только после некоторой задержки
```swift
var subscriptions = Set<AnyCancellable>()
let subjectInt = PassthroughSubject<Int, Never>()

subjectInt
    .delay(for: .seconds(5), scheduler: RunLoop.main)
    .sink(receiveCompletion: { completion in
        print(completion)
    }, receiveValue: { isValid in
        print(isValid)
    })
    .store(in: &subscriptions)

subjectInt.send(1) // Отсрочка публикации на 5 секунд
subjectInt.send(2) // Публикует сразу 2
subjectInt.send(3) // Публикует сразу 3
```
<img width="699" alt="Снимок экрана 2024-03-01 в 10 38 21" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/301a6989-448f-4cd3-bd68-2d9b14ad8cce">

## debounce
Этот оператор применим, когда мы не хотим, чтобы события испускались в слишком близком промежутке времени. На этот раз давайте возьмем пример пользовательского интерфейса: представьте, что у вас есть сцена SwiftUI, где вы отображаете TextField вертикально выровненный с меткой Text. Мы полагаемся на класс ContentViewModel в качестве StateObject для нашего ContentView и двух опубликованных свойств внутри: строка firstName, привязку которой мы вводим в наш TextField, и метку, которая должна отображаться на тексте:


Этот оператор применим, когда мы не хотим, чтобы события испускались в слишком близком промежутке времени. На этот раз давайте возьмем пример пользовательского интерфейса: представьте, что у вас есть сцена SwiftUI, где вы отображаете TextField вертикально выровненный с меткой Text. Мы полагаемся на класс ContentViewModel в качестве StateObject для нашего ContentView и двух опубликованных свойств внутри: строка firstName, привязку которой мы вводим в наш TextField, и метку, которая должна отображаться на тексте:

```swift
import SwiftUI
import PlaygroundSupport
import Combine

final class ContentViewModel: ObservableObject {
    @Published var firstName: String = ""
    @Published var label: String = "Default value"
    
    init() {
        $firstName.assign(to: &$label)
    }
}

struct ContentView: View {
    @StateObject var viewModel = ContentViewModel()
    
    var body: some View {
        VStack {
            TextField("First Name", text: $viewModel.firstName)
                .textFieldStyle(.roundedBorder)
                .cornerRadius(4)
                .padding(.bottom, 16)
                .padding(.horizontal, 16)
            Text(viewModel.label)
                .font(.footnote)
                .multilineTextAlignment(.leading)
        }
        .padding()
    }
}

PlaygroundPage.current.setLiveView(ContentView())
```
Поскольку мы структурировали наш поток Combine, каждый раз, когда издатель, завернутый в firstName, выдает новую строку, она будет назначена нашему содержимому label, который также заворачивает другого издателя, который при обновлении отображает новый текст на нашей Text метке в нашей сцене SwiftUI.

![sync](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/314ea96b-bf8e-4043-9dd0-09e0fb6b1e0e)

Но в нашем приложении мы не хотим, чтобы все было полностью синхронизировано. Представьте, что мы хотим получить последний контент всего через 1 секунду. Для этого мы будем полагаться на debounce, чтобы дождаться надлежащего времени для публикации любых изменений:

```swift
final class ContentViewModel: ObservableObject {
    @Published var firstName: String = ""
    @Published var label: String = "Default value"
    
    init() {
        $firstName
            .debounce(for: .seconds(1), // отсрочка
                      scheduler: RunLoop.main)
            .assign(to: &$label)
    }
}
```
Вот наш результат:

![debounce](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/480e3356-dc8b-4a97-b237-0715b7357403)

Последние изменения публикуются сразу после одной секунды.

<img width="702" alt="debounce" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/da0f0cff-5875-43e5-b6f1-e896d0fc65fc">


## throttle
Этот оператор публикует данные вниз с отсрочкой для каждого элемента сверху. Например, если бы мы применили это в нашем предыдущем примере, наш издатель label выдавал бы одно событие в секунду, являясь первым элементом из восходящего потока или последним (вы можете определить). Если новое значение не публикуется, последнее значение переиздается в нижестоящем потоке:

```swift
final class ContentViewModel: ObservableObject {
    @Published var firstName: String = ""
    @Published var label: String = "Default value"
    
    init() {
        $firstName
            .throttle(for: .seconds(1), scheduler: RunLoop.main, latest: true)
            .assign(to: &$label)
    }
}
```

![throttle](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/f747c2fd-f7ad-458d-aa0b-9d7cf6b94c60)

<img width="702" alt="throttle" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/4b851cd4-be6c-4e97-a6f1-295774f31827">
