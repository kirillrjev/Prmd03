## Практическая работа 15
Создание автоматизированных тестов в Android
 Цель работы

Научиться создавать локальные (unit) и инструментальные (UI) тесты в Android Studio на примере приложения Tip Time.

 Что такое автоматизированные тесты?

Автоматизированные тесты — это код, который проверяет другой код.
Они позволяют убедиться, что приложение работает правильно после каждого изменения.

Почему это важно

Уменьшают время ручного тестирования

Раннее обнаружение ошибок

Позволяют безопасно развивать код

Особенно важны в больших или коммерческих проектах
## Локальные тесты (Local tests)
Запускаются на компьютере, без эмулятора
Проверяют бизнес-логику (функции, классы)
Быстрые и лёгкие
Каталог:

app/src/test/java/...

## Инструментальные тесты (Instrumented/UI tests)
Запускаются на устройстве или эмуляторе
Проверяют UI, взаимодействуют с реальными компонентами
Используют Android API
Чаще медленнее, но имитируют реальный пользовательский опыт
Каталог:

app/src/androidTest/java/...
## оздание локального теста
✔ Подготовка кода

Метод calculateTip() должен быть доступен в тестах:

@VisibleForTesting
internal fun calculateTip(amount: Double, tipPercent: Double = 15.0, roundUp: Boolean): String {
    var tip = tipPercent / 100 * amount
    if (roundUp) tip = kotlin.math.ceil(tip)
    return NumberFormat.getCurrencyInstance().format(tip)
}
## Создание каталога тестов
Project → src → New → Directory → test/java
Создайте пакет:

com.example.tiptime
## оздание тестового класса

TipCalculatorTests.kt:

import org.junit.Assert.assertEquals
import org.junit.Test
import java.text.NumberFormat

class TipCalculatorTests {

    @Test
    fun calculateTip_20PercentNoRoundup() {
        val amount = 10.00
        val tipPercent = 20.00
        val expectedTip = NumberFormat.getCurrencyInstance().format(2)
        
        val actualTip = calculateTip(amount = amount, tipPercent = tipPercent, roundUp = false)

        assertEquals(expectedTip, actualTip)
    }
}
## Создание инструментального UI-теста
✔ Создание каталога
src → New → Directory → androidTest/java
Создайте пакет:

com.example.tiptime
## Создание тестового класса

TipUITests.kt:

import androidx.compose.ui.test.junit4.createComposeRule
import org.junit.Rule
import org.junit.Test
import androidx.compose.ui.test.*
import com.example.tiptime.ui.theme.TipTimeTheme
import java.text.NumberFormat

class TipUITests {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun calculate_20_percent_tip() {
        composeTestRule.setContent {
            TipTimeTheme {
                TipTimeLayout()
            }
        }

        composeTestRule.onNodeWithText("Bill Amount")
            .performTextInput("10")

        composeTestRule.onNodeWithText("Tip Percentage")
            .performTextInput("20")

        val expectedTip = NumberFormat.getCurrencyInstance().format(2)

        composeTestRule.onNodeWithText("Tip Amount: $expectedTip")
            .assertExists("No node with this text was found.")
    }
}
## Запуск UI-теста
Нажмите зелёную стрелку рядом с классом или методом
Приложение запустится на устройстве/эмуляторе
Вы увидите статус теста в панели Run