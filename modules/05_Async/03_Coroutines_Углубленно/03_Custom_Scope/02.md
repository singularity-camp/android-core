# Scope Functions

Создать скоуп можно также с помощью scope functions:

```
fun main(): Unit = runBlocking {
    coroutineScope {
        launch {
            println("1")
        }.join()

        launch {
            delay(300)
            println("2")
        }

        launch {
            delay(1000)
            println("3")
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

**coroutinScope{}** создает `CoroutinScope` и запускает в нем переданную лямбду.

Теперь добавим исключение в первой джобе:

```
fun main(): Unit = runBlocking {
    coroutineScope {
        launch {
            println("1")
            error("Throw from job 1") //throws exception with the given message
        }.join()

        launch {
            delay(300)
            println("2")
        }

        launch {
            delay(1000)
            println("3")
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Теперь после запуска вторая и третья корутины не запускаются, так как по правилам structured concurrency при падении корутины, отменяется родительская корутина и все ее дочерние корутины.

Такой поведение можно изменить при помощи **supervisorScope{}**:

```
fun main(): Unit = runBlocking {
    supervisorScope {
        println(isActive)
        launch {
            println("1")
            error("Throw from job 1")
        }.join()

        println(isActive)

        launch {
            delay(300)
            println("2")
        }

        launch {
            delay(1000)
            println("3")
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

**supervisorScope{}** создает `CoroutineScope` с `SupervisorJob()` внутри, что позволяет продолжить работу соседних корутин при ошибках одной из дочерних корутин.
