# Grand Central Dispatch
Библиотека для работы с потоком. GCD оперирует блоками кода. Управлять блоками кода и планировать их нельзя. GCD вызывает потоки автоматически. Его используют для простых случаев, когда необходимо перенести блоки кода из главного потока. Например:  DispatchQueue.main.async { code }. 

Логика: есть блок кода который надо выполнить, этот код передается библиотеке и она помещает его в очередь DispatchQueue в каком-то потоке. Очереди могут выполняться последовательно или параллельно и перемещаться между потоками. Здесь мы оперируем очередями, а библиотека решает на какой поток идти. Задания могут выполняться на разных потоках Очередь использует поток или несколько потоков для выполнения поступающих к ней задач.

## Методы запуска кода
- Sync
Метод, позволяющий выполнять задачи синхронно по отношению к вызывающей очереди, значит что очередь блокируется и мы ждем пока выполнится блок кода в другой очереди

![фынтс Main Serial Queue 2](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/a5c22e8e-c35d-411a-97d9-0247dba4255a)

- Async
Метод, позволяющий выполнять задачи асинхронно по отношению к текущей очереди, значит что блоки код выполняются параллельно

![Main Serial Queue](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/be7ce397-84a4-4964-a6eb-249771476174)

## Очереди
- serial (последовательная) – выполняет задачи последовательно (поочередно). До тех пор, пока задача не будет выполнена, поток не приступит к выполнения следующей задачи в очереди. 
```swift
let serialQueue = DispatchQueue(label: "ru.app.serial-queue")
```
- concurrent (параллельная) – выполняет задачи параллельно. Задачи, поступающие в concurrent очередь, могут выполняться одновременно на разных потоках. 
```swift
let concurrentQueue = DispatchQueue(label: "ru.app.concurrent-queue", attributes: .concurrent)
```

![Serial Queue](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/093c17b1-74a8-4c46-9325-b51a5a81885e)


## Варианты готовых очередей GCD
- одна последовательная главная main queue
- глобальные очереди global queues типов приоритета quality of service:
  - userInteractive
  - userInitiated
  - utility
  - background

Важно понимать, что все эти глобальные очереди являются СИСТЕМНЫМИ глобальными очередями и наши задания — не единственные задания в этой очереди! Также важно знать, что все глобальные очереди, кроме одной, являются concurrent (параллельными) очередями

## Создание очередей
### Главная очередь
Главаня очередь уже существует. Подключение к глобальной очереди:
```swift
DispatchQueue.main.async {
    print("😇")
}
```
### Глобальные очереди
Глобальные очереди тоже уже существуют. Подключение к глобальной очереди:
```swift
DispatchQueue.global(qos: .utility).async { // qos указываем приоритет
    print("🥹")
}
```

### Своя очередь
Последовательная
```swift
let serialQueue = DispatchQueue(label: "my.queue.serial", qos: .userInitiated)
serialQueue.async {
    print("😇")
}
```
Параллельная
let concurrentQueue = DispatchQueue(label: "my.queue.concurrentQueue", qos: .userInitiated, attributes: .concurrent)
concurrentQueue.async {
    print("🫣")
}

## Группы задач DispatchGroup
Объект, позволяющий объединить задачи в группу и синхронизировать их поведение. Группа позволяет присоединить к ней несколько задачь или `DispatchWorkItem` и запланировать их асинхронное выполнение на одной или нескольких очередях. Когда все задачи в группе будут выполнены, группа уведомит об этом какую-либо очередь и выполнит на ней completion handler. Так же группа позволяет нам дождаться выполнения задач в группе синхронно, без использования уведомления.
Сравним работу последовательной и параллельной очереди в контексте нескольких задач.

Создадим последовательную очередь и добавим несколько операций через группу операций. Код выглядит так:
```swift
// Создаем очередь
let serialQueue = DispatchQueue(label: "ru.app.serial-queue")

// Создаем 2 DispatchWorkItem
let workItem1 = DispatchWorkItem {
    print("workItem1: 😴")
    sleep(3)
    print("workItem1: 🤗")
}

let workItem2 = DispatchWorkItem {
    print("workItem2: 😴")
    sleep(3)
    print("workItem2: 🤗")
}

// Создаем группу
let group = DispatchGroup()

// Добавляем workItem в группе, планируем его выполнение на очереди serialQueue и сразу возвращаем управление
serialQueue.async(group: group, execute: workItem1)
serialQueue.async(group: group, execute: workItem2)

// Устанавливаем уведомление. Замыкание будет выполнено на главной очереди сразу после того, как все задачи в группе будут выполнены.
group.notify(queue: DispatchQueue.main) {
    print("All tasks on group completed")
}

// Console:
// workItem1: 😴
// workItem1: 🤗
// workItem2: 😴
// workItem2: 🤗
// All tasks on group completed
```
Мы получили выполнение задач одну за другой и уведомление по итогу.

Создадим паралелльную очередь и добавим несколько операций через группу операций. Код выглядит так:
```swift
// Создаем очередь
let concurrentQueue = DispatchQueue(label: "ru.app.concurrent-queue", attributes: .concurrent)

// Создаем 2 DispatchWorkItem
let workItem1 = DispatchWorkItem {
    print("workItem1: 😴")
    sleep(3)
    print("workItem1: 🤗")
}

let workItem2 = DispatchWorkItem {
    print("workItem2: 😴")
    sleep(3)
    print("workItem2: 🤗")
}

// Создаем группу
let group = DispatchGroup()

// Добавляем workItem в группе, планируем его выполнение на очереди serialQueue и сразу возвращаем управление
concurrentQueue.async(group: group, execute: workItem1)
concurrentQueue.async(group: group, execute: workItem2)

// Устанавливаем уведомление. Замыкание будет выполнено на главной очереди сразу после того, как все задачи в группе будут выполнены.
group.notify(queue: DispatchQueue.main) {
    print("All tasks on group completed")
}

// Console:
// workItem1: 😴
// workItem2: 😴
// workItem2: 🤗
// workItem1: 🤗
// All tasks on group completed
```
Мы получили выполнение задач одновременно и уведомление по итогу.

##  Возможности
- Барьер
- Симафор
