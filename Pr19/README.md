## Практическая работа 19. Создание арт-пространства
Необходимые условия

    Умение создавать и запускать проект в Android Studio. Опыт работы с синтаксисом Kotlin, включающим булевы выражения и выражения when.
    Умение применять основные концепции Jetpack Compose, такие как использование состояния с помощью объекта MutableState.
    Опыт работы с композитными функциями, включая композитные функции Text, Image и Button.

Что вы узнаете

    Как создавать прототипы низкой точности и воплощать их в коде.
    Как создавать простые макеты с помощью композитных элементов Row и Column и упорядочивать их с помощью параметров horizontalAlignment и verticalArrangement.
    Как настраивать элементы Compose с помощью объекта Modifier.
    Как определять состояния и изменять их при срабатывании триггеров, например нажатии кнопок.

Что вы создадите

    Приложение для Android, которое может отображать произведения искусства или семейные фотографии.

## Пример кода
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.Image
import androidx.compose.foundation.border
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.artspace.ui.theme.ArtSpaceTheme

// Модель данных для произведения искусства
data class Artwork(
    val id: Int,
    val title: String,
    val artist: String,
    val year: String,
    val imageRes: Int
)

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ArtSpaceTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    ArtSpaceApp()
                }
            }
        }
    }
}

@Composable
fun ArtSpaceApp() {
    // Список произведений искусства
    val artworks = listOf(
        Artwork(1, "Звездная ночь", "Ван Гог", "1889", R.drawable.starry_night),
        Artwork(2, "Мона Лиза", "Да Винчи", "1503", R.drawable.mona_lisa),
        Artwork(3, "Крик", "Мунк", "1893", R.drawable.the_scream)
    )

    // Текущее отображаемое произведение
    var currentIndex by remember { mutableStateOf(0) }
    val currentArtwork = artworks[currentIndex]

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.SpaceAround
    ) {
        // Изображение произведения
        Image(
            painter = painterResource(id = currentArtwork.imageRes),
            contentDescription = currentArtwork.title,
            modifier = Modifier
                .size(250.dp)
                .border(2.dp, Color.Gray, RoundedCornerShape(8.dp))
        )

        // Информация о произведении
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Text(text = currentArtwork.title, fontSize = 24.sp)
            Text(text = currentArtwork.artist, fontSize = 18.sp, color = Color.DarkGray)
            Text(text = currentArtwork.year, fontSize = 16.sp, color = Color.Gray)
        }

        // Кнопки управления
        Row(
            horizontalArrangement = Arrangement.SpaceEvenly,
            verticalAlignment = Alignment.CenterVertically,
            modifier = Modifier.fillMaxWidth()
        ) {
            Button(onClick = {
                // Предыдущее произведение
                currentIndex = if (currentIndex == 0) artworks.size - 1 else currentIndex - 1
            }) {
                Text("Предыдущее")
            }

            Button(onClick = {
                // Следующее произведение
                currentIndex = if (currentIndex == artworks.size - 1) 0 else currentIndex + 1
            }) {
                Text("Следующее")
            }
        }
    }
}

## Анимация при смене картин

Мы можем сделать плавное появление и исчезновение картин с помощью AnimatedVisibility или Crossfade. Пример с Crossfade:

import androidx.compose.animation.Crossfade

Crossfade(targetState = currentArtwork) { artwork ->
    Image(
        painter = painterResource(id = artwork.imageRes),
        contentDescription = artwork.title,
        modifier = Modifier
            .size(250.dp)
            .border(2.dp, Color.Gray, RoundedCornerShape(8.dp))
    )
}


Теперь картинки будут плавно переключаться при нажатии на кнопки.

## Всплывающая подсказка при долгом нажатии

Используем Tooltip через pointerInput (или проще с Toast для Android):

import android.widget.Toast
import androidx.compose.ui.platform.LocalContext
import androidx.compose.foundation.gestures.detectTapGestures
import androidx.compose.ui.input.pointer.pointerInput

val context = LocalContext.current

Image(
    painter = painterResource(id = currentArtwork.imageRes),
    contentDescription = currentArtwork.title,
    modifier = Modifier
        .size(250.dp)
        .border(2.dp, Color.Gray, RoundedCornerShape(8.dp))
        .pointerInput(Unit) {
            detectTapGestures(
                onLongPress = {
                    Toast.makeText(
                        context,
                        "Автор: ${currentArtwork.artist}\nГод: ${currentArtwork.year}",
                        Toast.LENGTH_SHORT
                    ).show()
                }
            )
        }
)


Теперь при долгом нажатии появляется подсказка с информацией о картине.

## Адаптивность для разных размеров экранов

Для планшетов используем BoxWithConstraints или проверяем maxWidth:

BoxWithConstraints(modifier = Modifier.fillMaxSize()) {
    if (maxWidth < 600.dp) {
        // Вертикальный макет для телефонов
        Column { /* Контент */ }
    } else {
        // Горизонтальный макет для планшетов
        Row { /* Контент */ }
    }
}