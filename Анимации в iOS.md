# Анимации в iOS

В нашем разговоре об анимациях нельзя обойтись без нескольких фундаментальных вещей. Определим их:

**Core Animation** — фреймворк для работы с базовыми классами анимации: 
- `CABasicAnimation`,
- `CAKeyFrameAnimation`,
- `CATransition`,
- `CAAnimationGroup`.

Использование `Core Animation` полностью автоматизировано: не нужно создавать циклов и таймеров, чтобы сделать анимацию.

**CALayer** — набор классов, управляющих анимацией корневого слоя объекта. Классы получают доступ к слою и применяют к нему одно из свойств. Среди таких свойств — размер и положение слоя, фоновый цвет слоя, тень, скруглённые углы и тому подобное. 

Чтобы при создании анимации указать путь к свойствам `CALayer`, используется метод `animationWithKeyPath` или свойство `keyPath`. Последние назначаются строковым видом @«название_ключа».

Явная модель анимации требует создания объекта анимации и постановки начальных и конечных значений и будет протекать плавно от одного значения к другому. Анимация не начнётся, пока не будет добавлена к слою.

## CABasicAnimation
Обеспечивает простую интерполяцию между значениями для слоя. Например, с этим классом мы можем перемещать слой из одной точки в другую, менять значение прозрачности от одного к другому и т.п. С помощью класса можно сделать анимации для привлечения внимания пользователя к определённому объекту на экране или показать обучающую информацию в виде анимации.
```swift
let theAnimation = CABasicAnimation(keyPath: "position");

theAnimation.fromValue = [NSValue(CGPoint: CGPointMake(screenWidth/2, self.animationButton.frame.origin.y))]
theAnimation.toValue = [NSValue(cgPoint: CGPoint(x: 100.0, y: 100.0))]
theAnimation.duration = 3.0;
theAnimation.autoreverses = false //true - возвращает в исходное значение либо плавно, либо нет
theAnimation.repeatCount = 2
theLayer.addAnimation(theAnimation, forKey: "animatePosition");
```
В верхней анимации параметр autoreverses = YES, в нижней — NO. То есть позиция либо не возвращается к первоначальному значению, либо плавно возвращается.
![fd8d591a6dd140c18e069ae2ae5a2579](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/853083dc-40a7-49a7-b491-e3f49cd0954f)
![e7d4c18a3a924cd7957d56e2f559de88](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/f1cdae30-6a58-42b5-ad8f-5a2640bd59d8)


## CAKeyFrameAnimation
Обеспечивает изменение значений свойства слоя по величинам, которые задаются в массиве. Для инициализации используется метод animationWithKeyPath с указанием свойства, которое нужно изменить.Также указываем массив значений, которые будут представляться на каждом этапе анимации. Мы можем задать несколько значений, в которые будет перемещаться слой — получается гораздо интереснее простого изменения позиции из одной точки в другую, как было в примерах выше.
```swift
let animation = CAKeyframeAnimation()

let pathArray = [[NSValue(cgPoint: CGPoint(x: 10.0, y: 10.0))],  [NSValue(cgPoint: CGPoint(x: 100.0, y: 100.0))], [NSValue(cgPoint: CGPoint(x: 10.0, y: 100.0))], [NSValue(cgPoint: CGPoint(x: 10.0, y: 10.0))], ]

animation.keyPath = "position"
animation.values = pathArray
animation.duration = 5.0
self.label.layer.add(animation, forKey: "position")
```
![557a34e71d0d46bc8dbd4d6a1e1566de](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/17657a8a-2c27-465c-9041-c488c6eb424d)


## CATransition
Обеспечивает эффект перехода, который влияет на контент всего слоя. Он исчезает, толкает или раскрывает содержимое слоя при анимации. CATransition можно использовать для перехода между UIView или для изменения переходов между UIViewController: плавное появление, появление с разных сторон, появление поверх текущего контента, выталкивание текущего контента.
```swift
let transition = CATransition()

transition.duration = 2.35
transition.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
transition.type = kCATransitionFade
self.navigationController?.view.layer.add(transition, forKey: nil)
self.navigationController?.pushViewController(ViewController(), animated: false)
```

Типы для transition.type:

 - kCATransitionFade. Содержимое слоя исчезает, как только он становится видимым или невидимым;
 - kCATransitionMoveIn. Содержимое слоя скользит поверх текущего контента. С этим типом используются простые подтипы transition.subtype;
 - kCATransitionPush. Содержимое слоя выталкивает существующий контент. С этим типом используются простые подтипы transition.subtype;
 - kCATransitionReveal. Содержание слоя раскрывается постепенно в направлении, указанном подтипом для перехода. С этим типом используются простые подтипы transition.subtype.

Подтипы для transition.subtype:

 - kCATransitionFromRight. Представление начинается справа;
 - kCATransitionFromLeft. Представление начинается слева;
 - kCATransitionFromTop. Представление начинается сверху;
 - kCATransitionFromBottom. Представление начинается снизу;

