# Практическая работа №X: Классы и объекты в Kotlin
**Вариант:** [2]
**Задание:** [Создать класс "Студент" с информацией о студенте и методами для работы с оценками.]
## class Student(
    val name: String,
    val age: Int,
    val studentId: String
) {
    private val grades = mutableListOf<Int>()

    // Добавить оценку
    fun addGrade(grade: Int) {
        if (grade in 1..10) {
            grades.add(grade)
        } else {
            println("Оценка должна быть от 1 до 10.")
        }
    }

    // Получить средний балл
    fun getAverageGrade(): Double {
        return if (grades.isNotEmpty()) grades.average() else 0.0
    }

    // Получить все оценки
    fun getGrades(): List<Int> {
        return grades.toList()
    }

    // Вывести информацию о студенте
    fun printInfo() {
        println("Студент: $name, Возраст: $age, ID: $studentId")
        println("Оценки: $grades")
        println("Средний балл: ${"%.2f".format(getAverageGrade())}")
    }
}

fun main() {
    val student = Student("Иван Иванов", 20, "123456")
    student.addGrade(8)
    student.addGrade(9)
    student.addGrade(10)
    student.printInfo()
}
