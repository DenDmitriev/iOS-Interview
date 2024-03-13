# Keychain
Apple Keychain - это очень популярный и функциональный инструмент Swift, который использует каждый пользователь iOS и MacOS. Его можно использовать для сохранения паролей, безопасных заметок, сертификатов и т. д. В целом, Keychain - это зашифрованная база данных с довольно сложным и надежным API.

Причины:
- Хранить конфиденциальные пользовательские данные, такие как пароли или учетные данные для входа
- Хранить ключи для API которые используют их для работы, наприер: GoogleMapsAPI.
 
![1c9e8103-fae2-45f4-832c-c528d2e0c2f6](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/8ce5ba3a-e2f6-4127-891a-3e3717e027a1)

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

## Источники:
- [iOS Data Persistence in Swift](https://iosapptemplates.com/blog/ios-development/data-persistence-ios-swift)