## CAAnimationGroup
Позволяет создать массив анимированных объектов, которые сгруппируются вместе и будут работать одновременно.
```swift
var animations = [CABasicAnimation]()

var posAnimation = CABasicAnimation(keyPath: "position")
posAnimation.duration = 1.0
posAnimation.autoreverses = true
posAnimation.fromValue = [NSValue(cgPoint: CGPoint(x: self.animationButton.frame.origin.x, y: self.animationButton.frame.origin.y))]
posAnimation.toValue = [NSValue(cgPoint: CGPoint(x: 100.0, y: 100.0))]
animations.append(posAnimation)
        
var heightAnimation = CABasicAnimation(keyPath: "bounds.size")

heightAnimation.autoreverses = true
heightAnimation.duration = 10.0
heightAnimation.fromValue =  [NSValue(cgSize: CGSize(width: self.animationButton.frame.size.width, height: self.animationButton.frame.size.height))]
heightAnimation.toValue = [NSValue(cgSize: CGSize(width: self.animationButton.frame.size.width, height: 200.0))]
heightAnimation.beginTime = 5.0
animations.append(heightAnimation)
        
let group = CAAnimationGroup()

group.duration = 10.0 
group.animations = animations
self.animationButton.layer.addAnimation(group, forKey: nil)
```
Здесь параметр BeginTime указывает, через какой промежуток после старта запустится анимация.

Анимация изменения позиции начинается сразу после старта, а анимация изменения высоты кнопки — с пятой секунды.
![8410bd258c374b6292029e7cef61e288](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/d8e73fe0-9b59-45ce-b6d1-6eec880b42f0)

## Удаление анимаций

Мы можем удалить как конкретную анимацию у layer, так и все анимации.

### Удаляем конкретную анимацию

При создании анимации мы указывали ключ @«animateOpacity», по которому сможем получить доступ к ней. Чтобы удалить эту анимацию, делаем следующее: 

```swift
removeAnimationForKey(«animateOpacity»)
```

### Удаляем все анимации

Чтобы удалить все анимации у layer, нужно отправить сообщение removeAllAnimations:

```swift
theLayer.removeAllAnimations()
```

## Блоки анимации
Есть заранее заготовленные блоки, в которых можно проигрывать нужную анимацию (изменение прозрачности, позиции, размеров). Таких блоков два: animations и completion. Определим их назначение:

Блок animations — блок, в котором код будет выполняться анимационно
Блок completion — блок, в котором код выполняется после того, как выполнится блок animations

Блоки анимаций нужно использовать при смене одного значения слоя (объекта) на другое с последующим сохранением нового значения. В блоках мы можем менять как свойства слоя, так и свойства самого объекта.

Альфа кнопки изменится из текущего состояние в конечное, которое указано в блоке, за 3 секунды.

```swift
UIView.animateWithDuration(3.0, animations: { 
self.theButton.alpha = 0.0 
})
```
![a18a035c23504b50868449b2bdd0f313](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/1dc927f6-c1b9-49b8-9b1f-5ea4d3ab7ccb)

Здесь первый параметр — скорость, с которой будет воспроизводиться анимация;
второй параметр — задержка;
третий параметр — опции, то есть в каком виде будет проигрываться анимация.

Опции проигрывания анимации:

 - `UIViewAnimationCurveLinear` — анимация выполняется на постоянной скорости в течение заданного времени;
 - `UIViewAnimationCurveEaseOut` — анимация начинается быстро и замедляется ближе к концу;
 - `UIViewAnimationCurveEaseIn` — анимация начинается медленно и ускоряется ближе к концу;
 - `UIViewAnimationCurveEaseInOut` — анимация начинается медленно, ускоряется и снова замедляется.

Есть блоки анимаций, в которых осуществляются анимационные переходы между UIView или добавление элементов на UIView, как в примере ниже. В них используется блок transitionWithView, нужный для анимационного перехода или представления UIView или унаследованных от UIView объектов.

```swift
UIView.transition(with: self.view, duration: 1.5, options: .transitionFlipFromBottom, animations: { 
            self.view.addSubview(self.imageView)
}, completion: nil)
```
Вот что в этом случае получится: В этом примере мы добавляем картинку на UIView с анимацией:
![c0423221a11c46ba87f77c0c6b96868e-2](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/107b838b-f758-4dcc-a50f-4ff9011162fe)

Объект UIImageView отображает одно изображение или последовательность изображений в анимированном интерфейсе. Мы можем анимировать UIImageView без использования блоков и анимаций типа CABasicAnimation. У UIImageView есть свойства animationImages, animationDuration, animationRepeatCount, а это значит, что мы можем, передав в animationImages массив с картинками, которые нам нужно проиграть, стартануть анимацию UIImageView. 

```swift
var imgListArray :NSMutableArray = []
for countValue in 1...8 {
var strImageName : String = "chicken_normal_fly_\(countValue).png"
var image = UIImage(named:strImageName)
imgListArray.add(image as Any)
}

self.imageView.animationImages = imgListArray;
self.imageView.animationDuration = 0.25
self.imageView.startAnimating()
```

![67077e2f0318441391f61abe71112f22](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/db2afe13-14d0-49a0-9da9-ea82a82adca2)


[Источник](https://habr.com/ru/companies/livetyping/articles/319592/)
