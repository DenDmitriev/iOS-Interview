# Зачем нужны compressionResistancePriority u contentHuggingPriority?

Для некоторых UIView мы не всегда можем применить конкретные значения. Например, если у нас есть UIButton с жестко заданным значением, при локализации текст может быть обрезан.
Чтобы справиться с этим, не требуя изменения приоритета, мы можем использовать другой набор функций, чтобы избежать такого поведения. Двумя из них являются Content Hugging и Content Resistance.

 - Content hugging priority - приоритет сопротивления превышения intrinsic content size (не хочу чтобы была больше)
 - Content compression resistance priority - приоритет сопротивления занижения intrinsic content size (не хочу чтобы была меньше)

<img width="354" alt="42c420886adf5adad1d49cf3481a61d1" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/d50c7404-4b7e-4cf2-bbea-73ebbf272909">

Вместо того чтобы устанавливать ограничение ширины, чтобы избежать обрезки, можно применить сопротивление избежав сжатия. Другой текст локализации расширит его.

```swift
button.setContentHuggingPriority(.required, for: .horizontal)
view.setContentHuggingPriority(.defaultLow, for: .vertical)
```
