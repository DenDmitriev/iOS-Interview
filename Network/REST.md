# Архитектура REST
**REST** *(Representational state transfer - Передача представительского состояния)* – это стиль архитектуры программного обеспечения для распределенных систем, таких как World Wide Web, который, как правило, используется для построения веб-служб. Термин REST был введен в 2000 году Роем Филдингом, одним из авторов HTTP-протокола. 

> Системы, поддерживающие REST, называются RESTful-системами.

![REST-API-1](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/63cd6b9f-db4c-4bd7-aaf6-6b49d1b0e7ab)


## REST API
Это архитектурный подход, который устанавливает ограничения для API: как они должны быть устроены и какие функции поддерживать. Это позволяет стандартизировать работу программных интерфейсов, сделать их более удобными и производительными.

В общем случае REST является очень простым интерфейсом управления информацией без использования каких-то дополнительных внутренних прослоек. Каждая единица информации однозначно определяется глобальным идентификатором, таким как URL. Каждая URL в свою очередь имеет строго заданный формат.

## Свойства
В отличие от, например, SOAP API, REST API — не протокол, а простой список рекомендаций, которым можно следовать или не следовать. Поэтому у него нет собственных методов. С другой стороны, его автор Рой Филдинг создал ещё и протокол HTTP, так что они очень хорошо сочетаются, и REST обычно используют в связке с HTTP. Хотя новичкам нужно помнить: REST — это не только HTTP, а HTTP — не только REST.

Отсутствие дополнительных внутренних прослоек означает передачу данных в том же виде, что и сами данные. Т.е. мы не заворачиваем данные в XML, как это делает SOAP и XML-RPC, не используем AMF, как это делает Flash и т.д. Просто отдаем сами данные.

Каждая единица информации однозначно определяется URL – это значит, что URL по сути является первичным ключом для единицы данных. Т.е. например третья книга с книжной полки будет иметь вид /book/3, а 35 страница в этой книге — /book/3/page/35. Отсюда и получается строго заданный формат. Причем совершенно не имеет значения, в каком формате находятся данные по адресу /book/3/page/35 – это может быть и HTML, и отсканированная копия в виде jpeg-файла, и документ Microsoft Word.

Как происходит управление информацией сервиса – это целиком и полностью основывается на протоколе передачи данных. Наиболее распространенный протокол конечно же HTTP. Так вот, для HTTP действие над данными задается с помощью методов: GET (получить), PUT (добавить, заменить), POST (добавить, изменить, удалить), DELETE (удалить). Таким образом, действия CRUD (Create-Read-Updtae-Delete) могут выполняться как со всеми 4-мя методами, так и только с помощью GET и POST.

## Как работает REST API: 6 принципов архитектуры
Всего в REST есть шесть требований к проектированию API. Пять из них обязательные, одно — опциональное:
1. Клиент-серверная модель (client-server model).
2. Отсутствие состояния (statelessness).
3. Кэширование (cacheability).
4. Единообразие интерфейса (uniform interface).
5. Многоуровневая система (layered system).
6. Код по требованию (code on demand) — необязательно.

## Для чего используют REST API
Архитектурный стиль REST — самый распространённый подход к проектированию API. Вот в каких случаях его применяют:
- пропускная способность соединения с сервером ограничена;
- нужно соединить мобильные приложения с серверными;
- проект разбит на микросервисы;
- сервис предоставляет свои возможности другим разработчикам;
- используется AJAX;
- известно, что систему нужно будет масштабировать.

## Пример
Вот как это будет выглядеть на примере:
- GET /book/ — получить список всех книг
- GET /book/3/ — получить книгу номер 3
- PUT /book/ — добавить книгу (данные в теле запроса)
- POST /book/3 – изменить книгу (данные в теле запроса)
- DELETE /book/3 – удалить книгу

> Существуют так называемые REST-Patterns, которые различаются связыванием HTTP-методов с тем, что они делают. В частности, разные паттерны по-разному рассматривают POST и PUT. Однако, PUT предназначен для создания, реплейса или апдейта, для POST это не определено (The POST operation is very generic and no specific meaning can be attached to it). Поэтому мой пример будет правильным и в таком виде, и в виде если поменять местами POST и PUT

Вообще, POST может использоваться одновременно для всех действий изменения:
- POST /book/ – добавить книгу (данные в теле запроса)
- POST /book/3 – изменить книгу (данные в теле запроса)
- POST /book/3 – удалить книгу (тело запроса пустое)

## Какие можно сделать из этого выводы:
Как видно, в архитектура REST очень проста в плане использования. По виду пришедшего запроса сразу можно определить, что он делает, не разбираясь в форматах (в отличие от SOAP, XML-RPC). Данные передаются без применения дополнительных слоев, поэтому REST считается менее ресурсоемким, поскольку не надо парсить запрос чтоб понять что он должен сделать и не надо переводить данные из одного формата в другой.

## Практическое применение.
Самое главное достоинство сервисов в том, что с ними работать может какая угодно система, будь то сайт, flash, программа и др. так как методы парсинга XML и выполнения запросов HTTP присутствуют почти везде.

Архитектура REST позволяет серьезно упростить эту задачу. Конечно в реальности, того что описано не достаточно, ведь нельзя кому угодно давать возможность изменять информацию, то есть нужна еще авторизация и аутентификация. Но это достаточно просто разрешается при помощи различного типа сессий или просто HTTP Authentication.

## Источники
- [REST API: что это такое и как работает](https://skillbox.ru/media/code/rest-api-chto-eto-takoe-i-kak-rabotaet/)
