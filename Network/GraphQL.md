# GraphQL
GraphQL – это язык запросов API, определяющий спецификации, по которым клиентское приложение должно запрашивать данные с удаленного сервера. Вы можете использовать GraphQL в вызовах API для определения запроса, не полагаясь на серверное приложение.

# REST vs GraphQL
Популярный способ организации клиент-серверного взаимодействия — REST API. Но у инструмента есть альтернативы. Одна из них — GraphQL. Разберемся в неочевидных преимуществах, посмотрим на недостатки, развенчаем несколько мифов и попытаемся ответить на основной вопрос — стоит ли переходить на GraphQL.

| | REST | GraphQL |
|-|-|-|
| Определение | Архетиктурный подход API | Язык запросов API |
| Взаимодействие | Каждый метод имеет свой URL | Один URL для всего |
| Методы | HTTP методы (GET, POST) | Schema c запросами (query) и мутации (mutation) |
| Данные | Чистые без обверток | Типы данных: ID, String, Boolean, Int, Float, Enum  |

А теперь подробнее:
GraphQL — язык, в отличие от REST. С его помощью разработчик описывает взаимодействие клиента и сервера.

В отличие от REST, в GraphQL всего одна точка взаимодействия (endpoint) с бэкендом. Т. е. неважно, что вы хотите запросить или изменить: клиент будет взаимодействовать с одной конечной точкой, одним URL.

Основная часть GraphQL — схема (schema), которая описывает все типы, запросы и их взаимодействие, которые вы можете использовать в рамках текущего endpoint. Схема — не удобное дополнение, как, например, Swagger, а обязательная начальная точка проектирования любого GraphQL API. 

Есть два типа операций: запросы (query) и мутации (mutation). Первые отвечают за получение данных, вторые — за их изменение. Еще есть подписки (subscriptions), но в этой статье мы не будем их рассматривать.

Доступны базовые типы: 
- ID: уникальный идентификатор объекта, реализован как String;
- String: строки, например, имя пользователя;
- Boolean: булевое значение (true / false);
- Int: целочисленное (1, 2, 3, …);
- Float: число с плавающей точкой (1.5);
- Enum: перечисления.
Также можно описать собственные типы на основе базовых, пометить значения как обязательные или нет (знак «!» после указания типа), а также указать, что ожидается список, а не единичное значение (помечается скобками «[]», как в примере ниже). 

### Рассмотрим пример схемы
```GraphQL
type Query {

  user(id: ID!): User

  users: [User]

}

type Mutation {

  updateUser(id: ID!, name: String, age: Int): User

}

type User {

  id: ID!

  name: String!

  age: Int!

}
```

Здесь говорится, что существует запрос (`Query`) user, который принимает `id` типа `ID` и возвращает объект типа `User`, состоящий из трех полей: `id` типа `ID`, `name` типа `String` и `age` типа `Int`.  

Запрос `users` вернет список всех `User`. 

А мутация `updateUser` обновляет поля существующего пользователя по его `id`.

### Request Запрос данных
Клиент может запросить данные, например, по пользователю с ID 42, следующим запросом на языке GraphQL:
```GraphQL
query {

  user(id: 42) {

    id

    name

  }

}
```
В запросе мы не указали age, а следовательно, эти данные не будут возвращены на клиент, т.е. если они нам не нужны, мы сохраним немного байт и времени на пересылке данных между клиентом и сервером. 

