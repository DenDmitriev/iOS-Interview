# Concurrency async-await
Используя async-await, мы можем писать асинхронный код без использования обработчиков завершения для возврата значений. Структурированный параллизм улучшает читаемость сложного асинхронного кода в Swift.

- async: Указывает на то, что функция/свойство асинхронны. Кроме того, он позволяет приостановить выполнение кода до тех пор, пока асинхронная функция/свойство не вернет значение или результат.

- await: Указывает, что ваш код может остановить его выполнение, пока он ожидает возврата асинхронной функции/свойства.

Ниже приведены некоторые шаги, которые мы используем asynchronous/await, чтобы сделать функцию/свойство асинхронным
1. Отметьте функцию/свойство, используя ключевое слово `async` в конце имени функции/свойства в определении.
2. Если асинхронная функция/свойство вызывает ошибку, отметьте ее ключевым словом `throw`, которое следует за ключевым словом `async`.
3. Функция/свойство вернули значение успеха. Если вызываемый выдаст ошибку, она будет обработана в блоке do-catch на стороне вызывающего абонента.
4. Мы не можем вызвать непосредственно асинхронную функцию/свойство из синхронного кода, она должна быть обернута в задачу, где она будет выполняться параллельно в фоновом потоке.

> Обработчик завершения (Completion Handler) неструктурирован, в то время как асинхронное ожидание (async-await) следует структурному последовательному шаблону. Ниже фрагменты кода показывают неструктурированный, структурный шаблон замыканий и async/await, соответно.

Функция  async-await ожидания позволяет структурировать параллелизм и, следовательно, улучшает читаемость сложного асинхронного кода в Swift. Вы можете увидеть приведенный ниже код, используя async-await.
```swift
func fetchThumbnail(for url: URL) async throws -> UIImage { // 1. Call the method
        // 2. Fetch images from url and return image
        let ( data, response) = try await URLSession.shared.data(from: url)
        // 3. Throw error if response status code is not succeess
        guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw ARError.badID}
        // 4. Create Image from response data
        let image = UIImage(data: data)
        // 5. Resize image and return
        // 6. Throw error if not valid image
        guard let thumbnail = await image?.byPreparingThumbnail(ofSize: CGSize(width: 300, height: 300)) else { throw ARError.badImage }
        // 7. return valid image
        return thumbnail
}
```
![1*NA9Uw5bR7xh3vIEryzxS6w](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/945427c2-6c86-4773-81f6-4ef4aa94da7c)

Ниже вы можете увидеть, как возвращается метод вызова до того, как изображения будут изобраны с помощью обработчика завершения. Когда результат получен, мы возвращаемся к обратному вызову завершения. Это неструктурированный порядок исполнения, и его трудно выполнить. Если бы мы выполнили другой асинхронный метод в рамках нашего обратного вызова завершения, он бы добавил еще один обратный вызов закрытия. Каждое закрытие добавляет еще один уровень отступа, что затрудняет выполнение приказа об исполнении.

```swift

func fetchThumbnail(for url: URL, completion: @escaping(UIImage?, Error?) -> Void) {
        // 1. Call the method
        let task = URLSession.shared.dataTask(with:url) { data, response, error in // 4. Asynchronous method returns
            
            if let error = error {
                // 5. Any error comes, completion block return with error
                completion(nil, error)
            } else if (response as? HTTPURLResponse)?.statusCode != 200 {
                // 6. Response status code is not succeess, completion block return with error
                completion(nil, ARError.badID)
            } else {
                guard let image = UIImage(data: data!) else {
                    // 7. Not a valid image, completion block return with error
                    completion(nil, ARError.badImage)
                    return
                }
                image.prepareThumbnail(of: CGSize(width: 300, height: 300)) { thumbnail in
                    guard let thumbnail = thumbnail else {
                        // 8. Not a valid image after resize, completion block return with error
                        completion(nil, ARError.badImage)
                        return
                    }
                    // 9. valid image, completion block return with image object
                    completion(thumbnail, nil)
                }
            }
        }
        // 2. Resume data task
        task.resume()
    }
    // 3.Calling method exits
```

## Что такое Task?
Все асинхронные функции выполняются как часть какой-либо задачи. Когда функция выполняет асинхронный вызов, вызываемая функция все еще работает как часть одной и той же задачи (и вызывающий ждет ее возвращения). Аналогично, когда функция возвращается из асинхронного вызова, вызывающий абонент возобновляет выполнение той же задачи. Асинхронный код не может быть вызван непосредственно из синхронного метода. Чтобы вызвать **асинхронный код из синхронного метода**, нам нужно создать задачу и вызвать из нее асинхронную функцию. Он создает **мост между кодом sync/async**. Задача создает среду, в которой вы можете выполнять и управлять асинхронной работой в отдельном потоке. Мы можем запускать, приостанавливать и отменять асинхронный код через тип Task.

```swift
Task {
    let resultImage = try await fetchThumbnail(for: imageURL)
    ... // futher work
    ... // take over (asynchronously)
}
```

При создании задачи также важно указать приоритет в зависимости от того, насколько срочная задача. Хотя вы можете напрямую назначить приоритет задаче при ее создании, если вы этого не сделаете, то Swift будет следовать трем правилам автоматического определения приоритета:

1. Если задача была создана из другой задачи, дочерняя задача унаследует приоритет родительской задачи.
2. Если новая задача была создана непосредственно из основного потока, а не из задачи, ей автоматически присваивается наивысший приоритет userInitiated.
3. Если новая задача не была выполнена другой задачей или основным потоком, Swift попытается запросить приоритет потока.

