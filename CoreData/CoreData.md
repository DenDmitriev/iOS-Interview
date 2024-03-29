# CoreData
Гибкий фрэймворк от Apple для работы с хранимыми на устройстве данными — Core Data. Не смотря на то, что Core Data может хранить данные в реляционной базе данных вроде SQLite, Core Data не является СУБД. Core Data скорее является оболочкой/фрэймворком для работы с данными, которая позволяет работать с сущностями и их связями (отношениями к другим объектами), атрибутами, в том виде, который напоминает работы с объектным графом в обычном объектно-ориентированном программировании.

CoreData очень мощный инструмент, который позволяет вам сохранять данные в вашем приложении и использовать их в дальнейшем в виде моделей. Модели представляют данные, хранящиеся в таблицах или коллекциях в вашей базе данных. Модели имеют одно или несколько полей, в которых хранятся значения. 

## Функциональные возможности
- Обертка базы данных без SQL запросов
- Простое чтение данных в основном потоке
- Простое в использовании сохранение данных в фоновом потоке
- Простая настройка базы данных в памяти (например, для кэширования или модульного тестирования)
- Поддержка автоматической миграции базы данных между выпусками приложений
- Простой в настройке инструмент моделирования базы данных (с помощью Interface Builder в `.xcdatamodeld`)

## Определение модели (сущность)
Для создания модели данных используется графический способ в файле `.xcdatamodeld`. Файл представляет собой директорию в файловой системе, которая содержит информацию о структуре модели данных. Модель данных является основой каждого приложения использующего Core Data. Если его нет, то нужно его создать cmd + N. 

<img width="1052" alt="Снимок экрана 2024-03-12 в 14 51 37" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/c0f028fe-0f45-4920-a36f-83ed1ef3b536">

Перейдя в этот файл мы видим следущие рабочие области:

<img width="1052" alt="Снимок экрана 2024-03-12 в 14 54 06" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/6f167121-a3fe-4983-827d-593040ea14c9">

### Entity
Вы можете думать об Entity как о вашей структуре данных, которая имеет наследование от определенного класса — NSManagedObject, а так же будет иметь определенные атрибуты для данных. В этом поле вы можете изменить имя для вашего Entity.

### Attributes
Это атрибуты вашего Entity о котором говорилось выше. По нажатию на "+" вы можете добавить необходимый для вас атрибут. 
Атрибуты бывают следующих видов:
- Integer 16/32/64 — используйте для целочисленных значений. Разница между этими значениями лишь в том, насколько большое число вы можете сохранить. Так же чтобы сохранить число типа Int, вы должны привести его к одному из этих значнией. Пример: `Integer16(15)`

- Decimal/Double/Float — используйте для чисел с плавающей точкой. Разница между этими значениями лишь в том, сколько цифр после точки будет сохранено.

- String — используйте для строковых значений.

- Boolean — используйте этот тип если хотите сохранить булевский оператор.

- Date — используйте для типа Date. 
> Core Data проводит операцию конвертации и просто сохраняет число в секундах с 00:00:00 UTC 1 Января 1970 до вашей даты.)

- Binary Data — используйте этот тип если хотите сохранить Data. 
>  Всегда ставьте чекмарк на поле Allow External Storage (Разрешить внешнее хранилище). Core Data проверяет обьем данных, и если он превышает 100 килобайт, то данные будут сохранены отдельно от Core Data а данное поле будет хранить ссылку на объект. Это улучшит производительность приложения и избавит от ненужных проблем с памятью.

- UUID — используйте этот тип если хотите сохранить уникальный идентификатор типа UUID
- URI — используйте этот тип если хотите сохранить ссылку URL
- Transformable — позволяет сохранить кастомный тип данных, однако эти данные должны поддерживать протокол NSCoding.

### Relationship
В этом поле вы можете назначать взаимосвязи с другими вашими Entity. Relationships (отношения) могут быть:
- One-to-One. У собаки есть всего один хозяин.
- One-to-Many. У человека может быть несколько домашних животных.
- Many-to-Many. У родителя может быть несколько детей а у детей может несколько родителей.

### Codegen
Вкратце, это поле указывает, кем будет регулироваться создание `NSManagedObject` классов, наследуемый Entity. Для простоты работы выбираем Manual/None, далее Core Data сможет сама создать необходимые классы для ваших Entity.


Ниже приведен пример простой модели с одним полем. 

