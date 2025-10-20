## Что нужно сделать:

Создайте новый проект в Android Studio:

Шаблон: Empty Activity

Название: Superheroes

Min SDK: 24

Добавьте ресурсы:

Изображения супергероев и иконку приложения – скопируйте в res/drawable.

Шрифты Cabin Regular и Bold – скопируйте в res/font.

Настройте иконку приложения, как описано в codelab “Изменение значка приложения”.
## Создание модели данных 
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
## strings.xml
## Добавьте строки в res/values/strings.xml:
<string name="app_name">Superheroes</string>
<string name="hero1">Nick the Night and Day</string>
<string name="description1">The Jetpack Hero</string>
<string name="hero2">Reality Protector</string>
<string name="description2">Understands the absolute truth</string>
<string name="hero3">Andre the Giant</string>
<string name="description3">Mimics the light and night to blend in</string>
<string name="hero4">Benjamin the Brave</string>
<string name="description4">Harnesses the power of canary to develop bravely</string>
<string name="hero5">Magnificent Maru</string>
<string name="description5">Effortlessly glides in to save the day</string>
<string name="hero6">Dynamic Yasmine</string>
<string name="description6">Ability to shift to any form and energize</string>

## Создание экрана со списком супергероев
## Создайте файл HeroesScreen.kt
package com.example.superheroes

import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import com.example.superheroes.model.Hero
import com.example.superheroes.model.HeroesRepository

@Composable
fun HeroItem(hero: Hero, modifier: Modifier = Modifier) {
    Card(
        modifier = modifier
            .padding(16.dp)
            .fillMaxWidth()
            .height(72.dp),
        shape = MaterialTheme.shapes.medium
    ) {
        Row(modifier = Modifier.padding(16.dp)) {
            Image(
                painter = painterResource(hero.imageRes),
                contentDescription = null,
                modifier = Modifier
                    .size(72.dp)
                    .clip(MaterialTheme.shapes.small),
                contentScale = ContentScale.Crop
            )
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
fun HeroesList(modifier: Modifier = Modifier) {
    LazyColumn(modifier = modifier) {
        items(HeroesRepository.heroes) { hero ->
            HeroItem(hero = hero)
        }
    }
}
## Верхняя панель приложения (TopAppBar)
## В MainActivity.kt
package com.example.superheroes

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import com.example.superheroes.ui.theme.SuperheroesTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            SuperheroesTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    SuperheroesApp()
                }
            }
        }
    }
}

@Composable
fun SuperheroesApp() {
    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(
                        text = stringResource(R.string.app_name),
                        style = MaterialTheme.typography.displayLarge
                    )
                }
            )
        }
    ) { innerPadding ->
        HeroesList(modifier = Modifier.padding(innerPadding))
    }
}
## От края до края (Edge-to-edge)

## В Theme.kt добавьте setUpEdgeToEdge() и вызовите его из SideEffect.
private fun setUpEdgeToEdge(view: View, darkTheme: Boolean) {
    val window = (view.context as Activity).window
    WindowCompat.setDecorFitsSystemWindows(window, false)
    window.statusBarColor = Color.Transparent.toArgb()
    val navigationBarColor = when {
        Build.VERSION.SDK_INT >= 29 -> Color.Transparent.toArgb()
        Build.VERSION.SDK_INT >= 26 -> Color(0xFF, 0xFF, 0xFF, 0x63).toArgb()
        else -> Color(0x00, 0x00, 0x00, 0x50).toArgb()
    }
    window.navigationBarColor = navigationBarColor
    val controller = WindowCompat.getInsetsController(window, view)
    controller.isAppearanceLightStatusBars = !darkTheme
    controller.isAppearanceLightNavigationBars = !darkTheme
}
## И внутри SuperheroesTheme:
SideEffect {
    setUpEdgeToEdge(view, darkTheme)
}
## ПРИЛОЖЕНИЕ ДОЛЖНО:
Показывать список супергероев с их изображением, именем и описанием.
Иметь верхнюю панель с названием.
Поддерживать светлую и тёмную темы.
Иметь стилистику Material 3 с пользовательскими цветами, шрифтами и формами.
Работать от края до края (edge-to-edge UI).