Самым низким приоритетом является background, где он может выполнить операцию не так срочно с точки зрения пользователя. Наивысший приоритет является high, что является синонимом userInitiated, что указывает на то, что задача важна для пользователя. Приоритетом по умолчанию является medium, где он будет рассматриваться так же, как и другие операции.

### Запуск нескольких асинхронных функций параллельно
Если мы хотим запустить несколько несвязанных асинхронных функций параллельно, мы можем обернуть их в их собственные задачи, которые будут выполняться параллельно. Заказ, в котором они выполняют, не определен.
```swift
Task(priority: .medium) {
	let result1: UIImage = try await fetchThumbnail(for: profileURL)
}

Task(priority: .medium) {
	let result2: UIImage = try await fetchThumbnail(for: profileThumbnailURL)
}

Task(priority: .medium) {
	let result3: UIImage = try await fetchThumbnail(for: productImageURL)
}
```
### Несколько асинхронных операций параллельно и получение результатов одновременно (Параллельная привяза)
Мы можем запускать несколько асинхронных операций параллельно и получать результаты одновременно. Это также называется асинхронным параллельным связыванием.
```swift
Task(priority: .medium) {
    do {

        async let image1 = try await fetchThumbnail(for: productImageURL1)
        
        async let image2 = try await fetchThumbnail(for: productImageURL2)
        
        async let image3 = try await fetchThumbnail(for: productImageURL3)
        
        let imagesArray = try await [image1, image2, image3]
        
    } catch {
        // Handle Error
    }
}
```

### Асинхронные Последовательности
An AsyncSequence resembles the Sequence type — offering a list of values you can step through one at a time — and adds asynchronicity. An may have all, some, or none of its values available when you first use it. Instead, you use await to receive values as they become available.

AsyncSequence напоминает тип Sequence, предлагая список значений, через которые вы можете пройти по одному за раз, и добавляет асинхронность. Она может иметь все, некоторые или ни одно из его значений, доступных при первом использовании. Вместо этого вы используете await для получения значений по мере их появления.

```swift
for await i in Counter(howHigh: 10) {   
  print(i, terminator: " ")
}
```

### Асинхронные свойства
Мы можем сделать свойство асинхронным, для этого нам нужно добавить асинхронное после того, как получим свойство.
```swift
Extension UIImage { 
   // async property 
   var thumbnail: UIImage? {
     get async { 
         let size = CGSize(width: 300, height: 300) 
         return await self.byPreparingThumbnail(ofSize: size)
       }
   }
}
```
>Только свойства только для чтения могут быть асинхронными, нам нужно создать явно установленный для этого свойства. Если мы попытаемся предоставить сеттер для свойства async, компилятор вызовет ошибку.

### Асинхронные свойства с ошибкой
Свойства Async также поддерживают ключевое слово throws. Нам нужно добавить ключевое слово throws после ключевого слова async в определение свойства и использовать try await с методом, который отвечает за выброс ошибки.
```swift
extension UIImage { 
   var posterImage: UIImage {
      get async throws {
     do {
        let ( data, _) = try await URLSession.shared.data(from: imageUrl)
         return UIImage(data: data)!
       } catch {
           throw error
         }
      }
   }
}
```

### Использование отсрочки (defer) внутри асинхронного контекста
Блок отсрочки `defer` выполняется последним перед выходом из контекста и гарантирует выполнение, гарантируя, что очистка ресурса не упускается из виду.

```swift
private func getMovies() {
  defer {
    print("Defer statement outside async")
  }
  Task {
     defer { 
        print("Defer statement inside async")
    }
    let result = try? await     ARMoviesViewModel().callMoviesAPIAsyncAwait(ARMovieResponse.self)
    switch result {
       case .failure(let error): print(error.localizedDescription)
       case .success(let items): movies = items.results
       case .none: print("None")
    }
   print("Inside the Task")
  }
 print("After the Task")
}
// After the Task
// Defer statement outside async
// Inside the Task
// Defer statement inside async
```

### Продолжение асинхронных API
Старый код использует обработчики завершения для уведомления нас, когда какая-то работа была завершена, когда нам придется использовать ее из асинхронной функции - либо для сторонней библиотеки, либо для наших собственных функций, но обновление ее до асинхронной функции потребует много работы. С помощью продолжения мы можем обернуть обработчики завершения в асинхронные API.
Существует несколько видов продолжений:
- withCheckedThrowingContinuation
- withCheckedContinuation
- withUnsafeThrowingContinuation
- withUnsafeContinuation

```swift
func getMoviesPostersAPI(_ movie: ARMovie, completion: @escaping PostersCompletionClosure) {
    let reviewURL = checkURL(ARAPI.moviePosters, movie)
    guard let url = reviewURL else {
        completion(nil, ARNetworkError.invalidUrl)
        return
    }
    ARNetworkManager().executeRequest(url: url, completion: completion)
}

func allPosters(_ movie: ARMovie) async -> (ARMoviePoster?, Error?) {
    await withCheckedContinuation { continuation in
        getMoviesPostersAPI(movie) { posters, error in
            continuation.resume(returning: (posters, error))
        }
    }
}
```
> Продолжение должно быть возобновлено ровно один раз. Не ноль раз, и не дважды или более раз - ровно один раз.

## Преимущества async-awai
- Избегание пирамид вложенности с замыканиями
- Сокращение кода
- Легче читать
- Безопасность с async/await, результат гарантирован, в то время как блоки завершения могут быть вызваны или не вызываться.
