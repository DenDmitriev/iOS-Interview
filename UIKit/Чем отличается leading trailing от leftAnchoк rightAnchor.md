# Чем отличается leading/trailing от leftAnchor/rightAnchor?

Если вы хотите закрепить кнопку на левом краю ее родителя, вы можете написать код следующим образом:

```swift
button.leftAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leftAnchor).isActive = true
```
Однако у этого есть проблема: в языках справа налево пользовательский интерфейс по большей части должен быть перевернут горизонтально, но ваша кнопка не будет двигаться - вы специально попросили прикрепить ее к левому краю независимо от настроек устройства пользователя.

Если это то, чего вы хотите, у вас нет проблем. Однако, если вы имели в виду «левый край для языков слева направо и правый край для языков справа налево», то вы должны использовать leadingAnchor вместо leftAnchor, вот так:

```swift
button.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor).isActive = true
```

Для того же эффекта на противоположном краю вы должны использовать trailingAnchor, а не rightAnchor. Опять же, если вы специально хотите, чтобы что-то появлялось справа, независимо от языка, то это не применимо.

Источники:
- [What’s the difference between leading, trailing, left, and right anchors?](https://www.hackingwithswift.com/example-code/uikit/whats-the-difference-between-leading-trailing-left-and-right-anchors)
