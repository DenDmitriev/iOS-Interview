# subscribe or receive
Операторы для изменения планировщика, на котором выполняются последующие операторы в Combine.

# Планировщик
Объект для управления временем и местом исполнения операторов. Такими планировщиками являются RunLoop.main и DispatchQueue.main.

## `receive(on:)`
Указывает для планировщика подписки, на каком потоке получать элементы от издателя.

Apple пишет:
> Вы используете оператор `receive(on:)` для получения результатов и завершения в определенном планировщике, например, при выполнении работы пользовательского интерфейса в основном цикле выполнения. В отличие от `subscribe(on:)`, которая влияет на восходящие сообщения, `receive(on:)` изменяет контекст выполнения последующих публикаций.

В отличие от `receive(on:options:)`, который влияет на нисходящие публикации, `subscribe(on:options:)` изменяет контекст выполнения вышестоящих публикаций.

## `subscribe(on:)`
`subscribe(on:)` Указывает планировщику поток, на котором выполнять операции подписки, отмены и запроса. Другими словами, `subscribe(on:)` устанавливает планировщик для подписки на вышестоящий поток.

## Проверим это в работе
Давайте создадим подписку которая публикует числа в прописном виде:
```swift
var subscriptions = Set<AnyCancellable>()

let formatter = NumberFormatter()
formatter.numberStyle = .spellOut
formatter.locale = Locale(identifier: "RU-ru")

(1...3).publisher
    .map({ value in
        print(value, Thread.current)
        return formatter.string(from: NSNumber(integerLiteral: value)) ?? ""
    })
    .sink { isValid in
        print(isValid, Thread.current)
    }
    .store(in: &subscriptions)
```

Консоль выведет:
```swift
1 <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
2 <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
3 <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
один <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
два <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
три <_NSMainThread: 0x6000028fc8c0>{number = 1, name = main}
```

Видим что все операции выполняются на главном потоке.
Теперь добавим оператора `subscribe(on:)` чтоб обрабатывать подписку на глобальном потоке:
```swift
(1...3).publisher
    .subscribe(on: DispatchQueue.global())
    .map({ value in
        print(value, Thread.current)
        return formatter.string(from: NSNumber(integerLiteral: value)) ?? ""
    })
    .sink { isValid in
        print(isValid, Thread.current)
    }
    .store(in: &subscriptions)
```
Смотим консоль:
```swift
1 <NSThread: 0x6000013b5840>{number = 2, name = (null)}
один <NSThread: 0x6000013b5840>{number = 2, name = (null)}
2 <NSThread: 0x6000013b5840>{number = 2, name = (null)}
два <NSThread: 0x6000013b5840>{number = 2, name = (null)}
3 <NSThread: 0x6000013b5840>{number = 2, name = (null)}
три <NSThread: 0x6000013b5840>{number = 2, name = (null)}
```
Видим что все операции выполняются на глобальном потоке. Но мы хотим выводить аолученные значения на главном. Используем оператора `receive(on:)`:
```swift
(1...3).publisher
    .subscribe(on: DispatchQueue.global())
    .map({ value in
        print(value, Thread.current)
        return formatter.string(from: NSNumber(integerLiteral: value)) ?? ""
    })
    .receive(on: DispatchQueue.main)
    .sink { isValid in
        print(isValid, Thread.current)
    }
    .store(in: &subscriptions)
```
Снова смотрим консоль:
```swift
1 <NSThread: 0x600001948a80>{number = 3, name = (null)}
2 <NSThread: 0x600001948a80>{number = 3, name = (null)}
один <_NSMainThread: 0x60000197c8c0>{number = 1, name = main}
3 <NSThread: 0x600001948a80>{number = 3, name = (null)}
два <_NSMainThread: 0x60000197c8c0>{number = 1, name = main}
три <_NSMainThread: 0x60000197c8c0>{number = 1, name = main}
```
Теперь мы разделили логику. Операции подписки и обработки значений обрабатываются на глобальном потоке а операции по публикации подписки на главном. Это даже видно по очередности вывода в консоль, результат "один" вывелся раньше чем издатель успел опубликовать третье значение.

Подведем итог на диаграмме:

<img width="702" alt="subscribe or receive" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/7f1584c1-0f01-4ace-84d2-969416ca3206">
