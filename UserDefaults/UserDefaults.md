# UserDefaults
 UserDefaults использоваться для хранения информации по умолчанию о приложении или его пользователях. Например, пользователь должен иметь возможность указать персональные настройки такие как валюту по умолчанию, которую вы должны отображать при представлении некоторых данных. Эти предпочтения должны быть специфичными для приложения и должны быть постоянными между запусками приложений.

 UserDefaults представляется классом, который предоставляет интерфейс для сохранения пар ключ-значение и дает нам синхронное чтение и запись (на уровне кэша) и асинхронный уровень постоянства.

 Типы для хранения:
 - NSData
 - NSString
 - NSNumber
 - NSDate
 - NSArray
 - NSDictionary

 ### Сохранение
 Для сохранения используется метод `set(_:forKey:)`. Устанавливает значение указанного ключа по умолчанию.

 ### Чтение
 Для чтения используется метод `value(forKey:)`. Возвращает значение для свойства, идентифицированного заданным ключом.

 ### Значения по умолчанию
 Все эти методы возвращают значение по умолчанию, если указанный ключ еще не был установлен.
 - Bool это false
 - Int это 0
 - String возвращает nil по умолчанию, поэтому возвращаемый тип является String?

 ## Использование
 UserDefaults иммет стандартный `standard` экземпляр типа который вы используете как общий объект по умолчанию, в котором мы можем считывать и записывать пары ключ-значение.

 ### Ключ
 Чтобы сохранить значение, вам нужно объявить его ключ. Ключ - это уникальная строка, которая идентифицирует значение в UserDefaults.

 Давайте рассмторим сервис для работы с этими данными:
 ```swift
 class UserRepository {
     enum Key: String, CaseIterable {
         case name, avatarData

         // Key for property
         func make(for userID: String) -> String {
             return self.rawValue + "_" + userID
         }
     }

     let userDefaults: UserDefaults

     // MARK: - Lifecycle
     init(userDefaults: UserDefaults = .standard) {
         self.userDefaults = userDefaults
     }

     // MARK: - API
     func storeInfo(forUserID userID: String, name: String, avatarData: Data) {
         saveValue(forKey: .name, value: name, userID: userID)
         saveValue(forKey: .avatarData, value: avatarData, userID: userID)
     }

     func getUserInfo(forUserID userID: String) -> (name: String?, avatarData: Data?) {
         let name: String? = readValue(forKey: .name, userID: userID)
         let avatarData: Data? = readValue(forKey: .avatarData, userID: userID)
         return (name, avatarData)
     }

     func removeUserInfo(forUserID userID: String) {
         Key
             .allCases
             .map { $0.make(for: userID) }
             .forEach { key in
                 userDefaults.removeObject(forKey: key)
             }
     }

     // MARK: - Private
     private func saveValue(forKey key: Key, value: Any, userID: String) {
         userDefaults.set(value, forKey: key.make(for: userID))
     }

     private func readValue<T>(forKey key: Key, userID: String) -> T? {
         return userDefaults.value(forKey: key.make(for: userID)) as? T
     }
 }
 ```

 ## Преимущества
 - UserDefaults прост в использовании, с простым API
 - Потокобезопасен (вы можете читать и записывать значения из любого потока), не беспокоясь о синхронизации
 - UserDefaults совместно используется приложением и расширениями приложения

 ## Ограничения
 - Можно легко переопределить значение для этого же ключа (коллизии ключей)
 - UserDefaults не зашифрован
 - Модульное тестирование пользовательских значений по умолчанию может происходить с некоторыми ложными срабатываниями.
 - UserDefaults могут быть изменены глобально, из любой точки приложения, поэтому вы можете легко работать в непоследовательных состояниях.

 ## Источники:
 - [iOS Data Persistence in Swift](https://iosapptemplates.com/blog/ios-development/data-persistence-ios-swift)
