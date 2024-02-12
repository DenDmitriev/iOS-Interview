# Property Wrappers 
SwiftUI предлагает 17 оберток свойств для наших приложений, каждое из которых обеспечивает различную функциональность. Знание того, какой из них использовать и когда имеет решающее значение для того, чтобы все было правильно.

- `@AppStorage` считывает и записывает значения из `UserDefaults`. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-appstorage-property-wrapper).
- `@SceneStorage` позволяет нам сохранять и восстанавливать небольшие объемы данных для восстановления состояния. Это работает немного похоже на `@AppStorage` в том, что вы предоставляете ему имя для сохранения свойства плюс значение по умолчанию, но вместо того, чтобы работать с UserDefaults, он вместо этого используется для восстановления состояния предстоваления, например показывается ли боковая панель iPad. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-scenestorage-property-wrapper).

- `@State` позволяет нам манипулировать небольшими объемами данных типа значений локально в представлении. Позволит нам изменять значения внутри структуры, что обычно не допускается, потому что структуры являются типами значений. Когда мы ставим @State перед свойством, мы эффективно перемещаем его хранилище из нашей структуры в общее хранилище, управляемое SwiftUI. Это означает, что SwiftUI может уничтожить и воссоздать нашу структуру, когда это необходимо (и это может произойти часто!), не теряя свойство, которое он хранил. `@State` следует использовать с простыми типами структур, такими как String, Int и массивы, и, как правило, не следует использовать совместно с другими представлениями. Если вы хотите делиться значениями между представлениями, вам, вероятно, следует использовать `@ObservedObject` или `@EnvironmentObject` - оба они гарантируют, что все представления будут обновлены при изменении данных. Это принадлежит своим данным. [Подробнее](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-state-property-wrapper).
- `@Binding` относится к данным типа значения `value type`, принадлежащим другому представлению. Изменение привязки локально также меняет удаленные данные. Прдестваление не владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-binding-property-wrapper).

- `@Environment` позволяет нам читать данные из системы, такие как цветовая схема, параметры доступности и коллекции черт, но вы можете добавить здесь свои собственные ключи, если хотите. Прдестваление не владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-environment-property-wrapper).

- `@EnvironmentObject` читает общий объект, который мы поместили в среду. Прдестваление не владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-environmentobject-property-wrapper).

- `@FetchRequest` запускает запрос на выборку `Core Data` для конкретной сущности. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-fetchrequest-property-wrapper).

- `@FocusedBinding` предназначен для наблюдения с в возможностью изменения за значениями в ключевом окне, такими как текстовое поле, которое в данный момент выбрано. Прдестваление не владеет данными . [Больше информации](https://swiftwithmajid.com/2021/03/03/focusedvalue-and-focusedbinding-property-wrappers-in-swiftui/)
- `@FocusedValue` это более простая версия - `@FocusedBinding`, которая только читает привязанное значение для вас. Прдестваление не владеет данными .

- `@GestureState` сохраняет значения, связанные с текущим жестом, например, как далеко вы его провели, отличается от `@State` тем что он будет сброшен до значения по умолчанию, когда жест остановится. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-gesturestate-property-wrapper).

- `@Namespace` создает пространство имен анимации, чтобы соединить соответствующие объекты общей анимацией, которые могут быть общими для других представлений. Прдестваление владеет данными . 

- `@ObservedObject` относится к экземпляру внешнего класса, который соответствует протоколу ObservableObject, служит чтобы представления могли следить за состоянием внешнего объекта и получать уведомления о том, что что-то важное изменилось. Он похож по поведению на `@StateObject`, за исключением того, что он не должен использоваться для создания объектов. Используйте @ObservableObject только с объектами, которые были созданы в другом месте, в противном случае SwiftUI может случайно уничтожить объект. Прдестваление не владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-observedobject-property-wrapper).
- `@Published` Прикрепляется к свойствам внутри ObservableObject и сообщает SwiftUI, что он должен обновлять любые представления, которые используют это свойство при его изменении. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-published-property-wrapper).

- `@StateObject` используется для хранения новых экземпляров данных ссылочного типа (reference type), которые соответствуют протоколу ObservableObject. Предназначена для заполнения очень специфического пробела в управлении состоянием: когда вам нужно создать тип ссылки внутри одного из ваших представлений и убедиться, что он остается живым для использования в этом представлении и других, с которыми вы делитесь им в отличии от `@ObservedObject`. Например мы создаем где-нибудь свой объект с помощью @StateObject, а затем используем его в других представлениях с помощью @ObservedObject. Прдестваление владеет данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-stateobject-property-wrapper).

- `@ScaledMetric` считывает настройку динамического типа пользователя и масштабирует числа вверх или вниз на основе исходного значения, заключанного вами, например для изменения размера объектов ы зависимости от настроек пользователя. Прдестваление владеет данными . [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-scaledmetric-property-wrapper).

