# Coroutine Flows

Возможно, вы знакомы с RxJava, либо в целом с [реактивным программированием](https://ru.wikipedia.org/wiki/%D0%A0%D0%B5%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5). Суть в том, чтобы реагировать на изменение потока данных и обрабатывать результат. В Kotlin для этого есть `Flow`.

`Flow` позволяет создавать поток данных, под потоком данных здесь подразумевается набор данных, которые могут исходить (emitted, produced) последовательно. В следующем примере поток данных создается и обрабатывается единовременно:

```
listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).forEach(::println)
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Сначала создается весь list, а затем каждое значение выводится на консоль.  `Flow` позволяет отдавать и обрабатывать такой поток данных последовательно по одному элементу:

```
fun main(): Unit = runBlocking {
    getNumbersFlow().collect(::println)
}

fun getNumbersFlow(): Flow<Int> = flow { // flow builder
    for (i in 1..10) {
        delay(50)
        emit(i)
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Метод создает `Flow``Int`-ов с помощью **flow{}** билдера. **Flow Builder** примает suspend функцию как аргумент, что позволяет вызывать suspend методы внутри блока как например **delay().**  Сигнатура suspend обязывает подписаться на flow в рамках корутины (либо другого suspend метода). Meтод **emit()** "испускает" данные во `flow`. Далее на этот флоу нужно "подписаться" с помощью метода **collect()**, который в аргумента принимает элемент `flow`. Запустите и проверьте как данных отображаются в консоли.

Можно также перевести существующий список во flow и обработать элементы:

```
flowOf(1,2,3,4,5,6,7,8,9,10).collect(::println)
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Однако, в примере выше нет suspend метода. Как и в примере ниже с использованием extention метода:

```
listOf(1,2,3,4,5,6,7,8,9,10).asFlow().collect(::println)
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")
