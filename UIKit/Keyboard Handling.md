# Работы с клавиатурой в iOS

## Как убрать эту клавиатуру?
Когда вы нажимаете на текстовое поле, появляется клавиатура, которая позволяет вам вводить все, что вы хотите. Теперь, как только вы закончите писать, нет кнопки "Сделано" по умолчанию, которая отключит клавиатуру, нажатие за пределами клавиатуры тоже ничего не сделает. Это как если бы iOS говорила: "Разве не достаточно, чтобы я автоматически выснул клавиатуру для вас? Придумайте способ, чтобы это исчезло самостоятельно прямо сейчас!".

К сожалению, Apple еще не сделала отключение клавиатуры по умолчанию. Поэтому мы, разработчики, должны обрабатывать этот сценарий программно в нашем приложении. К счастью, есть несколько способов, с помощью которых мы можем достичь этой функциональности, и это тоже в нескольких строках кода. Какой способ реализации в вашем коде может зависеть от вашего точного сценария.

### Жеста касания (Tap Gesture)
Это самый быстрый способ реализовать скрытие клавиатуры. Просто установите жест касания на главном представлении и подключите этот жест к функции, которая вызывает `view.endEditing`.
> `endEditing(_:)` Сообщает, что представление (или одно из встроенных текстовых полей) отпускает статус первого ответчика.

Вот и все! Теперь, когда вы нажмете на текстовое поле, и появится клавиатура, просто нажмите на любое внешнее место на представлении, и ваша клавиатура будет закрыта.

![0*JVjKRFvODJm5XBgt](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/0b703397-258f-4be3-964e-e7d43c773935)


```swift
class ViewController: UIViewController {

    @IBOutlet weak var emailTF: UnderlinedTextField!
    @IBOutlet weak var passwordTF: UnderlinedTextField!
    
    static func storyboardInstance() -> Solution1VC? {
        let storyboard = UIStoryboard(name: Storyboard.main ,bundle: nil)
        return storyboard.instantiateViewController(withIdentifier: "Solution1VC") as? Solution1VC
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Мы вызываем нашу функцию обработки клавиатуры, как только представление загружается.
        initializeHideKeyboard()
    }

}

extension ViewController {
    func initializeHideKeyboard(){
        // Объявите жест касания, который будет запускать нашу функцию dismissMyKeyboard().
        let tap: UITapGestureRecognizer = UITapGestureRecognizer(
            target: self,
            action: #selector(dismissMyKeyboard))
        
        // Добавьте этот распознаватель жестов касания в родительское представление.
        view.addGestureRecognizer(tap)
    }
    
    @objc func dismissMyKeyboard(){
        // endEditing заставляет представление (или одно из встроенных текстовых полей) отказываться от статуса первого ответчика.
        // Короче говоря - уберите активную клавиатуру.
        view.endEditing(true)
    }
}
```

### С помощью клавиши ввод клавиатуры (Return Key)
Еще один отличный вариант - использовать эту нефункциональную клавишу Return/Done/Continue keyboard. Он просто сидит и ничего не делает, если мы не указали какое-то пользовательское поведение в функции textFieldShouldReturn.
> textFieldShouldReturn Спрашивает делегата, должно ли текстовое поле обрабатывать нажатие кнопки возврата.

1. Здесь вам сначала нужно установить делегатов для текстовых полей `UITextField`.
2. Установите тег (tag) для текстовых полей. Что такое тег, спросите вы? В документации Apple говорится - целое число, которое вы можете использовать для идентификации объектов представления в вашем приложении. Настройка тега является необязательной и не требуется, если у вас есть только 1 текстовое поле. Но здесь я назначаю тег текстовым полям, увеличивая их значение на 1, и в том порядке, в котором они размещаются на экране. Это поможет нам определить текстовые поля в коде.
3. `textFieldShouldReturn(_:)` убирает клавиатуру, когда пользователь нажимает клавишу Return на ней. Следовательно, здесь мы проверяем - Есть ли какое-либо другое текстовое поле в представлении, тег которого на 1 больше, чем тег текущего текстового поля, на котором была нажата клавиша возврата. Если да →, то переместите курсор в следующее текстовое поле. Если нет → Убрать клавиатуру

