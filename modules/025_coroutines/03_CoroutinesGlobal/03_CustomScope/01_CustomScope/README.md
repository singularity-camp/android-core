# Custom Scope

`CoroutineScope` определяет жизненный цикл иерархии корутин. В прошлых примерах мы проходили `GlobalScope`, который имеет ряд недостатков по той причине, что корутины внутри него не привязаны ни к чему и останавливаются только тогда, когда завершится вся работа внутри скоупа.

`CoroutineScope`  представляет собой интерфейс, который предоставляет coroutineContext. Вокруг этого интерфейса строятся корутин билдера в виде extension functions. Вспомним прошлый пример, который немного видоизменен:

```
fun main() = runBlocking {
    val coroutineScope = CoroutineScope(SupervisorJob())
    val firstChildJob = coroutineScope.launch {
        throw ArithmeticException()
    }
    coroutineScope.launch { // 1
        try {
            delay(5000)
        } catch (e: CancellationException) {
            println("Cancelled because supervisor got cancelled.")
        }
    }
    firstChildJob.join()
    coroutineScope.cancel() //2
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Здесь мы создали свой собственный scope с помощью **CoroutineScope()**, в который передали уже нам известный `SupervisorJob`. В последней строчке [2] мы заменили **Job.cancel()** на **CoroutineScope.cancel()**, что по сути под капотом делает тоже самое.

Попробуйте убрать coroutineScope в строке 1 перед launch(). Вторая корутина отработает все 5 секунд и завершится без ошибок. Все потому, что она будет скоупиться на главную корутину, которая запущена в runBlocking и уже не имеет отношение к **coroutineScope**.

Как раз для этого и существует `CoroutineScope` - чтобы создавать иерархии корутин, отменять корутины при надобности или возникновения `Exception`.
