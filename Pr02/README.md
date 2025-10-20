# Практическая работа №3: Условные выражения
## Вариант: 2
## Написать программу, которая вычисляет стоимость билета в кино в зависимости от возраста
и времени сеанса.
### Код программы:
### Скриншоты работы программы:
fun calculateTicketPrice(age: Int, time: Int): Double {
    // time — время начала сеанса в 24-часовом формате (например, 18 для 18:00)

    val basePrice = 10.0  // базовая цена билета

    // Скидки по возрасту
    val ageDiscount = when {
        age < 12 -> 0.5   // детям до 12 лет скидка 50%
        age >= 65 -> 0.3  // пенсионерам скидка 30%
        else -> 0.0       // без скидки
    }

    // Скидка за ранний сеанс (до 16:00)
    val timeDiscount = if (time < 16) 0.2 else 0.0  // 20% скидка за дневной сеанс

    // Общая скидка — суммируем обе
    val totalDiscount = ageDiscount + timeDiscount

    // Итоговая цена с учетом скидки
    return basePrice * (1 - totalDiscount)
}

fun main() {
    print("Введите ваш возраст: ")
    val age = readLine()?.toIntOrNull()

    print("Введите время начала сеанса (в часах, 0-23): ")
    val time = readLine()?.toIntOrNull()

    if (age == null || time == null || time !in 0..23) {
        println("Ошибка: введены некорректные данные.")
        return
    }

    val price = calculateTicketPrice(age, time)
    println("Стоимость вашего билета: $price")
}
