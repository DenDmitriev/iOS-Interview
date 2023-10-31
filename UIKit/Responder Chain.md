# Responder Chain

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
`UIViewController`, `UIView`, `UIApplication` наследуют от UIResponder.
