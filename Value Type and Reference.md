# Value Type и Reference Type или чем стек отличается от кучи?

В Swift есть две категории типов: 
 - Value Type (Тип значения) — каждый экземпляр хранит уникальную копию своих данных. То есть происходит создание нового экземпляра (копии) при присвоении значения переменной/константе или при передаче экземпляра в функцию. 
 - Reference Type (Ссылочный тип) — каждый экземпляр использует одну копию данных. То есть сохраняется/возвращается ссылка на тот же экземпляр при присвоении значения переменной/константе или при передаче экземпляра в функцию.
 
Давайте разделим типы в Swift на две категории:
| Тип значения | Ссылочный тип |
| - | - |
| Int | Функции |
| Double | Замыкания 
| String | Классы |
| Array |
| Dictionary |
| Set |
| Struct |
| Enum |
| Tuple |

 - Экземпляры ссылочного типа (Reference Type), например, функции или классы хранятся в управляемой динамической памяти — куче (heap).
 - Экземпляры типа значения (Value Type), такие, как структура или массив, находятся в области памяти, называемой стеком (stack).

Существует одно важное замечание — если экземпляр типа значения является частью экземпляра ссылочного типа, то значение сохраняется в куче вместе с экземпляром ссылочного типа. Например, структура сама по себе хранится в стеке, но если эта структура расположена в классе, то так как класс хранится в куче, то и структура будет сохранена в куче. 

> Стек используется для распределения статической памяти, а куча — для распределения динамической памяти, которая хранятся в оперативной памяти компьютера.

[Источник](https://ios-interview.ru/value-and-reference-type/)