<img width="1052" alt="Снимок экрана 2024-03-12 в 15 29 53" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/d48cb20e-e7fc-46f5-a646-7e6ae42b8003">

## Создание типа объекта
Чтобы создать класс наших Entity используйте: Editor > Create NSManagedObject Subclass

<img width="379" alt="Снимок экрана 2024-03-12 в 15 31 17" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/e9487a79-f701-4db7-90b3-fe4916f8458a">

Xcode создаст два файла:

Planet+CoreDataClass.swift
```swift
@objc(Planet)
public class Planet: NSManagedObject {

}
```

Planet+CoreDataProperties.swift
```swift
extension Planet {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Planet> {
        return NSFetchRequest<Planet>(entityName: "Planet")
    }

    // The Planet's name.
    @NSManaged public var name: String?

}

extension Planet : Identifiable {

}
```

Если вы хотите задать дефолтные значения для ваших атрибутов используйте метод `awakeFromInsert()`:
```swift
extension Planet {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Planet> {
        return NSFetchRequest<Planet>(entityName: "Planet")
    }

    // The Planet's name.
    @NSManaged public var name: String?

    public override func awakeFromInsert() {
        super.awakeFromInsert()
        name = "Unknown"
    }
}
```

Для полей, содержащих необязательное значение, используйте Optional и отметьте галочку в `.xcdatamodeld` как в текущей модели. 
> Optional значение в CoreData не имеет никакого отношения к Swift Optional. Core Data понимает это по-своему и так, что при записи в базу данных этот атрибут может вообще не иметь значения. CoreData не знает что значит Swift Optinal, знает только, что это значение может быть равно nil.

Поэтому Xcode всегда использует Swift Optional значения в сгенерированном коде, так как это самый безопасный способ справиться с разницей Optional в Swift и Core Data. Если свойство является Optional для Swift, то он может безопасно обрабатывать более слабые ограничения Core Data. Здесь имеет место то, что называется “столкновением” Optionals.  

Можно ли убрать Swift Optional? 
Можно. Но вы только что нарушили правила инициализации переменных в Swift! В частности, вы инициализировали объект, но не предоставили значения для всех его НЕ Optional свойств. Вас никто не остановит, потому что `@NSManaged` говорит, что правила Swift на её территории не применяются. Таким образом, вы только что создали бомбу замедленного действия. Но можно убрать галочку указав дефолтное значение для этого атрибута.

<img width="1052" alt="Снимок экрана 2024-03-12 в 16 52 34" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/47bdfd4d-cf2b-42f2-8d22-6960b1f356e6">

> При создании объекта методом Editor > Create NSManagedObject Subclass свойство останется опциональным и нельзя самостоятельно убрать знак опционала.
```swift
extension Planet {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Planet> {
        return NSFetchRequest<Planet>(entityName: "Planet")
    }

    // The Planet's name.
    @NSManaged public var name: String?

    public override func awakeFromInsert() {
        super.awakeFromInsert()
        name = "Unknown"
    }
}
```

## Работа с данными
Чтоб рабоать с данными нужно разобрать иерархию Core Data Stack:
Core Data предоставляет набор классов, которые совместно поддерживают слой базы данных приложения:
- `NSPersistentContainer` (Постоянный контейнер) для согласованной настройки модели (Entity), контекста и координатора хранилища.
- `NSManagedObjectModel` (Модель управляемого объекта) описывает модели данных, включая их свойства и отношения.
- `NSManagedObjectContext` (Среда управляемого объекта) отслеживает изменения экземпляров данных приложения. 
- `NSPersistentStoreCoordinator` (Постоянный координатор магазина) сохраняет и избирает экземпляры данных из хранилища.

<img width="921" alt="Снимок экрана 2024-03-12 в 17 27 56" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/fad77e64-d14d-4c45-be98-f3bcae9e9982">

Если простыми словами, то все модели (Entity) на данной диаграмме это Managed Object. Все Managed Objects хранятся в NSManagedObjectContext. Context cсылается на NSPersistentContainer который является обверткой над NSPersistentStoreCoordinator. NSPersistentStoreCoordinator хранит после обращения к SQLite ваши сохраненные данные, в виде Row Cache.  

Чтобы создать экземпляр вашего ManagedObject (Entity), вы должны обратиться к своему NSPersistentContainer.

