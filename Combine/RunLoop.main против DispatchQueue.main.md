# RunLoop.main против DispatchQueue.main

## Планировщик
Объект для управления временем и местом исполнения операторов. Такими планировщиками являются `RunLoop.main` и `DispatchQueue.main`.

RunLoop.main может отложить обновление до момента окончания взаимодействия пользователя с экраном в то время как DispatchQueue.main обновит данный не зависимо от работы пользователя.

## Ичточники:
- [RunLoop.main vs DispatchQueue.main: The differences explained](https://www.avanderlee.com/combine/runloop-main-vs-dispatchqueue-main/)
