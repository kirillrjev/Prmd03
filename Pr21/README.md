## Практическая работа 21: Добавление ViewModel в приложение
##build.gradle (уровень проекта)

## Добавьте переменную версии:

buildscript {
    ext {
        ...
        lifecycle_version = '2.5.1'
    }
}

В app/build.gradle

## Добавьте зависимость ViewModel для Compose:

dependencies {
    ...
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:$lifecycle_version"
}


Синхронизируйте проект.

## Создайте класс UI state

Создайте файл DessertUiState.kt:

package com.example.dessertclicker.ui

data class DessertUiState(
    val revenue: Int = 0,
    val dessertsSold: Int = 0,
    val currentDessertIndex: Int = 0
)


Можно добавлять другие поля при необходимости (например, текущий объект Dessert).

## Создайте ViewModel

Создайте файл DessertViewModel.kt:

package com.example.dessertclicker.ui

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import com.example.dessertclicker.datasource.DessertDatasource

class DessertViewModel : ViewModel() {

    private val desserts = DessertDatasource.dessertList

    private val _uiState = MutableStateFlow(DessertUiState())
    val uiState = _uiState.asStateFlow()

    fun onDessertClicked() {
        val state = _uiState.value

        val newSold = state.dessertsSold + 1
        val newRevenue = state.revenue + desserts[state.currentDessertIndex].price
        val newIndex = determineDessertIndex(newSold)

        _uiState.value = state.copy(
            revenue = newRevenue,
            dessertsSold = newSold,
            currentDessertIndex = newIndex
        )
    }

    private fun determineDessertIndex(dessertsSold: Int): Int {
        var index = 0
        while (index < desserts.size - 1 &&
            dessertsSold >= desserts[index].startProductionAmount
        ) {
            index++
        }
        return index
    }
}

## Удалите состояние из MainActivity

Теперь MainActivity не должна содержать ни одного remember { ... }, mutableStateOf, ни данных.

## Используйте ViewModel в Compose

В MainActivity замените вызов DessertClickerApp() так:

setContent {
    val viewModel: DessertViewModel = viewModel()
    val uiState by viewModel.uiState.collectAsState()

    DessertClickerApp(
        uiState = uiState,
        onDessertClicked = { viewModel.onDessertClicked() }
    )
}

## Обновите composable DessertClickerApp
@Composable
fun DessertClickerApp(
    uiState: DessertUiState,
    onDessertClicked: () -> Unit
) {
    val desserts = DessertDatasource.dessertList
    val currentDessert = desserts[uiState.currentDessertIndex]

    DessertClickerScreen(
        revenue = uiState.revenue,
        dessertsSold = uiState.dessertsSold,
        dessertImageId = currentDessert.imageId,
        onDessertClick = onDessertClicked
    )
}