# @ViewBuilder
это конструктор результатов используемый в синтаксисе библиотеки SwiftUI. Данный конструктор является частью привычного протокола View:
```swift
public protocol View {
    associatedtype Body : View

    @ViewBuilder var body: Self.Body { get }
}
```

## Зачем?
Потребность в использовании @ViewBuilder возникает тогда, когда наше представление (нечто возвращающее some View) имеет внутреннее ветвление, и в кейсах этого ветвления возвращаются разные по структуре представления.

Например давайте попробуем создать структуру, в которой подменим протокол View, на наш кастомные протокол, не использующий @ViewBuilder:
```swift
public protocol ViewWithoutViewBuilder {
    associatedtype Body : View

    var body: Self.Body { get }
}
```
Попробуем подписать наш, ContentView под этот протокол, то получим следующее:
```swift
struct ContentView: ViewWithoutViewBuilder {
    var body: some View {
        Text("Hello, world!")
    }
}
```
Шок, но ошибок никаких нет. Давайте немного усложним наш стартовый ContentView, и добавим ветвление того, что мы хотим показать нашему юзеру:
```swift
struct ContentView: ViewWithoutViewBuilder {
    @State var isGreeting: Bool = true
    var body: some View {
        if isGreeting {
            Text("Hello, world!")
        } else {
            Text("See you!")
        }
    }
}
```
Jшибок никаких нет.
Давайте продолжим пробовать различные вариации, и в одной из таковых попробуем поменять один из возвращаемых типов:
```swift
struct ContentView: ViewWithoutViewBuilder {
    @State var isGreeting: Bool = true
    var body: some View {
        if isGreeting {
            Text("Hello, world!") // Error: Branches have mismatching types 'Text' and 'VStack<TupleView<(Image, Text)>>'
        } else {
            VStack {
                Image(systemName: "hand.wave")
                Text("See you!")
            }
        }
    }
}
```
Настал момент, когда появилась ошибка. Что у нас говорит компилятор?
> Ветви имеют несовпадающие типы 'Text' и 'VStack<TupleView<(Image, Text)>>'

Другими словами: "А вот это уже слишком сложно, вернуть надо разное, что мне вернуть-то???
Вот и практически доказанный ответ по области применения @ViewBuilder.

# Создание своего View
Cоздадим кастомный стек состоящий из VStack и HStack. Используя @Enviroment отслеживающий sizeClass, мы в случае поворота экрана, хотим чтоб тип возвращаемой ориентации стека  изменялся.
```swift
struct VorHStack<Content: View>: View {
    
    @Environment(\.horizontalSizeClass) var horizontalSizeClass

    let content: Content

    init(@ViewBuilder _ content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        if horizontalSizeClass == .compact {
            VStack { content }
        } else {
            HStack { content }
        }
    }
}
```
Исходя из написанного кода возможно следующее применение:
```swift
struct ContentView: View {
    
    var body: some View {
        VorHStack {
            Text("Hello, World!")
            Text("Hello, World2!")
        }
    }
}
```
Работать это будет отлично, но мы можем это сделать более компактным. Но как?
Для этого давайте провалимся внутрь одного из выше указанных контейнеров:
```swift
struct VStack<Content> : View where Content : View {

    @inlinable public init(
        alignment: HorizontalAlignment = .center,
        spacing: CGFloat? = nil,
        @ViewBuilder content: () -> Content
    )
    
    public typealias Body = Never
}
```
Мы видим, что "content", требуемый дефолтным инициализатором, принимает в себя "() -> Content", а так же у него тоже явно указан @ViewBuilder. Раз уж мы сегодня проваливались почти везде, то почему до сих пор не провалились в сам @ViewBuilder? Погнали!
```swift
@resultBuilder public struct ViewBuilder {

    /// Создает выражение внутри построителя.
    public static func buildExpression<Content>(_ content: Content) -> Content where Content : View

    /// Создает пустое представление из блока, не содержащего операторов.
    public static func buildBlock() -> EmptyView

    /// Передает одно представление, написанное как дочернее представление, в неизмененном виде.
    ///
    /// Пример одного представления, написанного как дочернее представление:
    /// `{ Text("Hello") }`.
    public static func buildBlock<Content>(_ content: Content) -> Content where Content : View

    public static func buildBlock<each Content>(_ content: repeat each Content) -> TupleView<(repeat each Content)> where repeat each Content : View
}
```
Здесь новым для нас будет атрибут `@resultBuilder`. Им помечают все создаваемые конструкторы. Пролистав мы добрались до вариации функции buildBlock() и обнаружили, что тут указан `each Content` позволяющий передавать произвольное количество представлений и получать в качестве возвращаемого значения кортеж представлений типа TupleView, созданный из быстрого списка значений View.

## Источники
- [@ViewBuilder Что? Зачем? Когда?](https://habr.com/ru/articles/761722/)