Также не беспокойтесь о том, что поле пароля не видно при наборе текста. Мы решим эту проблему, когда обсудим перемещение текстовых полей на видимость в соответствии с клавиатурой в более позднем разделе этой статьи.

![0*RatHAoXgoEHuGKaf-2](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/6d67eaea-8225-4d80-95fa-245659035f47)


```swift
class ViewController: UIViewController {

    @IBOutlet weak var emailTF: UnderlinedTextField!
    @IBOutlet weak var passwordTF: UnderlinedTextField!
    
    static func storyboardInstance() -> Solution2VC? {
        let storyboard = UIStoryboard(name: Storyboard.main ,bundle: nil)
        return storyboard.instantiateViewController(withIdentifier: "Solution2VC") as? Solution2VC
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        emailTF.delegate = self
        passwordTF.delegate = self
        
        emailTF.tag = 1
        passwordTF.tag = 2
    }

}

extension ViewController: UITextFieldDelegate {

    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        // Проверьте, есть ли в представлении какое-либо другое текстовое поле, тег которого на +1 больше, чем текущее текстовое поле, на котором была нажата клавиша return/done.
        // Если Да - переместите курсор в следующее текстовое поле.
        // Если Нет - отключить клавиатуру.
        if let nextField = self.view.viewWithTag(textField.tag + 1) as? UITextField {
            nextField.becomeFirstResponder()
        } else {
            textField.resignFirstResponder()
        }
        return false
    }
}
```

### Через панель инструментов для Number Pad
Вышеуказанное решение с `textFieldShouldReturn(_:)` отлично работает, но, к сожалению, на iOS Number Pad нет ключа "Return". Чтобы преодолеть наш барьер здесь в случае Number Pad, мы можем добавить простую панель инструментов над нашей клавиатурой с кнопкой "Готово". Эта кнопка "Done" вызовет ту же функцию, которую мы использовали в нашем методе жестов Tap выше, и отключит нашу клавиатуру. Чтобы сохранить согласованность по полям, вы можете использовать панели инструментов в приложении в качестве общего паттера для отключения клавиатуры.

![0*5ku5f1a4PMsB6dzX](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/3de26987-ccab-4071-a07d-aaf7b1ff1465)


```swift

class ViewController: UIViewController {

    @IBOutlet weak var pinTF: UnderlinedTextField!
    
    static func storyboardInstance() -> Solution4VC? {
        let storyboard = UIStoryboard(name: Storyboard.main ,bundle: nil)
        return storyboard.instantiateViewController(withIdentifier: "Solution4VC") as? Solution4VC
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupToolbar()
    }
    
    func setupToolbar(){
        // Создание toolbar
        let bar = UIToolbar()
        
        // Создайте кнопку «Done» с действием, которое активирует нашу функцию для отключения клавиатуры.
        let doneBtn = UIBarButtonItem(title: "Done", style: .plain, target: self, action: #selector(dismissMyKeyboard))
        
        // Создайте гибкий элемент пространства, чтобы мы могли добавить его на панель инструментов и расположить кнопку «Готово».
        let flexSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
        
        // Добавьте элементы кнопки «Создать» на панель инструментов.
        bar.items = [flexSpace, flexSpace, doneBtn]
        bar.sizeToFit()
        
        // Добавьте панель инструментов в наше текстовое поле
        pinTF.inputAccessoryView = bar
    }
    
    @objc func dismissMyKeyboard(){
        view.endEditing(true)
    }
}
```

Я нашел хорошую статью о расширении для создания панели инструментов Done Button, которая поможет вам уменьшить дублирование кода и подключить одну и ту же функцию в приложении - [SWIFT — Добавить кнопку "Готово" с помощью UIToolbar](https://medium.com/swift2go/swift-add-keyboard-done-button-using-uitoolbar-c2bea50a12c7)

Если вам нужно больше информации и способов борьбы с увольнением с клавиатуры, я нашел эту отличную статью на Medium, вы можете найти ее здесь - [https://medium.com/@KaushElsewhere/how-to-dismiss-keyboard-in-a-view-controller-of-ios-3b1bfe973ad1](https://medium.com/@KaushElsewhere/how-to-dismiss-keyboard-in-a-view-controller-of-ios-3b1bfe973ad1)

## Источники:
- [Keyboard Handling in iOS — Swift 5](https://nqaze.medium.com/keyboard-handling-in-ios-swift-5-8b60d602a8f)
