# Чем отличается leading/trailing от leftAnchor/rightAnchor?

Если вы хотите закрепить кнопку на левом краю ее родителя, вы можете написать код следующим образом:

```swift
button.leftAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leftAnchor).isActive = true
```
