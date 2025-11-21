## Практическая работа 28: Разработка приложения с фоновыми задачами

## Код, который нужно реализовать — scheduleReminder()

Файл:
app/src/main/java/com/example/waterme/data/WorkManagerWaterRepository.kt

Вот полностью готовая реализация, которую нужно вставить:

package com.example.waterme.data

import android.content.Context
import androidx.work.*
import com.example.waterme.worker.WaterReminderWorker
import java.util.concurrent.TimeUnit

class WorkManagerWaterRepository(
    private val context: Context
) : WaterRepository {

    private val workManager = WorkManager.getInstance(context)

    override fun scheduleReminder(
        plantName: String,
        duration: Long,
        timeUnit: TimeUnit
    ) {
        // ---- 1. Создаём Data с именем растения ----
        val data = Data.Builder()
            .putString(WaterReminderWorker.nameKey, plantName)
            .build()

        // ---- 2. Создаём OneTimeWorkRequest ----
        val request = OneTimeWorkRequestBuilder<WaterReminderWorker>()
            .setInitialDelay(duration, timeUnit)
            .setInputData(data)
            .build()

        // ---- 3. Ставим в очередь уникальную работу ----
        workManager.enqueueUniqueWork(
            "${plantName}_${duration}_${timeUnit.name}", // уникальное имя
            ExistingWorkPolicy.REPLACE,
            request
        )
    }
}

## 2. Worker (уже готов в стартовом коде, но привожу для удобства)

Файл:
worker/WaterReminderWorker.kt

package com.example.waterme.worker

import android.content.Context
import androidx.work.CoroutineWorker
import androidx.work.WorkerParameters
import com.example.waterme.R

class WaterReminderWorker(
    appContext: Context,
    workerParams: WorkerParameters
) : CoroutineWorker(appContext, workerParams) {

    companion object {
        const val nameKey = "NAME"
    }

    override suspend fun doWork(): Result {

        val plantName = inputData.getString(nameKey)

        makePlantReminderNotification(
            applicationContext.resources.getString(
                R.string.time_to_water,
                plantName
            ),
            applicationContext
        )

        return Result.success()
    }
}


Уведомление уже реализовано в функции makePlantReminderNotification().

## 3. Краткое объяснение, почему решение корректно
✔ 1. Создаём Data для Worker

Работнику нужно имя растения → кладём его в inputData.

✔ 2. Создаём OneTimeWorkRequest

setInitialDelay(duration, timeUnit) откладывает выполнение Worker.

✔ 3. enqueueUniqueWork с уникальным именем

Используем комбинацию:

plantName + duration + timeUnit


потому что:

можно ставить много напоминаний на одно растение (например: 5 сек и 1 день)

WorkManager различает их по уникальному имени

ExistingWorkPolicy.REPLACE перезаписывает только точно такое же по имени задание

## 4. Как протестировать

В приложении WaterMe! при выборе растения есть dropdown со временем → выбираете, например:

5 секунд

1 минута

1 день

После выбора: