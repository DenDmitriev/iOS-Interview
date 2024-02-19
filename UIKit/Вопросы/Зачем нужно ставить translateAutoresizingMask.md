# Зачем нужно ставить translateAutoresizingMask


```swift
view.translatesAutoresizingMaskIntoConstraints = true
```
Система создает набор статических ограничений используя свойства фрейма, границ или центра представления, на основе фрейма в Auto Layout.

```swift
view.translatesAutoresizingMaskIntoConstraints = false
```
Auto Layout для динамического расчета размера и положения view, по набору ограничений для представления.

По умолчанию свойство установлено значение true для любого создаваемого вами представления. Если вы добавляете представления в Interface Builder, система автоматически устанавливает это свойство в false.
