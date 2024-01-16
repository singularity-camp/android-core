# Callback Hell

Как же тогда обойти `NetworkOnMainThreadException` в Андроид? Все просто, делаем запросы в сеть в новом потоке. Создание нового потока вы уже проходили, теперь поговорим про взаимодействие с главным и worker (доп) потоками. Вспомните прошлый пример:

```
fetchNetworkData(
        onStart = {
            showLoader()
        },
        onFinish = {
            hideLoader()
        }
)
```

Представим, что fetchNetworkData() внутри создает поток и имеет интерфейсы для взаимодействия с главным потоком. Также мы решили добавить еще один запрос, который используем результат предыдущего запроса и хотим отловить ошибки:

```
fetchNetworkData(
        onStart = {
            showLoader()
        },
        onFinish = {
            hideLoader()
        },
        onError {
            handleError()
        },
        onComplete { networkData ->
            val networkDetails = networkData.getDetails()
            fetchNewDetails(networkDetails)
        }
)
```

Какая-то каша получается. Подобным образом работают AsyncTask-и, Retrofit. В RxJava немного по другому выглядело бы:

```
fetchNetworkData()
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.doOnSubscribe(::showLoader)
.doOnTerminate(::hideLoader)
.doOnError(::handleError)
.doOnComplete(::fethcDetails)
```

Тоже не намного лучше, чем было. Чтобы понимать, что происходит в коде, необходимо лезть в детали fetchDetails, который внутри также может содержать вызов дополнительного метода. Такие конструкции, содержащие много callback-ов, называются Callback Hell.
