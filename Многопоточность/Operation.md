# Operation
Более высокий уровень по сравнению с GCD. Есть возможность добавить зависимости, повторно использовать, отменить или приостановить. Состояние Operation можно контролировать. Можно указать максимальное количество операций в очереди, которые будут выполняться одновременно. Если нужно полностью контролировать процесс многопоточности, то используется Operation. Например нам нужно операции выстроить в цепочки.

## Очереди
 - На главном потоке `OperationQueue.main` уже существует
```swift
OperationQueue.main.addOperation {
    print("😇")
}
// 😇
```

 - Пользовательские `OperationQueue`
```swift
let operationQueue = OperationQueue()
operationQueue.addOperation {
    print("🙈")
}
// 🙈
```

### Тип очереди
Определение кол-ва операций параллельно указывается параметром `maxConcurrentOperationCount`. При указании параметра 1 мы получим последовательную очередь и паралллельную при большем числе. Если не указать параметр, то очередь будет параллельной.
- Последовательная
```swift
let operationQueue = OperationQueue()
operationQueue.maxConcurrentOperationCount = 1 // последовательно
let blockOperation1 = BlockOperation {
    print("blockOperation1: 😴")
    sleep(3)
    print("blockOperation1: 🤗")
}

blockOperation1.completionBlock = {
    print("blockOperation1 completed")
}

let blockOperation2 = BlockOperation {
    print("blockOperation2: 😴")
    sleep(3)
    print("blockOperation2: 🤗")
}

blockOperation2.completionBlock = {
    print("blockOperation1 completed")
}
operationQueue.addOperation(blockOperation1)
operationQueue.addOperation(blockOperation2)

// blockOperation1: 😴
// blockOperation1: 🤗
// blockOperation1 completed
// blockOperation2: 😴
// blockOperation2: 🤗
// blockOperation1 completed
```
- Параллельная
```swift
let operationQueue = OperationQueue()
operationQueue.maxConcurrentOperationCount = 10 // паралельно но не более 10 одновременно
let blockOperation1 = BlockOperation {
    print("blockOperation1: 😴")
    sleep(3)
    print("blockOperation1: 🤗")
}

blockOperation1.completionBlock = {
    print("blockOperation1 completed")
}

let blockOperation2 = BlockOperation {
    print("blockOperation2: 😴")
    sleep(3)
    print("blockOperation2: 🤗")
}

blockOperation2.completionBlock = {
    print("blockOperation1 completed")
}
operationQueue.addOperation(blockOperation1)
operationQueue.addOperation(blockOperation2)

// blockOperation1: 😴
// blockOperation2: 😴
// blockOperation1: 🤗
// blockOperation2: 🤗
// blockOperation1 completed
// blockOperation1 completed
```

## Operation
Ключевым отличием от GCD явялется переиспользование кода путем создания объекта `Operation`
Давайте создадим такой объект, который будет выдавать таблицу умнажения в виде массива числел.
```swift
final class MultiplierTableOperation: Operation {
    var input: Int // Принимает число
    var output: [Int]? // Результат перемножения
    var descriptions: [String]? // Выражение
    
    private let multipliers = [1,2,3,4,5,6,7,8,9]
    
    init(value: Int) {
        self.input = value
        super.init()
    }
    
    override func main() {
        descriptions = []
        output = multipliers.map { multiplier in
            let result = input * multiplier
            let formatted = "\(input) x \(multiplier) = \(result)"
            descriptions?.append(formatted)
            return result
        }
    }
}
```
Теперь у нас есть операция и мы можем создать массив операций с переиспользованием кода:
```swift
// Исходные данные для операций
let values = [1,2,3,4,5,6,7,8,9]

// Массив операций для чисел из массива
var operations = values.map { value in
    // Создаем операцию для числа
    let operation = MultiplierTableOperation(value: value)
    // Блок кода для результата таблицы умнажения
    operation.completionBlock = {
        print("Multiplier table for \(value)")
        operation.descriptions?.forEach { stringResult in
            print(stringResult, terminator: "\n")
        }
        print("\n")
    }
    return operation
}
```
Время создать очередь и запустить операции
```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1 // для того чтоб в консоль результаты пришли последовательно

// Добавим операции в очередь
queue.addOperations(operations, waitUntilFinished: false)
// Multiplier table for 1
// 1 x 1 = 1
// 1 x 2 = 2
// ... и так далее
// Multiplier table for 9
// ...
// 9 x 7 = 63
// 9 x 8 = 72
// 9 x 9 = 81
```
