## Практическая работа 16
Добавление навигации в приложение Lunch Tray

Цель:
Научиться добавлять навигацию в приложение Jetpack Compose, используя:

NavController

NavHost

Scaffold + верхняя панель (AppBar)

Enum для экранов

Навигационные действия

Приложение состоит из 5 экранов:

Start

Entree menu

Side dish menu

Accompaniment menu

Checkout

## Создаём перечисление экранов (enum)

Создайте файл Screen.kt:

package com.example.lunchtray.ui

import androidx.annotation.StringRes
import com.example.lunchtray.R

enum class Screen(@StringRes val title: Int) {
    Start(R.string.start_order),
    Entree(R.string.choose_entree),
    Side(R.string.choose_side),
    Accompaniment(R.string.choose_accompaniment),
    Checkout(R.string.order_checkout)
}
## Создаём NavController и отслеживаем текущий экран

В LunchTrayApp.kt (главный composable):

@Composable
fun LunchTrayApp() {
    val navController = rememberNavController()

    val backStackEntry by navController.currentBackStackEntryAsState()

    val currentScreen = Screen.valueOf(
        backStackEntry?.destination?.route ?: Screen.Start.name
    )

    Scaffold(
        topBar = {
            LunchTrayAppBar(
                currentScreen = currentScreen,
                canNavigateBack = navController.previousBackStackEntry != null,
                navigateUp = { navController.navigateUp() }
            )
        }
    ) { innerPadding ->

        NavHost(
            navController = navController,
            startDestination = Screen.Start.name,
            modifier = Modifier.padding(innerPadding)
        ) {
            // Навигация будет добавлена на следующем шаге
        }
    }
}

## Создаём AppBar

Создайте файл LunchTrayAppBar.kt:

@Composable
fun LunchTrayAppBar(
    currentScreen: Screen,
    canNavigateBack: Boolean,
    navigateUp: () -> Unit
) {
    TopAppBar(
        title = { Text(stringResource(currentScreen.title)) },
        navigationIcon = {
            if (canNavigateBack) {
                IconButton(onClick = navigateUp) {
                    Icon(
                        imageVector = Icons.Filled.ArrowBack,
                        contentDescription = stringResource(R.string.back_button)
                    )
                }
            }
        }
    )
}

## Требования выполнены:

✔ отображение заголовка
✔ кнопка "назад" только если это не стартовый экран
✔ использован Icons.Filled.ArrowBack

## Настраиваем NavHost (маршруты)

В LunchTrayApp() внутри NavHost:

NavHost(
    navController = navController,
    startDestination = Screen.Start.name
) {
    composable(Screen.Start.name) {
        StartOrderScreen(
            onStartOrder = { navController.navigate(Screen.Entree.name) }
        )
    }

    composable(Screen.Entree.name) {
        EntreeMenuScreen(
            onNext = { navController.navigate(Screen.Side.name) },
            onCancel = { navController.popBackStack(Screen.Start.name, inclusive = false) }
        )
    }

    composable(Screen.Side.name) {
        SideDishMenuScreen(
            onNext = { navController.navigate(Screen.Accompaniment.name) },
            onCancel = { navController.popBackStack(Screen.Start.name, false) }
        )
    }

    composable(Screen.Accompaniment.name) {
        AccompanimentMenuScreen(
            onNext = { navController.navigate(Screen.Checkout.name) },
            onCancel = { navController.popBackStack(Screen.Start.name, false) }
        )
    }

    composable(Screen.Checkout.name) {
        CheckoutScreen(
            onSubmit = {
                navController.popBackStack(Screen.Start.name, false)
            },
            onCancel = {
                navController.popBackStack(Screen.Start.name, false)
            }
        )
    }
}

## Логика соответствует требованиям:

✔ Start → Entree
✔ Entree → Side
✔ Side → Accompaniment
✔ Accompaniment → Checkout
✔ Checkout → Start (Submit)
✔ Cancel на любом экране → Start

## Итоговый поток приложения
Start → Entree → Side → Accompaniment → Checkout → Start