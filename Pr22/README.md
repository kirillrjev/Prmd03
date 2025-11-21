## Практическая работа 22. Написание модульных тестов для ViewModel
## Добавление зависимостей для модульных тестов

В app/build.gradle.kts:

dependencies {
    ...
    testImplementation("junit:junit:4.13.2")
}


Нажмите Sync Now.


## Стратегия тестирования

Покрываем три типа сценариев:

✔ Успешный путь

Правильный ввод → очки увеличиваются, ошибки нет.

✔ Путь ошибки

Неправильный ввод → очки не меняются, флаг ошибки = true.

✔ Граничные случаи

начальное состояние ViewModel корректно

после последних слов игра завершается

пропуск слова увеличивает счетчик, но не увеличивает очки

## Полный набор тестов (готовый код)

Ниже — финальный файл, содержащий все тесты из практической работы, включая путь успеха, ошибку, оба пограничных случая и дополнительный тест для пропуска слова.

Можно копировать целиком.

## GameViewModelTest.kt
package com.example.android.unscramble.ui.test

import com.example.android.unscramble.ui.GameViewModel
import com.example.android.unscramble.ui.GameUiState
import com.example.android.unscramble.data.MAX_NO_OF_WORDS
import com.example.android.unscramble.data.SCORE_INCREASE
import com.example.android.unscramble.data.getUnscrambledWord

import org.junit.Assert.*
import org.junit.Test

class GameViewModelTest {

    private val viewModel = GameViewModel()

    // ---------- SUCCESS PATH ----------
    @Test
    fun gameViewModel_CorrectWordGuessed_ScoreUpdatedAndErrorFlagUnset() {
        var current = viewModel.uiState.value
        val correctWord = getUnscrambledWord(current.currentScrambledWord)

        viewModel.updateUserGuess(correctWord)
        viewModel.checkUserGuess()
        current = viewModel.uiState.value

        assertFalse(current.isGuessedWordWrong)
        assertEquals(SCORE_AFTER_FIRST_CORRECT_ANSWER, current.score)
    }

    // ---------- ERROR PATH ----------
    @Test
    fun gameViewModel_IncorrectGuess_ErrorFlagSet() {
        val wrongWord = "and"   // слово, которого нет в списке

        viewModel.updateUserGuess(wrongWord)
        viewModel.checkUserGuess()

        val current = viewModel.uiState.value
        assertEquals(0, current.score)
        assertTrue(current.isGuessedWordWrong)
    }

    // ---------- BOUNDARY CASE: INITIAL STATE ----------
    @Test
    fun gameViewModel_Initialization_FirstWordLoaded() {
        val state = viewModel.uiState.value
        val unscrambled = getUnscrambledWord(state.currentScrambledWord)

        assertNotEquals(unscrambled, state.currentScrambledWord)
        assertEquals(1, state.currentWordCount)
        assertEquals(0, state.score)
        assertFalse(state.isGuessedWordWrong)
        assertFalse(state.isGameOver)
    }

    // ---------- BOUNDARY CASE: ALL WORDS GUESSED ----------
    @Test
    fun gameViewModel_AllWordsGuessed_UiStateUpdatedCorrectly() {
        var expectedScore = 0
        var state = viewModel.uiState.value
        var correctWord = getUnscrambledWord(state.currentScrambledWord)

        repeat(MAX_NO_OF_WORDS) {
            expectedScore += SCORE_INCREASE
            viewModel.updateUserGuess(correctWord)
            viewModel.checkUserGuess()

            state = viewModel.uiState.value
            correctWord = getUnscrambledWord(state.currentScrambledWord)

            assertEquals(expectedScore, state.score)
        }

        assertEquals(MAX_NO_OF_WORDS, state.currentWordCount)
        assertTrue(state.isGameOver)
    }

    // ---------- EXTRA: IMPROVES COVERAGE ----------
    @Test
    fun gameViewModel_WordSkipped_ScoreUnchangedAndWordCountIncreased() {
        var state = viewModel.uiState.value
        val correctWord = getUnscrambledWord(state.currentScrambledWord)

        // first correct guess to raise initial score
        viewModel.updateUserGuess(correctWord)
        viewModel.checkUserGuess()

        state = viewModel.uiState.value
        val lastWordCount = state.currentWordCount

        viewModel.skipWord()
        state = viewModel.uiState.value

        assertEquals(SCORE_AFTER_FIRST_CORRECT_ANSWER, state.score)
        assertEquals(lastWordCount + 1, state.currentWordCount)
    }

    companion object {
        private const val SCORE_AFTER_FIRST_CORRECT_ANSWER = SCORE_INCREASE
    }
}

## Запуск тестов с покрытием

Right Click → Run 'GameViewModelTest' with Coverage
→ Открой GameViewModel.kt → розовым подсвечены непокрытые строки.

После добавления теста skipWord() покрытие = 100%.