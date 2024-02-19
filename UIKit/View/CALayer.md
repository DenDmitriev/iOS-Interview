# Наследники CALayer

 - [CATextLayer](https://developer.apple.com/documentation/quartzcore/catextlayer)
 - [CAShapeLayer](https://developer.apple.com/documentation/quartzcore/cashapelayer)
 - [CAGradientLayer](https://developer.apple.com/documentation/quartzcore/cagradientlayer)
 - [CAScrollLayer](url)
 - [AVPlayerLayer](url)
 - [CAReplicatorLayer](url)
 - [CATransformLayer](url)
 - [CAEmitterLayer](url)

## CATextLayer
CATextLayer обеспечивает простой, но быстрый рендеринг обычного текста или приписываемых строк. В отличие от UILabel, CATextLayer не может иметь назначенный UIFont, только CTFont или CGFont.

```swift
let textLayer = CATextLayer()
textLayer.string = "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce auctor arcu quis velit congue dictum."
textLayer.frame = viewForTextLayer.bounds

textLayer.font = CTFontCreateWithName("Helvetica" as CFString, Constants.baseFontSize, nil)
textLayer.fontSize = 24.0

textLayer.foregroundColor = UIColor.darkGray.cgColor
textLayer.isWrapped = true
textLayer.alignmentMode = .left
textLayer.truncationMode = .end

textLayer.contentsScale = UIScreen.main.scale
```

## CAShapeLayer
CAShapeLayer обеспечивает аппаратное ускорение рисования всех видов 2D-форм и включает в себя дополнительные функции, такие как цвета заливки и обводки, шапки линий, узоры и многое другое.

Для начала используйте UIBezierPath для создания закруглого прямоугольника, который затем окрашивается в красный цвет с помощью CAShaperLayer. Помните, что CALayer находится под UIKit, поэтому вам нужно использовать CGColor, а не UIColor.
```swift
let layer = CAShapeLayer()
layer.path = UIBezierPath(roundedRect: CGRect(x: 64, y: 64, width: 160, height: 160),
                          cornerRadius: 50).cgPath
layer.fillColor = UIColor.red.cgColor
view.layer.addSublayer(layer)
```

## CAGradientLayer
```swift
import PlaygroundSupport
import UIKit

class PreviewViewController: UIViewController {

    let gradientLayer = CAGradientLayer()

    let colors: [CGColor] = [
        UIColor.red.cgColor,
        UIColor.green.cgColor,
        UIColor.blue.cgColor,
        UIColor.purple.cgColor
    ]

    let locations: [Float] = [0, 1 / 6.0, 1 / 3.0, 0.5, 2 / 3.0, 5 / 6.0, 1.0]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let viewForGradientLayer = UIView(frame: view.frame)
        viewForGradientLayer.backgroundColor = .red
        setUpGradientLayer()
        viewForGradientLayer.layer.addSublayer(gradientLayer)
        view.addSubview(viewForGradientLayer)
        
    }
    
    func setUpGradientLayer() {
      gradientLayer.frame = view.bounds
      gradientLayer.colors = colors
      gradientLayer.locations = locations.map { NSNumber(value: $0) }
      gradientLayer.startPoint = CGPoint(x: 0.5, y: 0.0)
      gradientLayer.endPoint = CGPoint(x: 0.5, y: 1.0)
    }
}

let preview = PreviewViewController()
PlaygroundPage.current.liveView = preview
```
<img width="326" alt="Снимок экрана 2023-10-25 в 10 20 23" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/6d918cae-3b04-4cfb-8056-a18df8e66ef4">


## CAScrollLayer
Класс CAScrollLayer - это подкласс CALayer, который упрощает отображение части слоя. Размер прокручиваемой области CAScrollLayer определяется компоновкой его подслоев. Видимая часть содержимого слоя указываются путем указания источника в качестве точки или прямоугольной области содержимого, которое будет отображаться. CAScrollLayer не обеспечивает обработку событий с помощью клавиатуры или мыши, а также не предоставляет видимых скроллеров.
```swift
@IBOutlet weak var scrollingView: ScrollingView!

var scrollingViewLayer: CAScrollLayer {
    // swiftlint:disable:next force_cast
    return scrollingView.layer as! CAScrollLayer
}

@IBAction func panRecognized(_ sender: UIPanGestureRecognizer) {
    var newPoint = scrollingView.bounds.origin
    newPoint.x -= sender.translation(in: scrollingView).x
    newPoint.y -= sender.translation(in: scrollingView).y
    sender.setTranslation(.zero, in: scrollingView)
    scrollingViewLayer.scroll(to: newPoint)

    if sender.state == .ended {
      UIView.animate(withDuration: 0.3) {
        self.scrollingViewLayer.scroll(to: CGPoint.zero)
      }
    }
}
```

## Ичтоники:
- [CALayer Tutorial for iOS: Getting Started](https://www.kodeco.com/10317653-calayer-tutorial-for-ios-getting-started)
