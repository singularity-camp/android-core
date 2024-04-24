# Structured Concurrency

Мы уже отменяли `Job` (корутину). Отменять и очищать код полезно и нужно, чтобы не занимать ресурсы и не плодить утечки памяти. Архитектура корутин составлена таким образом, что, по умолчанию, отмена любой джобы останавливает все дочерние джобы. Однако, отмена корутины не всегда гарантирует остановку корутины. Рассмотрим такой код:

```
fun main(args: Array<String>): Unit = runBlocking {
    val job = launch(Dispatchers.IO) {
        while (true) {
            println("We are inside coroutine")
        }
    }

    delay(50L)
    job.cancel()
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Хотя мы и вызываем сразу **cancel()** для `Job`, она по факту не остановит `Job`, а выставит некий флаг внутри корутины (isCancelled = true). И далее делается проверка **isActive(),** чтобы понимать нужно ли завершить работу корутины. Такая проверка происходит неявным образом перед созданием новой дочерней корутины, либо перед и после вызова другой suspend функции.

В нашем примере нет новый корутин и вызова внешних **suspend** функций, поэтому нам нужно самим проверять состояние джобы:

```
fun main(args: Array<String>): Unit = runBlocking {
    val job = launch(Dispatchers.IO) {
        while (true) {
            if (!isActive) {
                println("Ok, now I shut up")
                return@launch
            }
            println("We are inside coroutine")
        }
    }

    delay(50L)
    job.cancel()
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

При отмене корутина вызывает `CancellationException`. Это важный момент, так как именно этот тип исключение и позволяет останавливать родительские корутины:

```
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        repeat(5) { i ->
            try {
                // print a message twice a second
                println("job: I'm sleeping $i ...")
                delay(500)
            } catch (e: Exception) {
                // log the exception
                println(e)
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

В этом примере **job** останавливается при **job.cancelAndJoin**, однако **job** не остановится, так как мы отловили этот exception. `CancellationException` не пробросился вверх и родительская корутина не остановится. Для корректной остановки, нужно пробросить обработку этого exception:

```
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        repeat(5) { i ->
            try {
                // print a message twice a second
                println("job: I'm sleeping $i ...")
                delay(500)
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                println(e)
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")
