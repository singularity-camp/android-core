# SupervisorJob

В прошлых примерах при выбросе исключения из корутины происходит остановка всех корутин. Однако, такое поведение отличается для `GlobalScope`. На самом деле использование `GlobalScope` является **антипаттерном**. Оно нарушает всю идею Structured Concurrency. Рекомендую к прочтению [статью](https://elizarov.medium.com/the-reason-to-avoid-globalscope-835337445abc).

Итак, при выбросе exception **launch()** останавливает себя и передает исключение для обработки родительской корутине. В итоге исключение в одной из дочерних корутин останавливает все дочерние корутины. Если такое поведение не желательно, используйте `SupervisorJob`:

```
fun main() = runBlocking {
    val supervisorJob = SupervisorJob()
    val coroutineScope = CoroutineScope(coroutineContext + supervisorJob)
    val firstChild: Job = coroutineScope.launch {
        println("First child throwing an exception")
        throw ArithmeticException()
    }
    val secondChild: Job = coroutineScope.launch {
        println("First child is cancelled: ${firstChild.isCancelled}")
        try {
            delay(5000)
        } catch (e: CancellationException) {
            println("Second child cancelled because supervisor got cancelled.")
        }
    }
    firstChild.join()
    println("Second child is active: ${secondChild.isActive}")
    supervisorJob.cancel()
    secondChild.join()
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Здесь мы создали новую `Job`, которая наследует контекст из вызывающей корутины, но переопределяет `Job` своим `SupervisorJob`. Можете запустить и заметить, что падение **firstChild** не останавливает **secondChild**. Однако **secondChild** остановится при вызове **сancel()** у  `SupervisorJob`.

Также в примере выше мы создали свой скоуп (отличный от GlobalScope), о котором мы поговорим в следующем разделе.
