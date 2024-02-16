# Идентификация представлений

SwiftUI представления являются структурами, и у них нет указателей в отличии от представлений UIKit. Чтобы быть эффективным и оптимизированным, SwiftUI должен понять, одинаковы ли представления или отличаются друг от друга. Также важно, чтобы представление определило содержащиеся в ней представления, чтобы сделать правильный переход и правильно отобразить представление, как только некоторые значения представления изменились.
При обновдении значений структура представдения должна отреагировать, однако это не всегда приводит к полному повторному отрендеру затронутого вида. И идентичность прдеставления является ключом к пониманию этого. Существует два способа идентификации представления в SwiftUI:
- Явная (Explicit)
- Структурная (Structural)

Давайте глубоко погрузимся в оба.

## Явный идентификатор
Представления могут быть идентифицированы с помощью пользовательских или управляемых данными идентификаторов. Явная идентификация может быть предоставлена с помощью идентификатора напрямую: `.id("Identifier")` . Он связывает идентификацию представления с заданным значением, которое должно быть хешируемым:

```swift
struct Fruit {
    let name: String
}
```
Чтобы показать прокручиваемый список фруктов, можно использовать структуру List:

```swift
struct ContentView: View {
    
    @State var fruits: [Fruit] = [
        Fruit(name: "🍊"),
        Fruit(name: "🍑"),
        Fruit(name: "🍌")
    ]
    
    var body: some View {
        ScrollView {
            VStack {
                List(fruits) { fruit in // Error: Инициализатор «init(_:rowContent:)» требует, чтобы «Fruit» соответствовал «Identifiable».
                    Text(fruit.name)
                }
            }
        }
    }
}
```
Однако это не будет скомпилировано, и будет ошибка: ссылка на инициализатор 'init(_:content:)' на 'List' требует, чтобы 'Fruit' соответствовал 'Identifiable'
Эту проблему можно решить либо путем реализации идентифицируемого протокола в структуре Fruit, либо путем предоставления ключевого пути. В любом случае, это позволит SwiftUI узнать, какую явную идентичность должен иметь FruitView:
```swift
struct ContentView: View {
    
    @State var fruits: [Fruit] = [
        Fruit(name: "🍊"),
        Fruit(name: "🍑"),
        Fruit(name: "🍌")
    ]
    
    var body: some View {
        ScrollView {
            VStack {
                List(fruits, id: \.name) { fruit in
                    Text(fruit.name)
                }
            }
        }
    }
}
```
Этот новый код будет скомпилироваться, и FruitView будет идентифицирован по имени, так как название фрукта разработано таким образом, чтобы быть уникальным.

## Структурная идентичность
Каждый вид SwiftUI должен иметь идентификатор. Если представление не имеет явной идентичности, оно имеет структурную идентичность. Структурная идентичность - это когда представление идентифицируется с помощью его типа и его положения в иерархии представлений. SwiftUI использует иерархию представлений для генерации неявной идентификации для представлений.
Рассмотрим следующий пример:
```swift
struct ContentView: View {
    @State var isRounded: Bool = false
    
    var body: some View {
        // Два различных представления. При ищменении isRounded будет показываться отличные друг от друга представления
        if isRounded {
            PizzaView()
                .cornerRadius(25)
        } else {
            PizzaView()
                .cornerRadius(0)
        }
 
        PizzaView() // Одно представление
            .cornerRadius(isRounded ? 25 : 0) // Будет скругление текущего представления
        
        Toggle("Round", isOn: $isRounded.animation())
            .fixedSize()
    }
}
```
Как видно из приведенного выше примера, существует два разных подхода к реализации анимированного изменения радиуса угла для PizzaView.

Первый подход создает два совершенно разных представления, в зависимости от логического состояния. На самом деле SwiftUI создаст экземпляр представления ConditionalContent за кулисами. Это представление ConditionalContent отвечает за представление того или иного представления на основе условия. И эти виды пиццы имеют разные виды из-за используемого состояния. В этом случае SwiftUI перерисует представление, как только переключатель изменится, и применит переход в/выход для изменения, как показано на рисунке 8 ниже. Важно понимать, что это не один и тот же PizzaView, это два разных взгляда, и у них есть свои структурные идентичности. 

Второй вариант меняется в обозначенном модификаторе. Это сохранит структурную идентичность представления, и SwiftUI не будет применять переход на вход/выход. Он оживит изменение радиуса угла, как показано на рисунке, потому что для структуры это один и тот же вид, только с разными значениями свойств.

![image1](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/b197d6ea-053f-41e9-80a6-9203bccba8e6)

В этом случае структурная идентичность представления не меняется. Apple рекомендует сохранить идентичность представления, поместив условия в модификатор представления, а не используя утверждения if/else.

Есть несколько вещей, которые нужно иметь в виду для достижения лучшей производительности:
- Сохраняйте идентичность представления. Если вы можете, не используйте условия if/else заявления для сохранения идентичности.
- Используйте стабильные идентификаторы для вашего представления, если они явно предоставлены.
- [Избегайте использования AnyView, если это возможно](https://www.swiftbysundell.com/articles/avoiding-anyview-in-swiftui/)

Источники:
- [Как работает жизненный цикл и идентификация SwiftUI View](https://doordash.engineering/2022/05/31/how-the-swiftui-view-lifecycle-and-identity-work/)