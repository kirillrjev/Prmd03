## Практическая работа 20. Корутины. Разработка многопоточного приложения
Предварительные условия

    Знание основ языка Kotlin, включая функции и лямбды
    Умение создавать макеты в Jetpack Compose
    Уметь писать модульные тесты на Kotlin (см. урок «Написание модульных тестов для ViewModel»).
    Как работают потоки и параллелизм
    Базовые знания о корутинах и CoroutineScope

Что вы будете создавать

    Приложение Race Tracker, которое симулирует ход гонки между двумя игроками. Рассматривайте это приложение как возможность поэкспериментировать и узнать больше о различных аспектах корутинов.

Что вы узнаете

    Использование корутинов в жизненном цикле приложений для Android.
    Принципы структурированного параллелизма.
    Как писать юнит-тесты для проверки корутинов.

## Класс RaceParticipant
class RaceParticipant(
    val name: String,
    val maxProgress: Int = 100,
    private val progressIncrement: Int = 1,
    private val progressDelayMillis: Long = 100L,
    private val initialProgress: Int = 0
) {
    var currentProgress by mutableStateOf(initialProgress)
        private set

    suspend fun run() {
        try {
            while (currentProgress < maxProgress) {
                delay(progressDelayMillis)
                currentProgress += progressIncrement
            }
        } catch (e: CancellationException) {
            Log.e("RaceParticipant", "$name: ${e.message}")
            throw e // Обязательно пробросить CancellationException
        }
    }

    fun reset() {
        currentProgress = initialProgress
    }
}

## Запуск гонки в RaceTrackerApp()
@Composable
fun RaceTrackerApp() {
    var raceInProgress by remember { mutableStateOf(false) }
    val playerOne = remember { RaceParticipant("Player 1") }
    val playerTwo = remember { RaceParticipant("Player 2", progressIncrement = 2) }

    if (raceInProgress) {
        LaunchedEffect(playerOne, playerTwo) {
            coroutineScope {
                launch { playerOne.run() }
                launch { playerTwo.run() }
            }
            raceInProgress = false
        }
    }

    RaceTrackerScreen(
        playerOne = playerOne,
        playerTwo = playerTwo,
        raceInProgress = raceInProgress,
        onStartPauseClick = { raceInProgress = !raceInProgress },
        onResetClick = {
            playerOne.reset()
            playerTwo.reset()
            raceInProgress = false
        }
    )
}


Здесь используется coroutineScope { launch { ... } }, чтобы обе корутины выполнялись одновременно, а raceInProgress обновлялся только после их завершения.

## Юнит-тесты для RaceParticipant
class RaceParticipantTest {

    private val raceParticipant = RaceParticipant("TestPlayer", progressDelayMillis = 100L)

    @Test
    fun raceParticipant_RaceStarted_ProgressUpdated() = runTest {
        val expectedProgress = 1
        launch { raceParticipant.run() }
        advanceTimeBy(raceParticipant.progressDelayMillis)
        runCurrent()
        assertEquals(expectedProgress, raceParticipant.currentProgress)
    }

    @Test
    fun raceParticipant_RaceFinished_ProgressUpdated() = runTest {
        launch { raceParticipant.run() }
        advanceTimeBy(raceParticipant.maxProgress * raceParticipant.progressDelayMillis)
        runCurrent()
        assertEquals(raceParticipant.maxProgress, raceParticipant.currentProgress)
    }
}


runTest { ... } запускает тест в корутине.

advanceTimeBy() и runCurrent() ускоряют выполнение delay() без реального ожидания.