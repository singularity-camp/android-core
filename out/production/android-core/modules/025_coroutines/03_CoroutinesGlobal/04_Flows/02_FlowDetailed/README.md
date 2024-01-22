# Подробнее про Flow

![](https://ucarecdn.com/072ee210-9635-4dc1-8428-e6319a27d501/)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

В структуре Flow существует 3 "стороны":

* **Producer** создается поток данных. Это может быть база данных, репозиторий, сеть и т д.
* **Intermediary** может преобразовывать данные для фильтрации, видоизменения и обработкой данных.
* **Consumer** "собирает" готовые данные.

```
fun main(): Unit = runBlocking {
    getNumbersFlow()
        .filter { it % 2 == 0 }
        .map { "The value is :$it" } // Intermediary
        .collect(::println) // Consumer
}

//Producer
fun getNumbersFlow(): Flow<Int> = flow {
    for (i in 1..10) {
        delay(50)
        emit(i)
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

На примере выше используется фильтрация и маппинг данных во flow. Запустите и проверьте результат.

Теперь добавим логирование, чтобы понимать как проходит видоизменение данных в потоке:

```
fun main(): Unit = runBlocking {
    getNumbersFlow()
        .onEach { println("Before filter with value $it") }
        .filter { it % 2 == 0 }
        .onEach { println("Inside map with value $it") }
        .map { "The value is :$it" } // Intermediary
        .collect(::println) // Consumer
}

//Producer
fun getNumbersFlow(): Flow<Int> = flow {
    for (i in 1..10) {
        delay(50)
        emit(i)
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Результат:

```
Before filter with value 1
Before filter with value 2
Inside map with value 2
The value is :2
Before filter with value 3
Before filter with value 4
Inside map with value 4
The value is :4
Before filter with value 5
Before filter with value 6
Inside map with value 6
The value is :6
Before filter with value 7
Before filter with value 8
Inside map with value 8
The value is :8
Before filter with value 9
Before filter with value 10
Inside map with value 10
The value is :10
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Заметили, что в блок **map** заходят уже отфильтрованные данные? То есть поочередно данные заходят в **filter**, а затем в **map**. Таким образом вся трансформация данных занимает одну итерацию по всему потоку данных. Если бы мы использовали обычный лист, то он весь фильтровался бы перед заходом в map:

```
listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .filter {
        println("Inside filter with value $it")
        it % 2 == 0
    }
    .map {
        println("Inside map with value $it")
        "The value is :$it"
    }
    .forEach(::println)
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Результат:

```
Inside filter with value 1
Inside filter with value 2
Inside filter with value 3
Inside filter with value 4
Inside filter with value 5
Inside filter with value 6
Inside filter with value 7
Inside filter with value 8
Inside filter with value 9
Inside filter with value 10
Inside map with value 2
Inside map with value 4
Inside map with value 6
Inside map with value 8
Inside map with value 10
The value is :2
The value is :4
The value is :6
The value is :8
The value is :10
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")
