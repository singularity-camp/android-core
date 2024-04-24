# Exception Handling

В асинхронном мире выброс исключения может произойти в неожиданных местах. Работая с `launch{}` билдером, выброс exception прекращает **job**, что также прекращает все дочерние и родительскую job. Прекращение родительской корутины рекурсивно прекращает все ее дочерние и родительские корутины. В итоге все корутины завершат свою работу.

```
fun main() = runBlocking {
    val launchJob = GlobalScope.launch {
        println("1. Exception created via launch coroutine")
        throw IndexOutOfBoundsException()
    }
    launchJob.join()
    println("2. Joined failed job")
    val deferred = GlobalScope.async {
        println("3. Exception created via async coroutine")
        throw ArithmeticException()
    }
    try {
        deferred.await()
        println("4. Unreachable, this statement is never executed")
    } catch (e: Exception) {
        println("5. Caught ${e.javaClass.simpleName}")
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Вывод:

```
1. Exception created via launch coroutine
2. Joined failed job
3. Exception created via async coroutine
5. Caught ArithmeticException
Exception in thread "DefaultDispatcher-worker-2" java.lang
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Как видно из логов,  `IndexOutOfBoundsException` не остановил все корутины. Это потому что эта корутина была запущена в другом (`GlobalScope`) скоупе. Попробуйте убрать `GlobalScope` и посмотреть вывод.

Далее при `await()` у нас произошло исключение, так как корутина выбросила его. При этом в самом async билдера ничего не останавливается, так билдер нужно запустить с помощью `await()`.  Строчка   `println("4. Unreachable, this statement is never executed")` так и. не вызвалась из-за прыжка в блок catch. Об этом даже IDE предупреждает перед запуском кода.

Нужно отметить, что при **launch{}** обработать ошибки нужно в рамках самой корутины, а при **async{}** - вне корутины. Обработка исключений является ответственностью разработчика.
