# CoroutineExceptionHandler

Теперь рассмотрим такой код:

```
fun main() {
    runBlocking {
        val exceptionHandler = CoroutineExceptionHandler { _, exception ->
            println("Caught $exception")
        }
        val job = GlobalScope.launch(exceptionHandler) {
            println("job")
            throw AssertionError("My Custom Assertion Error!")
        }
        val job2 = GlobalScope.launch(exceptionHandler) {
            println("job2")
            throw AssertionError("My Custom Assertion Error!")
        }
        val deferred = async(exceptionHandler) {
            throw ArithmeticException()
        }
        joinAll(job, job2, deferred)
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

`ExceptionHandler` является частью `CoroutineContext` и позволяет отловить исключения внутри заданной корутины и всех ее дочерних корутин. Важно отметить, что он не является обработчиком исключений корутины, так как exception-ы все равно **пойдут вверх по иерархии**, но мы можем таким образом написать свою логику прежде, чем это случится. Обычно там логируются исключения.

В примере выше также родительская корутина не остановилась из-за глобального скоупа. Попробуйте убрать и посмотреть как поведет себя код. Можете проверить на каком потоке запускается скоуп с помощью метода

```
Thread.currentThread()
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Ваша программа упадет только в том случае, если было исключение на main потоке, а используя `GlobalScope.launch()/GlobalScope.async()` без явного указания `Dispatchers.Main`, у вас корутина будет выполняться (и выбрасывать исключения) в другом потоке, поэтому ваша программа продолжает работать, хоть и выведет само исключение на консоль.