```swift
let container: NSPersistentContainer = {
        // 1. Инициализация постоянного контейнера
        let container = NSPersistentContainer(name: "CoreDataDocumentation")

        // 2. Поручить контейнеру загрузить постоянные хранилища (PersistentStores) и завершить создание стека основных данных.
        // Обработчик завершения `completionHandler` будет вызываться один раз для каждого созданного постоянного хранилища.
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            // Если есть ошибка при загрузке постоянных хранилища (PersistentStores), будет заполнено значение NSError.
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        // Указывает, автоматически ли контекст объединяет изменения, сохраненные в его постоянном координаторе (PersistentStoreCoordinator) или родительском контексте.
        container.viewContext.automaticallyMergesChangesFromParent = true
        return container // Cтек полностью инициализировался и готов к использованию
    }()
```

Далее мы должны обратиться к NSManagedObjectContext через наш PersistentContainer:

```swift
let context = container.viewContext
```

Теперь мы наконец можем создать экземпляр нашего Manage Object из нашего контекста и присвоить ему значение:

```swift
let planet = Planet(context: context)
planet.name = "Earth"
```

Чтоб объект попал в базу данных, нужно сохранить объект:

```swift
try? context.save()
```

Теперь рассмотрим типовые функции работы с данными используя `container` (Persistent Container) с `context` контекстом (Managed Object Context).

```swift
class PlanetsPersistenceController: ObservableObject {
    let container: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "CoreDataDocumentation")
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        container.viewContext.automaticallyMergesChangesFromParent = true
        return container
    }()
    
    lazy var context = container.viewContext
}
```

## Создание

```swift
let planet = Planet(context: context)
planet.name = "Earth"
try? context.save()
```

## Обновление

## Удаление
Функция контекста `delete(_:)` позволяет нам легко удалить объект из Core Data:
```swift
context.delete(planet)
try? context.save()
```

## Получение
### Получение всех объектов типа
Для того чтобы получить массив всех объектов типа из Core Data мы должный выполнить NSFetchRequest.
```swift
let request = Planet.fetchRequest() // NSFetchRequest<Planet>
let planets = try? context.fetch(request) // [Planet]?
```

или используя пустой предикат
```swift
let request = Planet.fetchRequest()
request.predicate = NSPredicate(value: true)
let planets = try? context.fetch(request)
```

### Получение выборки объектов по предикату
Чтобы добавить фильтр в запрос на выборку данных, используется предикат. Затем используйте функцию fetch(_:) на экземпляре NSManagedObjectContext:
```swift
let request = Planet.fetchRequest()
request.predicate = NSPredicate(format: "name LIKE %@", "Mars")
let mars = try? context.fetch(request)
```

### Получение объекта по ID
```swift
// mars - Planet object
let id = mars.objectID // NSManagedObjectID
let planet = try? context.existingObject(with: id) as? Planet
```

### FetchRequest
Под капотом NSFetchRequest делает огромную работу:
1. NSFetchRequest отправляется в Persistent Store Coordinator
2. Persistent Store Coordinator отправляет запрос в Persistent Store
3. Persistent Store отправляет отправляет запрос в SQL
4. SQL делает запрос и выдает все подходящие строки. Каждая строка имеет ID и Row Value. Это Row Value передается в Persistent Store и храниться там в виде Row Cache со своим ID.
5. Persistent Store инициализирует Managed Objects для определенных ID и возвращает их в Persistent Store Coordinator. Полученные данные являются fault (пустыми) до того момента пока мы не обратимся к конкретному объекту. Если в Managed Object Context уже был объект с нужным ID, Core Data не будет отправлять запрос ниже к SQL.
6. Persistent Store Coordinator возвращает в context массив Managed Object
7. Перед возвращением объекта Core Data проверяет объекты на изменения. Чтобы убрать это поведение можете установить флаг includePendingChanges = false
8. Возвращается массив NSManagedObject

> Пока выполняется NSFetchRequest Managed Object Context и Persistent Store Coordinator выполняется в синхронной очереди и все процессы блокируются пока другой процесс отрабатывается

Свойства FetchRequest:
#### FetchLimit
FetchLimit — указывает лимит по выборке объектов.

#### FetchOffset
FetchOffset — указывает отступ. Если указать значение 2, то показываться элементы будут со второго элемента.

#### FetchBatchSize
FetchBatchSize — указывает какое количество элементов будет подгружено за раз. Если вы отображаете элементы в Table/CollectionView и у вас за раз показывается всего 5 элементов, в целях производительности лучше подгружать не больше 7 элементов за раз, чем брать сразу все данные и держать их в памяти.

