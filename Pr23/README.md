## Практическая работа 23: Разработка приложения Material Design - Список супергероев
## Часть 1 — Структура проекта и подготовка
Создайте новый проект

Empty Activity, API 35

Название: Superheroes

Добавьте ресурсы

Вставьте изображения супергероев в app/src/main/res/drawable

Добавьте шрифты в app/src/main/res/font

Добавьте строки в strings.xml (как в задании)

## Модель данных

Создайте пакет:

com.example.superheroes.model

## Hero.kt
package com.example.superheroes.model

import androidx.annotation.DrawableRes
import androidx.annotation.StringRes

data class Hero(
    @StringRes val nameRes: Int,
    @StringRes val descriptionRes: Int,
    @DrawableRes val imageRes: Int
)

## HeroesRepository.kt
package com.example.superheroes.model

import com.example.superheroes.R

object HeroesRepository {
    val heroes = listOf(
        Hero(R.string.hero1, R.string.description1, R.drawable.android_superhero1),
        Hero(R.string.hero2, R.string.description2, R.drawable.android_superhero2),
        Hero(R.string.hero3, R.string.description3, R.drawable.android_superhero3),
        Hero(R.string.hero4, R.string.description4, R.drawable.android_superhero4),
        Hero(R.string.hero5, R.string.description5, R.drawable.android_superhero5),
        Hero(R.string.hero6, R.string.description6, R.drawable.android_superhero6)
    )
}

## Настройка Material Theme

Ты уже получил весь код цветов, типографики и форм — просто разложи файлы:

ui/theme/Color.kt
ui/theme/Type.kt
ui/theme/Shape.kt
ui/theme/Theme.kt


И помести туда код из задания.

Не забудь вызвать:

setUpEdgeToEdge(view, darkTheme)


внутри SideEffect.

## UI: создание экрана и элемента списка

Создайте файл:

com.example.superheroes/HeroesScreen.kt

## Элемент списка одного супергероя (HeroItem)

Это точное следование спецификации из задания:
высота 72dp, карточка 2dp, скругление 16dp, картинка 72dp, текст 16dp справа.

package com.example.superheroes

import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import com.example.superheroes.model.Hero

@Composable
fun HeroItem(
    hero: Hero,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .padding(8.dp)
            .fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp),
        shape = RoundedCornerShape(16.dp)
    ) {
        Row(
            modifier = Modifier
                .height(72.dp)
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            HeroImage(hero.imageRes)

            Spacer(modifier = Modifier.width(16.dp))

            Column {
                Text(
                    text = stringResource(hero.nameRes),
                    style = MaterialTheme.typography.displaySmall
                )
                Text(
                    text = stringResource(hero.descriptionRes),
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
    }
}

@Composable
fun HeroImage(imageRes: Int) {
    Image(
        painter = painterResource(imageRes),
        contentDescription = null,
        modifier = Modifier
            .size(72.dp)
            .clip(RoundedCornerShape(8.dp))
    )
}

## Создание LazyColumn для списка героев
@Composable
fun HeroesList(
    heroes: List<Hero>,
    modifier: Modifier = Modifier
) {
    androidx.compose.foundation.lazy.LazyColumn(
        modifier = modifier.padding(horizontal = 16.dp, vertical = 8.dp)
    ) {
        items(heroes.size) { index ->
            HeroItem(hero = heroes[index])
        }
    }
}

## Верхняя панель приложений

В Material 3 используем CenterAlignedTopAppBar.

📄 Добавьте в HeroesScreen.kt
import androidx.compose.material3.CenterAlignedTopAppBar
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBarDefaults

@Composable
fun SuperheroesTopAppBar() {
    CenterAlignedTopAppBar(
        title = {
            Text(
                text = stringResource(id = R.string.app_name),
                style = MaterialTheme.typography.displayLarge
            )
        },
        colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    )
}
## Собираем всё в MainActivity

Открой MainActivity.kt.

package com.example.superheroes

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material3.Scaffold
import com.example.superheroes.model.HeroesRepository
import com.example.superheroes.ui.theme.SuperheroesTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            SuperheroesTheme {
                Scaffold(
                    topBar = { SuperheroesTopAppBar() }
                ) { padding ->
                    HeroesList(
                        heroes = HeroesRepository.heroes,
                        modifier = Modifier.padding(padding)
                    )
                }
            }
        }
    }
}