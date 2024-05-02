# Android Specific Scopes

Помимо **CoroutineScope()**, который использует `Dispatchers.Default` по умолчанию для создания корутин, существует **MainScope()**:

```
fun main(): Unit = runBlocking {
    val scope = MainScope()
    scope.launch {
        println("Hello From MainScope")
    }
    scope.cancel()

    println("end")
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Если запустить, можно заметить, что корутина не отработает с

```
IllegalStateException: Module with the Main dispatcher is missing. Add dependency providing the Main dispatcher
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Все потому, что `Dispatcher.Main` должен переопределяться самой платформой, на которой запускаются корутины. Это может быть сервер, браузер или мобильный телефон. Мы рассмотрим `MainScope` подробнее при работе с приложением.

Для работы в рамках ЖЦ UI компонентов существует **LifecycleScope**. Для того, чтобы его использовать, необходимо подключить зависимость в Android проект - `androidx.lifecycle:lifecycle-runtime-ktx:2.4.0` (или выше). Скоуп привязан к ЖЦ компонента и отменяет свои дочерние корутины при **DESTROYED** state:

```
class MyFragment: Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewLifecycleOwner.lifecycleScope.launch {
            val params = TextViewCompat.getTextMetricsParams(textView)
            val precomputedText = withContext(Dispatchers.Default) {
                PrecomputedTextCompat.create(longTextContent, params)
            }
            TextViewCompat.setPrecomputedText(textView, precomputedText)
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Однако запуск корутин внутри фрагмента/активити это редкий случай, поэтому для `ViewModel` существует аналогичный `ViewModelScope`:

```
class MyViewModel: ViewModel() {
    init {
        viewModelScope.launch {
            // Coroutine that will be canceled when the ViewModel is cleared.
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

Для добавления **viewmodelScope** необходимо добавить

```
androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "Click and drag to move")

(и выше).
