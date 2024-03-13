# Keychain
Apple Keychain - это очень популярный и функциональный инструмент Swift, который использует каждый пользователь iOS и MacOS. Его можно использовать для сохранения паролей, безопасных заметок, сертификатов и т. д. В целом, Keychain - это зашифрованная база данных с довольно сложным и надежным API.

## Причины
Украденные данные из вашего приложения, взятые у вашего пользователя, будут на вашей ответственности. Поэтому некоторые данные нужно защищать, такие как:
- Конфиденциальные пользовательские данные, такие как пароли или учетные данные для входа
- Токены для API которые использет ваше приложение для работы, наприер: GoogleMapsAPI.
 
![1c9e8103-fae2-45f4-832c-c528d2e0c2f6](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/8ce5ba3a-e2f6-4127-891a-3e3717e027a1)

## Недостатки
- API был написан на Objective-C, и это может быть сложно для использования

## Отличия с другими способами
| Название | Шифрование |
| - | - |
| UserDefaults | ❌ |
| Keychain | ✔️ |

## Атрибуты данных
При чтении или сохранении данных нужно указывать тип данных в этом запросе
Значения, которые вы используете с ключом kSecClass:
- `kSecClassGenericPassword` Значение, указывающее элемент общего пароля.
- `kSecClassInternetPassword` Значение, указывающее элемент пароля Интернета.
- `kSecClassCertificate` Значение, указывающее элемент сертификата.
- `kSecClassKey` Значение, указывающее элемент криптографического ключа.
- `kSecClassIdentity` Значение, указывающее элемент идентификации.
> Поскольку API связки ключей был написан на Objective-C, то эти ключи являются типом CFString


В следующем фрагменте Swift мы можем увидеть простой запрос, который позволяет прочитать пароль из Keychain для данной учетной записи внутри данной службы:

```swift
struct KeychainItemStorable {
    let service: String
    let account: String
}
extension KeychainItemStorable {
    enum Error: Swift.Error {
        case passwordNotFound
        case unknownError(String)
        case invalidData
    }
    
    func readPassword() throws -> String {
        var query: [String : Any] = [:]
        query[kSecClass as String] = kSecClassGenericPassword
        query[kSecAttrService as String] = service
        query[kSecAttrAccount as String] = account as AnyObject?
        query[kSecMatchLimit as String] = kSecMatchLimitOne
        query[kSecReturnAttributes as String] = kCFBooleanTrue
        query[kSecReturnData as String] = kCFBooleanTrue
        
        // Try to fetch the existing keychain item that matches the query.
        var queryResult: AnyObject?
        let status = withUnsafeMutablePointer(to: &queryResult) {
            SecItemCopyMatching(query as CFDictionary, UnsafeMutablePointer($0))
        }
        
        switch status {
        case noErr:
            guard let items = queryResult as? [String: Any],
                  let valueData = items[kSecValueData as String] as? Data,
                  let valueString = String(data: valueData, encoding: .utf8) else
            { throw Error.invalidData }
            return valueString
        case errSecItemNotFound:
            throw Error.passwordNotFound
        default:
            throw Error.unknownError(status.description)
        }
    }
}
```

Сохранение паролей еще сложнее, так как собственный API Keychain не проводит различий между добавлением элемента и обновлением элемента. Принимая во внимание все это, в целом, рекомендуется использовать для этой цели некоторые внешние библиотеки с некоторыми удобными API, такими как Locksmith или KeychainAccess.

## Преимущества
- Все элементы, сохраненные в связке ключей, зашифрованы (свячка ключей является самым безопасным уровнем сохранения данных в iOS)
- Существует много оберток с открытым исходным кодом с удобными API Swift
- Еще раз безопасность
- Потокобезопасность

## Ограничения
- Нет удобного API для ключей, поэтому вы обычно в конечном итоге зависите от проекта с открытым исходным кодом.
- Трудно тестировать, если вы не создадите свою собственную обертку вокруг нее
- Скорость: чтение или сохранение большого блика информации в Keychain может быть медленным
- Не рекомендуется для хранения больших предметов

# Реализация обвертки
Давайте начнем с создания нового класса, который будет выполнять наши функции. Кроме того, давайте определим тип алиас для использования в словаре атрибутов, который нам понадобится.
```swift
final class KeychainManager {
  
   typealias ItemAttributes = [CFString : Any]
  
   static let shared = KeychainManager()
  
   private init() {}
}
```
Для каждого метода, который предоставляет наш менеджер, нам нужен параметр, который указывает тип класса элемента для взаимодействия.
Нам все еще нужно использовать тип CFString в наших функциях, потому что мы не знаем фактического значения константы. По этой причине нам нужно внедрить RawRepresentable в нашем перечислении ItemClass.