## Базовый пример работы
Давайте посомтрим на тестовом сервере [Countries GraphQL API 🌎](https://countries.trevorblades.com/graphql) принцип работы.
Вы сможете посмотреть описание схемы на этой [странице](https://studio.apollographql.com/public/countries/variant/current/schema/sdl?selectedSchema=%23%40%21api%21%40%23). API schema:

```GraphQL
""""""
directive @cacheControl(maxAge: Int, scope: CacheControlScope) on FIELD_DEFINITION | OBJECT | INTERFACE

""""""
type Query {
  """"""
  continents: [Continent]
  """"""
  continent(code: String): Continent
  """"""
  countries: [Country]
  """"""
  country(code: String): Country
  """"""
  languages: [Language]
  """"""
  language(code: String): Language
}

""""""
type Continent {
  """"""
  code: String
  """"""
  name: String
  """"""
  countries: [Country]
}

""""""
type Country {
  """"""
  code: String
  """"""
  name: String
  """"""
  native: String
  """"""
  phone: String
  """"""
  continent: Continent
  """"""
  currency: String
  """"""
  languages: [Language]
  """"""
  emoji: String
  """"""
  emojiU: String
  """"""
  states: [State]
}

""""""
type Language {
  """"""
  code: String
  """"""
  name: String
  """"""
  native: String
  """"""
  rtl: Int
}

""""""
type State {
  """"""
  code: String
  """"""
  name: String
  """"""
  country: Country
}

""""""
enum CacheControlScope {
  """"""
  PUBLIC
  """"""
  PRIVATE
}

"""The `Upload` scalar type represents a file upload."""
scalar Upload
```

Важно поэкспериментировать с самими запросами, например, получить все страны, доступных в системе c полями `name` и `emoji`:
```HTTP
POST /graphql HTTP/1.1
Host: countries.trevorblades.com
Content-Type: application/json
Content-Length: 78

{"query":"query {\n  countries {\n    name\n    emoji\n  }\n}","variables":{}}
```
<img width="600" alt="Снимок экрана 2024-03-21 в 08 40 57" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/bca23e14-48f2-4e54-9efc-6003f48a50ce">

Или конкретную страну:
```HTTP
POST /graphql HTTP/1.1
Host: countries.trevorblades.com
Content-Type: application/json
Content-Length: 90

{"query":"query {\n  country(code: \"RU\") {\n    name\n    emoji\n  }\n}","variables":{}}
```
<img width="600" alt="Снимок экрана 2024-03-21 в 08 45 52" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/d95aef27-03cb-43cf-af44-24d7e86b223f">

Важный вывод, который мы должны сделать в роли фронтенд-разработчика — это то, что мы не написали ни строчки кода, а уже смогли поработать с бэкендом, увидеть доступные запросы, их параметры, выполнить их, проанализировать ответы и т. п. 

## Директивы (Directives)
Директивы помогают динамически изменять структуру и форму наших запросов с помощью переменных.
`@include` и `@skip` – две директивы, доступные в GraphQL

Пример директивы:

#### `@include`
Используется для включения определенных полей в запросе
@include(if: Boolean) — Включить поле, если значение boolean-переменной = true
```GraphQL
query GetFollowers($id: String) {
  user(id: $id) {
    fullname: name,
    followers: @include(if: $getFollowers) {
      name
      userHandle
      tweets
    }
  }
}

{
  "id": "1",
  "$getFollowers": false
}
```
Здесь $getFollowers имеет значение false, следовательно, поле имён подписчиков followers не будет включено в ответ.

#### `@skip`
Используется для пропуска определенных полей в запросе
@skip(if: Boolean) —  Пропустить поле, если значение boolean-переменной = true
```GraphQL
query GetFollowers($id: String) {
  user(id: $id) {
    fullname: name,
    followers: @skip(if: $getFollowers) {
      name
      userHandle
      tweets
    }
  }
}
{
  "id": "1",
  "$getFollowers": true
}
```
Здесь $getFollowers имеет значение true, следовательно, поле имен подписчиков followers будет пропущено, т. е. исключено из ответа.

## Недостатки GraphQL 
- Реализация бэкенда может быть затратнее. Это зависит от специфики, но,  например, GitHub рекомендует использовать REST для некоторых операций. Я часто слышал мнение, что GraphQL сложнее в оптимизации, т. к. один запрос может содержать любое число подзапросов и полей. Оптимизация всех комбинаций — головная боль.
- Некоторые операции, например, загрузку файлов или обработку ошибок  реализовать сложнее.
- Правильная архитектура схемы — отдельная нетривиальная задача. В REST можно создавать новые типы под каждый запрос. В GraphQL, если многое завязано на кэше, это неэффективно.

## Приемущества GraphQL
Ускоряет разработку на фронтенде. Это связано с несколькими факторами, упомянутыми выше:
- Самодокументированный API. Вы всегда знаете, что доступно на бэкенде, и можете сгенерировать нужные типы и методы (например, хуки в случае React).
- Возможность работать с бэкендом из песочницы.
- Так как есть вся информация о типах и их взаимосвязях, легко организовать работу с состоянием (например, кэш в Apollo). В более сложных сценариях использования подписок это позволит получать изменения через WebSocket без дополнительных запросов. Будет проще реализовать, например, многопользовательскую работу, когда важно получать измененные данные в ответ на чью-то активность.
- Продвинутый инструментарий. Есть полезные техники работы с запросами. Например, повторный запрос (refetching) данных в ответ на мутацию, возможность переиспользовать части схемы (fragments), возможность написания кастомных модификаторов кэша, тонкие настройки политик-запросов и т. п.
- Можно реализовать SSR (Server Side Rendering) и вшивать данные на сервере, чтобы потом не запрашивать их при старте приложения. Полезно для оптимизации начальной загрузки.

Также GraphQL хорошо «ложится» на микросервсиную архитектуру или serverless. Потому что у вас уже распределенная система, и серверные resolvers просто направляют запросы к ответственным сервисам или функциям в случае serverless. Для монолитных систем это работает хуже.

## Стоит ли переходить на GraphQL
Все зависит от конкретного кейса и даже от типа мышления. GraphQL не решит все ваши проблемы. Но это очень популярная технология с растущим комьюнити, развитым тулингом и бесконечным списком примеров успешного применения.

## Источники:
- [Как новичку начать пользоваться GraphQL и зачем это нужно](https://blog.skillfactory.ru/kak-novichku-nachat-polzovatsya-graphql-i-zachem-eto-nuzhno/)
- [Что же такое этот GraphQL?](https://habr.com/ru/articles/326986/)
