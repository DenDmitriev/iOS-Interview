## Offscreen Rendering

- Чтобы избежать расходов CPU, нужно перевести методы где происходит например скругление или обрезание изоюражения на `CAShapeLayer` которые используют GPU.
- Загружать изображения сразу нужного размера, чтоб избежать изменения размера