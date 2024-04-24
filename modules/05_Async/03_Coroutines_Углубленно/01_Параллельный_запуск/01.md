# Deferred

До этого времени мы запускали задачи в корутинах последовательно. То есть,  в рамках одной корутины запускается один метод, после его завершения запускается следующий, и так далее:

```
fun main(args: Array<String>): Unit = runBlocking {
    println(getRandomNumber())
    println(getRandomNumber())
    println(getRandomNumber())
    println(getRandomNumber())
    println(getRandomNumber())
}

private suspend fun getRandomNumber(): Int {
    delay(Random().nextInt(5000).toLong())
    return Random().nextInt()
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

На примере выше есть метод с (условно) долгим выполнением (до 5 секунд). Чтобы сгенерировать 5 цифр, нам нужно вызвать метод соответственно 5 раз. В реальных задачах это может быть 5 запросов в сеть. Ожидание результата 5 запросов сеть в может быть долгим, не говоря о том, что результаты нужно обработать (десериализовать, замаппать в нужные модель и преобразовать в нужный вид).

В рамках одной корутины выполнение suspend функций в последовательном виде является правильным. Ведь **suspend** для этого и придуман, чтобы приостанавливать поток, вместе его блокирования, что приводит к приостановлению всей корутины. Давайте для каждого вызова getRandomNumber создадим отдельную корутину:

```
fun main(args: Array<String>): Unit = runBlocking {
    launch { println(getRandomNumber()) }
    launch { println(getRandomNumber()) }
    launch { println(getRandomNumber()) }
    launch { println(getRandomNumber()) }
    launch { println(getRandomNumber()) }
}

private suspend fun getRandomNumber(): Int {
    println("getRandomNumber called")
    delay(Random().nextInt(5000).toLong())
    return Random().nextInt()
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Сейчас если запустить, то 5 корутин не работают последовательно: вторая корутина не ждет завершения первой для запуска. Вызов **getRandomNumber()** идет последовательно (судя по логам), но первый вызов **launch** не приостанавливает весь метод **main()**, поэтому мы не получает результаты getRandomNumber последовательно как в прошлом примере.

Чтобы вызовы корутин были последовательны, нам нужно дождаться результата выполнения корутин. Для этого добавьте вызов метода join() для launch билдера:

```
launch { println(getRandomNumber()) }.join()
launch { println(getRandomNumber()) }.join()
launch { println(getRandomNumber()) }.join()
launch { println(getRandomNumber()) }.join()
launch { println(getRandomNumber()) }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Как вы помните, **launch()** возвращает `Job` этой корутины. Метод **Job.join()** позволяет приостановить выполнение родительской корутины до завершения выполнения данной job.

Теперь, когда все корутины не зависят друг от друга, мы можем распараллелить их запуск. Для немного перепишем код:

```
fun main(args: Array<String>): Unit = runBlocking {
    val async1 = async { getRandomNumber(1) }
    val async2 = async { getRandomNumber(2) }
    val async3 = async { getRandomNumber(3) }
    val async4 = async { getRandomNumber(4) }
    val async5 = async { getRandomNumber(5) }

    async1.await()
    async2.await()
    async3.await()
    async4.await()
    async5.await()
}

private suspend fun getRandomNumber(index: Int) {
    println("coroutine $index started")
    delay(Random().nextInt(5000).toLong())
    println("got ${Random().nextInt()} from coroutine $index")
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Вместо **launch** теперь мы используем **async**, а вместо **join** - **await()**. Далее мы разберем для чего это нужно а пока запустите и убедитесь, что корутины работают паралельно.
