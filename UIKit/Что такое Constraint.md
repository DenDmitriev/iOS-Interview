# Что такое constraint?

Сonstraint - это ограничение или правило, позволяющее операционной системе размещать компонент пользовательского интерфейса. 
В UIKit есть 2 основных способа установки ограничений: 
 - [Storyboard](#storyboard)
 - [Кодом](#кодом)

## Storyboard

<img width="277" alt="EEYEi" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/29675c5f-57b7-4ebf-9a0c-4f829a164faf">


## Кодом
Например:
```swift
let newView = UIView()
newView.backgroundColor = UIColor.red
view.addSubview(newView)
newView.translatesAutoresizingMaskIntoConstraints = false
```
И далее:
```swift
let widthConstraint = NSLayoutConstraint(item: newView, attribute: .width, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: 100)
```
либо
```swift
view.addConstraints([
        newView.widthAnchor.constraint(equalToConstant: 100),
        newView.heightAnchor.constraint(equalToConstant: 100),
        newView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
        newView.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```
