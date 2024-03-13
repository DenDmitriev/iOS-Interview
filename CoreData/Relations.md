# Relations
CoreData позволяет вам создавать и поддерживать ссылки между вашими моделями через отношения. Поддерживаются три типа отношений:
- One-to-One. У планеты есть всего одна звезда вокруг которой она вращается (считаем двойные звезды как одну).
- One-to-Many. У звезды может быть несколько планет которые вращаются вокруг неё.
- Many-to-Many. У созвездия может быть несколько звезд а звезда может входит в различные созвездия.

Для демонстрации отношений создадим новую сущность Star и определим отношение с Planet как One-to-Many. У одной звезды могут быть несколько планет а у планеты всего одна звезда. Отразим это в директории xcdatamodeld.

![Relations  xcdatamodeld](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/9f882347-fb12-4bc4-b2f7-8660a28f5803)


После генрации кода модели будут выглядет так:
```swift
extension Star {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Star> {
        return NSFetchRequest<Star>(entityName: "Star")
    }

    @NSManaged public var name: String?
    @NSManaged public var planets: NSSet? // Свойство planets отношение один ко многим между двумя моделями.

}
```

```swift
extension Planet {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Planet> {
        return NSFetchRequest<Planet>(entityName: "Planet")
    }

    @NSManaged public var name: String?
    @NSManaged public var star: Star? // Отношение star хранит ссылку на свойство другой модели.

}
```

Рассмотрим методы для работы с этими моделями и отношениями:

Для начала получим звезду солнце из базы данных:
```swift
let request = Star.fetchRequest()
request.predicate = NSPredicate(format: "name == %@", name)
request.fetchLimit = 1
let sun = (try? context.fetch(request))?.first // Star?
```
Приведение к неопуиональному типу Star и поработаем с моделью:

## Получеение
Получение всеx планет солнца.
```swift
let context = container.viewContext
let planets = star.planets.allObjects as? [Planet] // NSSet -> [Any] -> [Planet]?
```

## Query
Получение всех планет солнца начинающихся на букву М.
```swift
let context = container.viewContext
let request = Planet.fetchRequest()
request.predicate = NSPredicate(format: "name CONTAINS %@ && star = %@", "M", sun)
let planets = try? context.fetch(request)
```
