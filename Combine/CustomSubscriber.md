# Создание подписчика

Давайте создадим подписчика для получения целочисленных данных:
```swift
// Создайте издателя целых чисел с помощью свойства publisher для диапазона.
let publisher = (1...6).publisher

// Определите пользовательского подписчика, IntSubscriber
final class IntSubscriber: Subscriber {
    // Реализуйте псевдонимы типа, чтобы указать, что этот подписчик может получать
    // целочисленные данные и никогда не будет получать ошибки.
    typealias Input = Int
    typealias Failure = Never
    
    // Реализуйте необходимые методы, начиная с receive(subscription:),
    // который вызывается издателем; и в этом методе вызовите .request(_:) в подписке (subscription),
    // указав, что подписчик готов получить до трех значений при подписке.
    func receive(subscription: Subscription) {
        subscription.request(.max(3))
    }
    
    // Распечатайте каждое значение по мере его получения и верните .none,
    // указывая на то, что абонент не будет корректировать свое требование;
    // .none эквивалентно .max(0).
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received value", input)
        return .none
    }
    
    // Распечатайте событие завершения.
    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion", completion)
    }
}
```

Чтобы издатель опубликовал что-либо, ему нужен подписчик. Добавьте следующее в конце примера:

```swift
// В этом коде вы создаете подписчика, который соответствует типам Output и Failure издателя.
let subscriber = IntSubscriber()

// затем вы говорите издателю прикрепить подписчика
publisher.subscribe(subscriber)

// Управляя playground, вы увидите следующее, напечатанное на консоли:
// Received value 1
// Received value 2
// Received value 3
```

Вы не получили событие завершения. Это связано с тем, что издатель имеет конечное количество значений, и вы указали требование .max(3).
В вашем пользовательском receive(_:) вашего подписчика попробуйте изменить .none на .unlimited, чтобы ваш метод receive(_:) выглядел следующим образом:

```swift
func receive(_ input: Int) -> Subscribers.Demand {
  print("Received value", input)
  return .unlimited
}
```

Снова запустим playground. На этот раз вы увидите, что все значения получены и распечатаны вместе с событием завершения:

```swift
Received value 1
Received value 2
Received value 3
Received value 4
Received value 5
Received value 6
Received completion finished
```

Попробуйте изменить .unlimited на .max(1) и снова запустите playground.

```swift
Received value 1
Received value 2
Received value 3
Received value 4
Received value 5
Received value 6
Received completion finished
```

Вы увидите тот же вывод, что и при возврате .unlimited, потому что каждый раз, когда вы получаете событие, вы указываете, что хотите увеличить максимум на 1.

Измените .max(1) обратно на .none и вместо этого измените определение издателя на массив строк. Смотрим:

```swift
```

Запустим playground. Вы получаете ошибку, что метод подписки требует, чтобы типы String и IntSubscriber.Input (т.е. Int) были эквивалентны. Вы получаете эту ошибку, потому что типы, связанные с выводом и сбоями издателя, должны соответствовать типам ввода и сбоя подписчика, чтобы создать подписку между ними.
