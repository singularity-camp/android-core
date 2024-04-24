# Async/await

Конструкция **async + await** позволяет запустить несколько корутин одновременно. `Async` является корутин билдером как и `launch{}`  и подчиняется всем правилам, применимым к `launch`. Помните, как мы останавливали корутину, которая запускалась через `launch`? Для этого мы вызывали `job.cancel`. Интерфейс Job позволяет получить информацию о состоянии корутины, дочерних корутин,  а также самим методы ЖЦ корутины.

Вызов `async{}` возвращает `Deferred<T>`. `Deferred` является наследником `Job` и обозначает, что в будущем через метод `Deferred.await()` мы можем получить результат `<T>`:

```
fun main(args: Array<String>): Unit = runBlocking {
    val async = async {
        100 * 1000
    }
    val result: Int = async.await()
    println(result)
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Можно много методов комбинировать и вызвать одновременно:

```
fun main(args: Array<String>): Unit = runBlocking {
    val async1 = async {
        100 * 1000
    }
    val async2 = async {
        100000 / 2.5
    }
    val async3 = async {
        "Hello"
    }
    listOf(async1, async2, async3).awaitAll().forEach(::println)
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

`awaitAll()` является extension методом, который вернет `List<T>` с результатами корутин.
