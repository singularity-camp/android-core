# Смена контекста

Часто нам нужно комбинировать работу в разных контекстах в рамках одной корутины. Изменять контекст можно с помощью метода **withContext**:

```
fun main(args: Array<String>) {
    GlobalScope.launch(Dispatchers.Unconfined) {
        println("I am in ${Thread.currentThread()}")
        withContext(Dispatchers.IO) {
            println("Now I am in ${Thread.currentThread()}")
        }
    }

    Thread.sleep(1000)
}
```

Запустите и проверьте на каких потоках запустился код.