- `@NSApplicationDelegateAdaptor` используется для создания и регистрации класса в качестве делегата приложения для приложения macOS. Он владеет его данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/how-to-add-an-appdelegate-to-a-swiftui-app).
- `@UIApplicationDelegateAdaptor` используется для создания и регистрации класса в качестве делегата приложения для приложения iOS. Он владеет своими данными. [Больше информации](https://www.hackingwithswift.com/quick-start/swiftui/what-is-the-uiapplicationdelegateadaptor-property-wrapper).


## Хранение данных в представлении
Когда дело доходит до хранения данных в вашем приложении, самая простая обертка свойств - это @State. Он предназначен для хранения типов значений, которые используются локально вашим представлением, поэтому он отлично подходит для хранения целых чисел, булов и даже локальных экземпляров структур.

Для сравнения, @Binding используется для простых данных, которые вы хотите изменить, но не принадлежат вашему представлению. В качестве примера представьте, как работает встроенный тумблер: ему нужно переключаться между состояниями включения и выключения, но он не хочет сохранять это значение сам, поэтому вместо этого у него есть привязка к некоторому внешнему значению, которым мы владеем. Итак, у нашего представления есть свойство @State, а у Toggle — свойство @Binding.

Для более сложных целей — например, для работы с классами или совместного использования данных во многих местах — вам не следует использовать @State и @Binding. Вместо этого вам следует создать где-нибудь свой объект с помощью @StateObject, а затем использовать его в других представлениях с помощью @ObservedObject.

> Простое правило заключается в следующем: если вы видите **State** в имени обертки свойств, это означает, что представление владеет этими данным.

Таким образом, `@State` означает простые данные типа значения (value type), созданные и управляемые локально, но, возможно, общие в других местах с помощью @Binding, а @StateObject означает данные эталонного типа, созданные и управляемые локально, но, возможно, общие в другом месте с помощью чего-то вроде @ObservedObject.

> Если вы когда-нибудь увидите @ObservedObject `var something = SomeType()`, это почти наверняка должно быть @StateObject, чтобы SwiftUI знало, что представление должно владеть данными, а не просто ссылаться на них в другом месте. Использование @ObservedObject здесь иногда может привести к сбою вашего приложения, потому что объект преждевременно уничтожен.

Если вы обнаружите, что передаете одни и те же данные от просмотра к просмотру, вы найдете полезную обертку свойств `@EnvironmentObject`. Это позволяет читать объект ссылочного типа из общей среды, а не передавать его явно.

Как и @ObservableObject, @EnvironmentObject не следует использовать для создания вашего объекта изначально. Вместо этого создайте его в другом представлении и используйте модификатор `environmentObject(myViewModel)` для введения его в среду. Хотя среда автоматически сохранит право собственности на ваш объект, вы также можете использовать @StateObject для хранения его там, где он был первоначально создан. Однако это не обязательно: поместить объект в окружающую среду достаточно, чтобы сохранить его жизнь без дальнейшего владения.

Например моздадим приложение список продуктов где реализуем обвертки:
```swift
struct ContentView: View {
    var body: some View {
        ShoppingList() // Представление которое будет показывать список покупок
    }
}
```

```swift
struct ShoppingList: View {
    
    @StateObject var shoppingNote = ShoppingNote() // 1: Класс который будет хранить все продукты
    @State var showAddProduct = false
    
    var body: some View {
        NavigationStack {
            VStack {
                if !shoppingNote.products.isEmpty {
                    List($shoppingNote.products) { product in
                        ProductItem(product: product) // 2: Представление которое оторажает объект
                    }
                } else {
                    Text("Add products")
                        .foregroundStyle(.secondary)
                }
            }
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Button {
                        showAddProduct.toggle()
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $showAddProduct) {
                AddProductView(shoppingNote: shoppingNote) // 3: Представление спомощью которого можно будет добавить продукты
                    .presentationDetents([.medium])
            }
        }
    }
}
```
```swift
class ShoppingNote: ObservableObject {
    @Published var products = [Product]() // Массив с нашим списком продуктов
}
```
```swift
struct AddProductView: View {
    @State var name: String = ""
    @State var emoji: String = ""
    @ObservedObject var shoppingNote: ShoppingNote // 4: Тот же объект хранения покупок из основного представления
    @Environment(\.dismiss) var dismiss
    private let emojiProduct = [...]
    
    var body: some View {
        VStack {
            HStack {
                TextField("🛒", text: $emoji)
                    .frame(width: 40)
                TextField("Product", text: $name)
                    .textFieldStyle(.roundedBorder)
            }
            .frame(height: 40)
            
            ScrollView {
                LazyVGrid(columns: [GridItem(.adaptive(minimum: 20))]) {
                    ForEach(emojiProduct, id: \.self) { emoji in
                        Text(emoji)
                            .onTapGesture {
                                self.emoji = emoji
                            }
                    }
                }
            }
            
            Button("Add to list") {
                shoppingNote.products.append(Product(name: name, emoji: emoji))
                dismiss()
            }
            .buttonStyle(.borderedProminent)
            .disabled(!isProduct)
        }
        .padding()
    }
    
    var isProduct: Bool {
        !name.isEmpty && !emoji.isEmpty
    }
}
```

1: Так как мы хотим создать хранимый class объект c данными покупок то используем в исходном представлении обвертку @StateObject для ShoppingNote

2: Мы передаем экземпляр Product в виде обвертки @Binding так как нам нужна привзяка с исходному объекту и мы хотим его менять в представлении.

3: AddProductView принимает в инициализатор уже созданный экземпляр shoppingNote объекта типа ShoppingNote

4: Объект ShoppingNote в представлении AddProductView уже в обвертке @ObservedObject так как мы его создали ранее в представлении ShoppingList в обвертке @StateObject и он не уничтожится
