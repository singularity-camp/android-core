# Корутины

Как уже мы повторяли ранее, создание и управление потоками дело сложное с точки зрения ресурсов. К тому же инструменты, которые предоставляют интерфейсы для работы с потоками имеют свои недостатки. Котлин предоставляет эффективный инструмент - Coroutines.

Основные плюсы корутин:

* Легкость использования. Везде дается определение корутин как "легковесные потоки". Почему это на самом деле так, мы разберем позже.
* Легкость написания. Корутины позволяют писать асинхронный код в синхронном стиле! Это здорово упрощает чтение и понимание кода.
* Coroutines - фича Kotlin. То есть помимо всех плюсов языка мы получается независимость от платформы, на которой этот код запускается. (Помимо аппаратных ограничений, конечно же)

```
//Вместо 
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
//Мы пишем:
suspend fun fetchNetworkData() {
        try {
            showLoader()
            hideLoader()
            val networkDetails = networkData.getDetails()
            fetchNewDetails(networkDetails)
        } catch(e: Exception) {
            handleError()
        }
    }
```
