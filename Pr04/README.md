# Практическая работа №4. Коллекции
## Вариант 2
### Задание
Создайте ассоциативный массив (Map), где ключ – название города, значение – население. Напишите
функцию, которая возвращает город с наибольшим населением.
### fun cityWithMaxPopulation(cities: Map<String, Int>): String? {
    return cities.maxByOrNull { it.value }?.key
}

fun main() {
    val cities = mapOf(
        "Москва" to 12500000,
        "Санкт-Петербург" to 5350000,
        "Новосибирск" to 1600000,
        "Екатеринбург" to 1500000,
        "Казань" to 1250000
    )

    val maxCity = cityWithMaxPopulation(cities)
    if (maxCity != null) {
        println("Город с наибольшим населением: $maxCity")
    } else {
        println("Список городов пуст.")
    }
}