```swift
extension KeychainManager {
    enum ItemClass: RawRepresentable {
        typealias RawValue = CFString
        
        case generic
        case password
        case certificate
        case cryptography
        case identity
        
        init?(rawValue: CFString) {
            switch rawValue {
            case kSecClassGenericPassword:
                self = .generic
            case kSecClassInternetPassword:
                self = .password
            case kSecClassCertificate:
                self = .certificate
            case kSecClassKey:
                self = .cryptography
            case kSecClassIdentity:
                self = .identity
            default:
                return nil
            }
        }
        
        var rawValue: CFString {
            switch self {
            case .generic:
                return kSecClassGenericPassword
            case .password:
                return kSecClassInternetPassword
            case .certificate:
                return kSecClassCertificate
            case .cryptography:
                return kSecClassKey
            case .identity:
                return kSecClassIdentity
            }
        }
    }
}
```

Следующий шаг - определить некоторые ошибки для наших операций.
```swift
extension KeychainManager {
    enum KeychainError: Error {
        case invalidData
        case itemNotFound
        case duplicateItem
        case incorrectAttributeForClass
        case unexpected(OSStatus)
        
        var localizedDescription: String {
            switch self {
            case .invalidData:
                return "Invalid data"
            case .itemNotFound:
                return "Item not found"
            case .duplicateItem:
                return "Duplicate Item"
            case .incorrectAttributeForClass:
                return "Incorrect Attribute for Class"
            case .unexpected(let oSStatus):
                return "Unexpected error - \(oSStatus)"
            }
        }
    }
}
```

Мы получим результат типа OSStatus от всех четырех операций. Поскольку мы хотим вернуть наши собственные ошибки, давайте создадим функцию-вспомогатель, которая преобразует этот тип результата в нашу пользовательскую ошибку.

```swift
extension KeychainManager {
    private func convertError(_ error: OSStatus) -> KeychainError {
        switch error {
        case errSecItemNotFound:
            return .itemNotFound
        case errSecDataTooLarge:
            return .invalidData
        case errSecDuplicateItem:
            return .duplicateItem
        default:
            return .unexpected(error)
        }
    }
}
```

Теперь мы находимся в хорошем месте для разработки наших методов API. Для всех четырех методов, которые будет предоставляться наш API, есть три параметра, которые нам нужно будет запросить:
- Класс элемента, с которым мы хотим работать.
- Ключ, который будет уникальным образом идентифицируть вместе с классом предмет в связке ключей.
- Набор необязательных атрибутов, которые помогут быстрее получить доступ к элементам.

## Сохранить элемент
```swift
func saveItem<T: Encodable>(_ item: T, itemClass: ItemClass, key: String, attributes: ItemAttributes? = nil) throws {
       // 1
       let itemData = try JSONEncoder().encode(item)

       // 2
       var query: [String: AnyObject] = [
          kSecClass as String: itemClass.rawValue,
          kSecAttrAccount as String: key as AnyObject,
          kSecValueData as String: itemData as AnyObject
       ]

       // 3
       if let itemAttributes = attributes {
          for (key, value) in itemAttributes {
              query[key as String] = value as AnyObject
          }
       }

       // 4
       let result = SecItemAdd(query as CFDictionary, nil)

       // 5
       if result != errSecSuccess {
          throw convertError(result)
       }
    }
```

1. Мы кодируем товар для хранения. По этой причине предмет для хранения должен соответствовать протоколу Encodable. Сочетая использование дженериков, мы пишем только одну функцию, которая может быть использована для хранения любого элемента до тех пор, пока реализован протокол Encodable.
2. Мы создаем словарь со всей информацией об элементе для хранения it.kSecClass, класс элемента. kSecAttrAccount, чтобы идентифицировать элемент вместе с классом элемента. kSecValueData, фактические данные для хранения в связке ключей.
3. Если мы получаем некоторые дополнительные атрибуты, мы добавляем их в запрос.
4. Мы выполняем операцию с ключевой связкой.
5. Мы проверяем результат. Если это что-то отличное от успеха, мы преобразуем ошибку в наши собственные ошибки API и отбрасываем ее обратно.

## Источники:
- [iOS Data Persistence in Swift](https://iosapptemplates.com/blog/ios-development/data-persistence-ios-swift)
- [Keychain Services in Swift](https://blorenzop.medium.com/keychain-services-in-swift-ecb9d6d5c6cd)
