# Responder Chain
`@MainActor class UIResponder : NSObject` Это класс объекта ответчика, который составляют основу для обработки событий приложения UIKt. Многие ключевые объекты также являются ответчиками, включая объект UIApplication, объекты UIViewController и все объекты UIView (включая UIWindow). По мере возникновения событий UIKit отправляет их в объекты-ответчик вашего приложения для обработки.

```swift
import UIKit

class ViewController: UIViewController {
    @IBOutlet weak var button: UIButton!
    
    override func viewDidLoad () {
        super.viewDidLoad()
        // Пример 1. self
        button.addTarget(self, action: #selector (doSomething), for: .touchUpInside)
        // Пример 1. nil
        button.addTarget(nil, action: #selector (doSomething), for: .touchUpInside)
    }
    
    @objc func doSomething () {
        print("👾")
    }
}
```
- Какая разница между первым и вторым примером?
- За что отвечает таргет?
- В каком случае вызывается метод при нажатие кнопки?

При нажатии на кнопку наш метод вызывается в обоих случаях.
Только в первом примере UIKit попытается вызвать метод в назначенном таргете(у нас это `ViewController`). Будет краш, если этого метода не существует.
Во втором же примере используется iOS Responder Chain, UIKit будет искать самого ближнего `UIResponder`-a у которого есть данный метод. Краша не будет, если наш метод не найден.
`UIViewController`, `UIView`, `UIApplication` наследуются от UIResponder.

## iOS Responder Chain и что под капотом

Всем процессом iOS Responder Chain занимается UIKit, который динамично работает со связным списком UIResponder-ов. Этот список UIKit создает из first responder(первый UIResponder который зарегистрировал событие, у нас это UIButton(UIView) и его subviews.

![yrt9yg8suuum84wzfkjt5mh_3ri](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/14b6e861-f124-4703-9639-e983a4a5c109)

UIKit проходит через список UIResponder-ов и проверяет с помощью canPerformAction на наличие нашей функции.

```swift
open func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool
```

Если выбранный UIResponder не может работать с конкретным методом,
UIKit рекурсивно посылает действия к следующему UIResponder-у в списке с помощью метода target который возвращает следующего UIResponder-а.
```swift
open func target(forAction action: Selector, withSender sender: Any?) -> Any?
```
Этот процесс повторяется до тех пор, пока кто-то из UIResponder-ов сможет работать с нашим методом или список закончится и это событие система проигнорирует.

Во втором примере нажатия обработалось UIViewController-ом, но UIKit сначала отправил запрос к UIView так как он был first responder. У него не было нужного метода, поэтому UIKit перенаправил действия на следующего UIResponder-а в связном списке кем являлся UIViewController у которого был нужный метод.

В большинстве случаев iOS Responder Chain это простой связной список subviews, но его очередность можно изменить. Можно заставить UIResponder (becomeFirstResponder) стать
первым UIResponder и вернуть его к старой позиции с помощью resignFirstResponder. Это часто используется с UITextField для показа клавиатуры которая будет вызвана, только когда UITextField является first responder-ом.
