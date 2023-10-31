# Паттерн Factory

Предположим, мы в приложении торгуем средствами передвижения, и от типа конкретного средства зависит отображение: мы будем использовать разные подклассы UIViewController для разных средств передвижения. Помимо этого, все средства передвижения различаются состоянием (новые и б/у):
```swift
enum VehicleCondition{
    case new
    case used
}

final class BicycleViewController: UIViewController {
    
    private let condition: VehicleCondition
    
    init(condition: VehicleCondition) {
        self.condition = condition
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder aDecoder: NSCoder) {
        fatalError("BicycleViewController: init(coder:) has not been implemented.")
    }
    
}

final class ScooterViewController: UIViewController {
    
    private let condition: VehicleCondition
    
    init(condition: VehicleCondition) {
        self.condition = condition
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder aDecoder: NSCoder) {
        fatalError("ScooterViewController: init(coder:) has not been implemented.")
    }
    
}
```

Таким образом, у нас есть семейство объектов одной группы, экземпляры типов которых создаются в одних и тех же местах в зависимости от какого-то условия (например, пользователь нажал на товар в списке, и в зависимости от того, самокат это или велосипед, мы создаем соответствующий контроллер). Конструкторы контроллеров имеют некоторые параметры, которые также необходимо каждый раз задавать. Не свидетельствуют ли эти два довода в пользу создания «фабрики», которая одна будет обладать знаниями о логике создания нужного контроллера?

Конечно, пример достаточно простой, и в реальном проекте в похожем случае вводить «фабрику» будет явным «overengineering». Тем не менее, если представить, что типов транспортных средств у нас не два, а параметров у конструкторов – не один, то преимущества «фабрики» станут более очевидными.

Итак, объявим интерфейс, который будет играть роль «абстрактной фабрики»:

```swift
protocol VehicleViewControllerFactory {
    func makeBicycleViewController() -> UIViewController
    func makeScooterViewController() -> UIViewController
}
```

Интерфейс абстрактной фабрики содержит всего два метода: для создания контроллеров для продажи велосипедов и самокатов. Методы возвращают экземпляры не конкретных подклассов, а общего базового класса. Таким образом, ограничивается область распространения знаний о конкретных типах пределами той области, в которой это действительно необходимо.

В качестве «конкретных фабрик» будем использовать две реализации интерфейса абстрактной фабрики:

```swift
struct NewVehicleViewControllerFactory: VehicleViewControllerFactory {
    
    func makeBicycleViewController() -> UIViewController {
        return BicycleViewController(condition: .new)
    }
    func makeScooterViewController() -> UIViewController {
        return ScooterViewController(condition: .new)
    }
    
}

struct UsedVehicleViewControllerFactory: VehicleViewControllerFactory {
    
    func makeBicycleViewController() -> UIViewController {
        return BicycleViewController(condition: .used)
    }
    func makeScooterViewController() -> UIViewController {
        return ScooterViewController(condition: .used)
    }
    
}
```

В данном случае, как видно из кода, конкретные фабрики отвечают за транспортные средства разного состояния (новые и подержанные).

Создание нужного контроллера отныне будет выглядеть примерно так:

```swift
let factory: VehicleViewControllerFactory = NewVehicleViewControllerFactory()
let vc = factory.makeBicycleViewController()
```