#### SortDescriptor
SortDescriptor — указывает по какому ключу следует отсортировать запрос, а так же по возрастанию либо наоборот.

#### ReturnsObjectsAsFaults
ReturnsObjectsAsFaults — указывает, что наши значения могут придти пустыми, и когда мы к ним обратимся, они будут подгружены с нашего PersistentStore которые в данный момент находятся там в виде RawCache

## Обновление
Чтоб обновить объект нужно получить объект одним из способо, изменить его, затем сохранить контекст
```swift
let venera = try? context.existingObject(with: id) as? Planet
venera?.name = "Mars"
try? context.save()
```

## Чтение и сохранения данных
Если ваше приложение активно задействует какой-нибудь API через которое оно получает данные, и вы хотите эти данные где-то сохранять, то отличный выбор — Core Data. Какие у вас могут появиться проблемы с этим? Первое и самое очевидное — количество данных будет так велико, что при попытке сохранить их, наше приложение зависнет на какое-то время, что скажется впечатлении пользователя. Обычным GCD тут не обойтись так как независимо, наш Core Data Stack, о котором говорилось тут, работает синхронно и при любом обращении к нашему NSManageObjectContext нам придется ждать до конца выполнения цикла.

Но тут на помощь нам приходит приватный NSManageObjectContext который работает в background потоке. Для того чтобы его вызвать требуется сначала обратиться к нашему NSPersistentContainer.

Инициализация будет выглядеть следующим образом:
```swift
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "Preset")
    container.loadPersistentStores { (persistent, error) in
        if let error = error {
            fatalError("Error: " + error.localizedDescription)
        }
    }
    container.viewContext.mergePolicy = NSMergeByPropertyStoreTrumpMergePolicy
    container.viewContext.shouldDeleteInaccessibleFaults = true
    container.viewContext.automaticallyMergesChangesFromParent = true
    return container
}()
```

### MergePolicy
Политика слияния контекста. Тут вы указываем политику слияния наших NSManageObjectContext. Тут мы явно указываем как должна себя повести Core Data в случае конфликта данных.

Варианты MergePolicy:
- Rollback `NSRollbackMergePolicy` — В случае появления конфликта, отбрасывает все изменения до его появления
- Overwrite `NSOverwriteMergePolicy` — Сохранит новые значения независимо от данных
- MergeByPropertyStore `NSMergeByPropertyStoreTrumpMergePolicy` — Сохраняет измененные объекты свойство за свойством, в данном случае преобладать будут сохраненные данные
- MergeByPropertyObjectTrump `NSMergeByPropertyObjectTrumpMergePolicy` — Сохраняет измененные объекты свойство за свойством, в данном случае преобладать будут новые данные

### AutomaticallyMergesChangesFromParent
Говорит о том будет ли наш контекст автоматически объединять данные
После чего создаем новый контекст:
```
let context = persistentContainer.viewContext
let backgroundContext = persistentContainer.newBackgroundContext()
```
Теперь у нас имеется два NSManageObjectContext. 
1. `context` служит для работы с UI и работает на главном потоке, а второй имеет privateQueueConcurrencyType для работы в фоне.
2. `backgroundContext` мы будем использовать его для скачивания данных.

Тут мы создаем наше Entity и далее можем присвоить ему необходимые свойства, после чего вызываем метод сохранения, выглядит он следующим образом:
```swift
    func saveChanges(with context: NSManagedObjectContext) {
        context.performAndWait {
            if context.hasChanges {
                do {
                    try context.save()
                } catch {
                    context.rollback()
                }
            }
            context.reset()
        }
    }
```

Есть 2 метода на выбор:
1. performAndWait — выполняет действия на потоке контекста синхронно
2. perform — выполняет действия на потоке контекста асинхронно

## Источники
- [Стартуем с Core Data! Сложное простыми словами. Часть 1](https://habr.com/ru/articles/493262/)
- [Стартуем с Core Data! Сложное простыми словами. Часть 2](https://habr.com/ru/articles/495602/)
- [Core Data в современном интерьере SwiftUI. Некоторые уточнения и заблуждения. часть 1](https://habr.com/ru/articles/663974/)
- [Core Data stack](https://developer.apple.com/documentation/coredata/core_data_stack)
