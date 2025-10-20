# Практическая работа №10: Корутины в Kotlin

## Вариант 3: Одна корутина отменяется через 500 мс, а другая завершается успешно

### Задание:
Написать программу, в которой запускаются две корутины:
- Первая выполняется 1 секунду, но отменяется через 500 мс.
- Вторая выполняется до конца (например, 800 мс).
Выводить в консоль состояние выполнения и отмены корутин.

### Код:
```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job1 = launch {
        try {
            println("Job 1 started")
            delay(1000)
            println("Job 1 finished")
        } catch (e: CancellationException) {
            println("Job 1 was cancelled")
        }
    }

    val job2 = launch {
        println("Job 2 started")
        delay(800)
        println("Job 2 finished")
    }

    delay(500) // ждем 500 мс и отменяем первую корутину
    job1.cancelAndJoin()
    job2.join()

    println("Main: all jobs completed")
}
