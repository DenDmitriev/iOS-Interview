# RunLoop.main против DispatchQueue.main

RunLoop.main может отложить обновление до момента окончания взаимодействия пользователя с экраном в то время как DispatchQueue.main обновит данный не зависимо от работы пользователя.

## Ичточники:
- [RunLoop.main vs DispatchQueue.main: The differences explained](https://www.avanderlee.com/combine/runloop-main-vs-dispatchqueue-main/)
