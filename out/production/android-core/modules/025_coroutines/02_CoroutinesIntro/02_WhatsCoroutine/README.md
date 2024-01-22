# Что такое корутины

"Легковесные потоки". На этом у нас все, всем спасибо.

Нет, не так просто...

Хотя работа корутин напрямую связана с потоками, они не про многопоточность, а про асинхронность. Потоками являются инструменты, которыми эффективно пользуются корутины. К примеру, создадим **миллион** потоков:

```
@OptIn(ExperimentalTime::class)
fun main(args: Array<String>) {
    val time: Duration = measureTime {
        repeat(1_000_000) {
            Thread {
                println("Hi from thread")
            }.start()
        }
    }

    println("Finished in ${time.inWholeSeconds} seconds")
}
```

Вывод получился таким:

![](https://ucarecdn.com/78a1fe52-9746-41be-95d2-bda093e28e6c/)

Теперь создадим **миллион** корутин. Как именно они создаются мы разберем позже.

```
fun main(args: Array<String>) = runBlocking {
    repeat(1_000_000) {
        launch { // запуск корутины
            println("Hi from thread")
        }
    }
}
```

Угадаете за сколько отработал код? Чтобы добавить логирование, нужно немного повозиться:

```
@OptIn(ExperimentalTime::class)
fun main(args: Array<String>) = runBlocking {
    val time: Duration = measureTime {
        launch { // корутина для измерения времени
            repeat(1_000_000) {
                launch {
                    println("Hi from thread")
                }
            }
        }.join()
    }
    println("Finished in ${time.inWholeMilliseconds} ms")
}
```

![](https://ucarecdn.com/f99f8cc4-9ae2-44f7-bdb8-004005946f8d/)

**1.2 секунды!!!**

Разница ощутимая, однако в коде выше запускалась лишняя корутина для высчитывать времени, так что смею округлить до 1.1 секунд.
