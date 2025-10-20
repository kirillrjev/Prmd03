# Практическая работа №1: Основы Kotlin - переменные и функции
## Вариант 2: Вычисление периметра прямоугольника
### Задание
Напишите программу, которая конвертирует градусы Цельсия в Фаренгейты.
### Код программы
fun celsiusToFahrenheit(celsius: Double): Double {
    return celsius * 9 / 5 + 32
}

fun main() {
    print("Введите температуру в градусах Цельсия: ")
    val celsius = readLine()?.toDoubleOrNull()

    if (celsius != null) {
        val fahrenheit = celsiusToFahrenheit(celsius)
        println("$celsius °C = $fahrenheit °F")
    } else {
        println("Ошибка: введено некорректное значение.")
    }
}
