# CoreData
Гибкий фрэймворк от Apple для работы с хранимыми на устройстве данными — Core Data. Не смотря на то, что Core Data может хранить данные в реляционной базе данных вроде SQLite, Core Data не является СУБД. Core Data скорее является оболочкой/фрэймворком для работы с данными, которая позволяет работать с сущностями и их связями (отношениями к другим объектами), атрибутами, в том виде, который напоминает работы с объектным графом в обычном объектно-ориентированном программировании.

CoreData очень мощный инструмент, который позволяет вам сохранять данные в вашем приложении и использовать их в дальнейшем в виде моделей. Модели представляют данные, хранящиеся в таблицах или коллекциях в вашей базе данных. Модели имеют одно или несколько полей, в которых хранятся значения. 

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

Для полей, содержащих необязательное значение, используйте Optional и отметьте галочку в `.xcdatamodeld` как в текущей модели. Если нужно указать не опциональное значение то нужно снять галочку и указать дефолное значение

<img width="1052" alt="Снимок экрана 2024-03-12 в 16 52 34" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/47bdfd4d-cf2b-42f2-8d22-6960b1f356e6">

> При создании объекта методом Editor > Create NSManagedObject Subclass свойство останется опциональным и нужно самостоятельно убрать знак опционала.
```swift
extension Planet {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Planet> {
        return NSFetchRequest<Planet>(entityName: "Planet")
    }

    // The Planet's name.
    @NSManaged public var name: String

    public override func awakeFromInsert() {
        super.awakeFromInsert()
        name = "Unknown"
    }
}
```

## Работа с данными
Чтоб рабоать с данными нужно разобрать иерархию Core Data Stack:
Core Data предоставляет набор классов, которые совместно поддерживают слой базы данных приложения:
- `NSPersistentContainer` для согласованной настройки модели (Entity), контекста и координатора хранилища.
- `NSManagedObjectModel` описывает модели данных, включая их свойства и отношения.
- `NSManagedObjectContext` (среда управляемых объектов) отслеживает изменения экземпляров данных приложения. 
- `NSPersistentStoreCoordinator` сохраняет и избирает экземпляры данных из хранилища.

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

## Обновление
Чтоб обновить объект нужно получить объект одним из способо, изменить его, затем сохранить контекст
```swift
let venera = try? context.existingObject(with: id) as? Planet
venera?.name = "Mars"
try? context.save()
```
