# Size Classes или что такое размреный класс?
![SizeClassesExample](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/ff075eea-259a-41a9-82f6-39175c773589)

Существует два типа классов размеров:
- Горизонтальный размерный класса (Horizontal Size Class)
- Вертикальный размерный класс (Vertical Size Class)

Они используются для адаптации при изменении ориентации экрана.

### Черты Размера Класса
Каждый класс размеров имеет две черты:
- compact
- regular

Компактный (compact) означает узкое пространство, а обычный (regular) означает более широкое пространство. Это все, что вам нужно понять.

### Убедитесь, что вы это понимаете!
![SizeClasses](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/7a310e49-fd84-4f84-b204-28dfec37d9b6)

Если ваш телефон находится в портретном режиме, какова будет черта горизонтального класса?
===> Ответ: компактная (compact)

Если ваш телефон находится в альбомном режиме, какова будет черта горизонтального класса?
===> Ответ: Обычный (regular)

### Заметки Size Classes
Некоторые устройства iPhone имеют компактный размер как в горизонтальном, так и в вертикальном классе, потому что все они узкие!

Большинство устройств iPad имеют обычный размер как в горизонтальном, так и в вертикальном классе, потому что он широкий!

## Использование в коде
### SwiftUI пример
```swift
struct ContentView: View {
    
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @Environment(\.verticalSizeClass) var verticalSizeClass
    
    var body: some View {
        
        if horizontalSizeClass == .regular {
            
        }
        
        if verticalSizeClass == .compact {
            
        }
        
        Text("Hello, world!")
            .padding()
    }
}
```

### UIKit пример
```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        
        if view.traitCollection.horizontalSizeClass == .compact {
            
        }
        
        if view.traitCollection.verticalSizeClass == .regular {
            
        }
        
    }

}
```

## Время Викторины

Прежде чем начать викторину, давайте повторим еще раз!
Компактный (compact) означает узкое пространство
Регулярный (regular) означает более широкое пространство

1. Скорее всего, это будет пейзаж или портрет?

horizontalClass == .compact && verticalClass == .regular

===> Ответ: Скорее всего, это будет портретный режим iPhone.

2. Это портрет или пейзаж?

horizontalClass == .regular && verticalClass == .compact

===> Ответ: Скорее всего, это ландшафтный режим iPhone.

3. Это портрет или пейзаж?

horizontalClass == .regular && verticalClass == .regular

===> Ответ: Скорее всего, это будет IPad.

## Источники
- [Swift Size Class that can be understood in 3 seconds](https://paigeshin1991.medium.com/swift-size-class-that-can-be-understood-in-3-seconds-with-bad-brains-2782953197f0)
