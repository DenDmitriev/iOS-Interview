# AppDelegate и SceneDelegate
При создании нового проекта XCode создаются файлы по умолчанию, такие как:
- AppDelegate.swift и SceneDelegate.swift
- ViewController.swift
- Main.storyboard

а также несколько других файлов.

## SceneDelegate
Начиная с iOS 13, обязанности AppDelegate были разделены между AppDelegate и SceneDelegate.
Возможности SceneDelegate:
- Поддержка нескольких окон, которая представлена с iPad-OS.
- `UIScene` позволяет нам создавать несколько экземпляров пользовательского интерфейса нашего приложения как отдельные экземпляры в переключателе приложений.

## Какие изменения произошли в новом `UIScene`?
Чтобы понять изменения в жизненном цикле приложения, вам нужно знать о следующих основных объектах нового `UIScene`.

### `UIScene` и `UISceneDelegate`:
- Теперь приложение может запускать несколько экземпляров пользовательского интерфейса одновременно.
- `AppDelegate` больше не несет ответственности за настройку пользовательского интерфейса и обработку событий жизненного цикла, связанных с пользовательским интерфейсом, таких как переход на передний план или фон.
- Каждый экземпляр пользовательского интерфейса приложений представлен объектом `UIScene`.
- Как `UIApplication` уведомляет о событиях приложения через делегата AppDelegate приложения, так и сцена `UIScene` делает это через делегата сцены `UIWindowSceneDelegate`? 
- Обычно вы работаете с `UIWindowScene` и `UIWindowSceneDelegate`.

#### `UIWindowScene`
`UIWindowScene` — это подкласс `UIScene` и наиболее распространенный тип сцены, с которой вам придется взаимодействовать.
Не следует напрямую создавать экземпляр `UIWindowScene`. `UIKit` создаст его за вас. У UIWindowScene есть жизненый цикл и на его события реагирует `UIWindowSceneDelegate`.

#### `UIWindowSceneDelegate`
`UIWindowSceneDelegate` — это протокол делегата, который является продолжением UISceneDelegate и содержит основные методы, которые используются для реагирования на события жизненного цикла UIWindowScene.

### `UISceneSession`
Жизненный цикл UISceneSession обрабатывается `UIApplicationDelegate`. Каждая сцена (окно) управляется `UISceneSession`, Который содержит все данные конфигурации и данные восстановления состояния, необходимые для создания объекта сцены.

#### `UISceneConfiguration`
Этот объект содержит информацию, которая сообщает `UIKit`, как создать экземпляр вашей сцены. Вы настраиваете `UISceneConfiguration` в коде или в своем Info.plist. Это выглядит следующим образом.

![UISceneConfig](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/d14db7f0-e3df-4f40-89dc-a50f4a37312f)

- Application Session Role - говорит, что вы можете иметь несколько конфигураций в своем Info.plist.
- Enable Multiple Windows - ключ по умолчанию ложный. Установка для свойства «Enable Multiple Windows» значения «YES» позволит пользователям открывать несколько окон вашего приложения на iPadOS (или даже на macOS).

Самая важная информация хранится внутри элементов в массиве Application Session Role:
- Название этой конфигурации, которое должно быть уникальным
- Название класса сцены `UIWindowScene`
- Имя класса делегата для этой сцены, которое обычно `SceneDelegate`
- Название storyboard, которая содержит начальный пользовательский интерфейс для этой сцены - Если вы используете storyboard, устанавливать это свойство нет необходимости.

### Кто за что отвечает
| AppDelegate | SceneDelegate |
| - | - |
| UISceneSession | UIScene/UIWindowScene |
| UISceneConfiguration | UISceneDelegate/UIWindowSceneDelegate |


## Изменения в AppDelegate и его обязанностей:
Это по-прежнему основная точка входа для приложения iOS 13+. Система вызывает методы `AppDelegate` для событий жизненного цикла уровня приложения.
В шаблоне Apple по умолчанию вы можете найти ниже три метода:
1. `func application(_:didFinishLaunchingWithOptions:) -> Bool`
Этот метод используется для выполнения настройки приложения.
В iOS 12 или более ранней версии вы, возможно, использовали этот метод для создания и настройки объекта UIWindow и назначили экземпляр `UIViewController` в окно, чтобы он появился. Теперь `AppDelegate` больше не несет за это ответственность.

2. `func application(_:configurationForConnecting:options:) -> UISceneConfiguration`
Этот метод вызывается всякий раз, когда ожидается, что ваше приложение предоставит новую сцену или окно для отображения iOS.
Этот метод не вызывается при первоначальном запуске вашего приложения, он вызывается только для получения и создания новых сцен.

3. `func application(_:didDiscardSceneSessions:)`
Этот метод вызывается всякий раз, когда пользователь завершает сцену.
Вы можете использовать эту функцию, чтобы избавиться от ресурсов, которые использовали эти сцены, потому что они больше не нужны.

> В дополнение к этим методам по умолчанию `AppDelegate` по-прежнему можно использовать для открытия URL-адресов, перехвата предупреждений памяти, определения момента завершения работы вашего приложения, определения регистрации пользователя для удаленных уведомлений и многого другого.

## Обязанности SceneDelegate
По умолчанию в файле SceneDelegate.swift есть несколько методов:
- `sceneDidDisconnect(_:)` вызывается, когда сцена была отключена от приложения (обратите внимание, что она может подключиться позже).
- `sceneDidBecomeActive(_:)` вызывается, когда пользователь начинает взаимодействовать со сценой, например, выбирает ее из переключателя приложений
- `sceneWillResignActive(_:)` вызывается, когда пользователь перестает взаимодействовать со сценой, например, переключаясь на другую сцену
- `sceneWillEnterForeground(_:)` вызывается, когда сцена выходит на передний план, то есть запускается или возобновляется из фонового состояния
- `sceneDidEnterBackground(_:)` вызывается, когда сцена попадает в фон, т.е. приложение свернуто, но все еще присутствует в фоновом режиме.

### Жизненный цикл сцены (scene)

<img width="750" alt="Life Cycle of Scene" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/1231c3be-21c0-48ca-86ae-28495dd233b9">

## Программное добавление нескольких сцен
Перед запуском убедитесь, что устройство Supports multiple windows в info.plist. 
