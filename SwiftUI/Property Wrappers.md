# Property Wrappers 
SwiftUI предлагает 17 оберток свойств для наших приложений, каждое из которых обеспечивает различную функциональность. Знание того, какой из них использовать и когда имеет решающее значение для того, чтобы все было правильно.

- `@AppStorage` считывает и записывает значения из `UserDefaults`. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-appstorage-property-wrapper).
- `@SceneStorage` позволяет нам сохранять и восстанавливать небольшие объемы данных для восстановления состояния. Это работает немного похоже на `@AppStorage` в том, что вы предоставляете ему имя для сохранения свойства плюс значение по умолчанию, но вместо того, чтобы работать с UserDefaults, он вместо этого используется для восстановления состояния предстоваления, например показывается ли боковая панель iPad. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-scenestorage-property-wrapper).

- `@State` позволяет нам манипулировать небольшими объемами данных типа значений локально в представлении. Позволит нам изменять значения внутри структуры, что обычно не допускается, потому что структуры являются типами значений. Когда мы ставим @State перед свойством, мы эффективно перемещаем его хранилище из нашей структуры в общее хранилище, управляемое SwiftUI. Это означает, что SwiftUI может уничтожить и воссоздать нашу структуру, когда это необходимо (и это может произойти часто!), не теряя свойство, которое он хранил. `@State` следует использовать с простыми типами структур, такими как String, Int и массивы, и, как правило, не следует использовать совместно с другими представлениями. Если вы хотите делиться значениями между представлениями, вам, вероятно, следует использовать `@ObservedObject` или `@EnvironmentObject` - оба они гарантируют, что все представления будут обновлены при изменении данных. Это принадлежит своим данным. [Подробнее](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-state-property-wrapper).
- `@Binding` относится к данным типа значения `value type`, принадлежащим другому представлению. Изменение привязки локально также меняет удаленные данные. Это не владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-binding-property-wrapper).

- `@Environment` позволяет нам читать данные из системы, такие как цветовая схема, параметры доступности и коллекции черт, но вы можете добавить здесь свои собственные ключи, если хотите. Это не владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-environment-property-wrapper).

- `@EnvironmentObject` читает общий объект, который мы поместили в среду. Это не владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-environmentobject-property-wrapper).

- `@FetchRequest` запускает запрос на выборку `Core Data` для конкретной сущности. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-fetchrequest-property-wrapper).

- `@FocusedBinding` предназначен для наблюдения с в возможностью изменения за значениями в ключевом окне, такими как текстовое поле, которое в данный момент выбрано. Это не владеет своими данными. [Больше информации](https://swiftwithmajid.com/2021/03/03/focusedvalue-and-focusedbinding-property-wrappers-in-swiftui/)
- `@FocusedValue` это более простая версия - `@FocusedBinding`, которая только читает привязанное значение для вас. Это не владеет своими данными.

- `@GestureState` сохраняет значения, связанные с текущим жестом, например, как далеко вы его провели, отличается от `@State` тем что он будет сброшен до значения по умолчанию, когда жест остановится. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-gesturestate-property-wrapper).

- `@Namespace` создает пространство имен анимации, чтобы соединить соответствующие объекты общей анимацией, которые могут быть общими для других представлений. Это владеет своими данными. 

- `@ObservedObject` относится к экземпляру внешнего класса, который соответствует протоколу ObservableObject, служит чтобы представления могли следить за состоянием внешнего объекта и получать уведомления о том, что что-то важное изменилось. Он похож по поведению на `@StateObject`, за исключением того, что он не должен использоваться для создания объектов. Используйте @ObservableObject только с объектами, которые были созданы в другом месте, в противном случае SwiftUI может случайно уничтожить объект. Это не владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-observedobject-property-wrapper).
- `@Published` Прикрепляется к свойствам внутри ObservableObject и сообщает SwiftUI, что он должен обновлять любые представления, которые используют это свойство при его изменении. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-published-property-wrapper).

- `@StateObject` используется для хранения новых экземпляров данных ссылочного типа (reference type), которые соответствуют протоколу ObservableObject. Это владеет своими данными. Предназначена для заполнения очень специфического пробела в управлении состоянием: когда вам нужно создать тип ссылки внутри одного из ваших представлений и убедиться, что он остается живым для использования в этом представлении и других, с которыми вы делитесь им в отличии от `@ObservedObject`. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-stateobject-property-wrapper).

- `@ScaledMetric` считывает настройку динамического типа пользователя и масштабирует числа вверх или вниз на основе исходного значения, заключанного вами, например для изменения размера объектов ы зависимости от настроек пользователя. Это владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-scaledmetric-property-wrapper).

- `@NSApplicationDelegateAdaptor` используется для создания и регистрации класса в качестве делегата приложения для приложения macOS. Он владеет его данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/how-to-add-an-appdelegate-to-a-swiftui-app).
- `@UIApplicationDelegateAdaptor` используется для создания и регистрации класса в качестве делегата приложения для приложения iOS. Он владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-uiapplicationdelegateadaptor-property-wrapper).