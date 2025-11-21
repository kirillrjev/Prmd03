## Практическая работа 25. Работа с API

## Шаг 1. Создай новый проект

Empty Activity → Amphibians

##     Шаг 2. Добавь зависимости в app/build.gradle.kts
dependencies {
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-scalars:2.9.0")

    // Kotlinx Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.1")
    implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:0.8.0")

    // Coil
    implementation("io.coil-kt:coil-compose:2.4.0")

    // Lifecycle + ViewModel
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.6.2")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
}


Не забудь:

plugins {
    kotlin("plugin.serialization") version "1.9.0"
}

##     Шаг 3. Создай модель данных

data/model/Amphibian.kt

package com.example.amphibians.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class Amphibian(
    val name: String,
    val type: String,
    val description: String,
    @SerialName("img_src") val imageSrc: String
)

##     Шаг 4. Настрой сеть (Retrofit)

data/network/AmphibiansApiService.kt

package com.example.amphibians.data.network

import com.example.amphibians.data.model.Amphibian
import retrofit2.http.GET

interface AmphibiansApiService {
    @GET("amphibians")
    suspend fun getAmphibians(): List<Amphibian>
}


data/network/AmphibiansApi.kt

package com.example.amphibians.data.network

import com.jakewharton.retrofit2.converter.kotlinx.serialization.asConverterFactory
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType
import retrofit2.Retrofit

private const val BASE_URL =
    "https://android-kotlin-fun-mars-server.appspot.com/"

private val json = Json { ignoreUnknownKeys = true }

object AmphibiansApi {
    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
        .build()

    val retrofitService: AmphibiansApiService by lazy {
        retrofit.create(AmphibiansApiService::class.java)
    }
}

##    Шаг 5. Создай репозиторий

data/repository/AmphibiansRepository.kt

package com.example.amphibians.data.repository

import com.example.amphibians.data.model.Amphibian

interface AmphibiansRepository {
    suspend fun getAmphibians(): List<Amphibian>
}


data/repository/NetworkAmphibiansRepository.kt

package com.example.amphibians.data.repository

import com.example.amphibians.data.model.Amphibian
import com.example.amphibians.data.network.AmphibiansApi

class NetworkAmphibiansRepository : AmphibiansRepository {
    override suspend fun getAmphibians(): List<Amphibian> {
        return AmphibiansApi.retrofitService.getAmphibians()
    }
}

##     Шаг 6. DI-контейнер приложения

data/AppContainer.kt

package com.example.amphibians.data

import com.example.amphibians.data.repository.AmphibiansRepository
import com.example.amphibians.data.repository.NetworkAmphibiansRepository

interface AppContainer {
    val repository: AmphibiansRepository
}

class DefaultAppContainer : AppContainer {
    override val repository: AmphibiansRepository = NetworkAmphibiansRepository()
}

##     Шаг 7. Application-класс

AmphibiansApplication.kt

package com.example.amphibians

import android.app.Application
import com.example.amphibians.data.AppContainer
import com.example.amphibians.data.DefaultAppContainer

class AmphibiansApplication : Application() {
    lateinit var container: AppContainer
        private set

    override fun onCreate() {
        super.onCreate()
        container = DefaultAppContainer()
    }
}


И добавь в AndroidManifest.xml:

android:name=".AmphibiansApplication"

##     Шаг 8. ViewModel

ui/screens/AmphibiansViewModel.kt

package com.example.amphibians.ui.screens

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.amphibians.data.model.Amphibian
import com.example.amphibians.data.repository.AmphibiansRepository
import kotlinx.coroutines.launch

sealed interface AmphibiansUiState {
    object Loading : AmphibiansUiState
    object Error : AmphibiansUiState
    data class Success(val amphibians: List<Amphibian>) : AmphibiansUiState
}

class AmphibiansViewModel(private val repository: AmphibiansRepository) : ViewModel() {

    var uiState: AmphibiansUiState = AmphibiansUiState.Loading
        private set

    init { loadAmphibians() }

    fun loadAmphibians() {
        uiState = AmphibiansUiState.Loading
        viewModelScope.launch {
            uiState = try {
                AmphibiansUiState.Success(repository.getAmphibians())
            } catch (e: Exception) {
                AmphibiansUiState.Error
            }
        }
    }
}

##    Шаг 9. Экран (UI): список амфибий

ui/screens/HomeScreen.kt

@Composable
fun AmphibianCard(amph: Amphibian, modifier: Modifier = Modifier) {
    Card(
        modifier = modifier.padding(8.dp),
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(Modifier.padding(16.dp)) {
            Text(amph.name, style = MaterialTheme.typography.titleLarge)
            Text(amph.type, style = MaterialTheme.typography.titleMedium)

            Spacer(Modifier.height(8.dp))

            AsyncImage(
                model = amph.imageSrc,
                contentDescription = amph.name,
                contentScale = ContentScale.Crop,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(200.dp)
            )

            Spacer(Modifier.height(8.dp))

            Text(amph.description)
        }
    }
}

Экран со списком
@Composable
fun HomeScreen(
    uiState: AmphibiansUiState,
    retry: () -> Unit
) {
    when (uiState) {
        is AmphibiansUiState.Loading -> LoadingScreen()
        is AmphibiansUiState.Error -> ErrorScreen(retry)
        is AmphibiansUiState.Success -> AmphibiansList(uiState.amphibians)
    }
}

Список
@Composable
fun AmphibiansList(amphibians: List<Amphibian>) {
    LazyColumn {
        items(amphibians) { amph ->
            AmphibianCard(amph)
        }
    }
}

##  Шаг 10. MainActivity
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val viewModel: AmphibiansViewModel = viewModel(
            factory = AmphibiansViewModelFactory(
                (application as AmphibiansApplication).container.repository
            )
        )

        setContent {
            AmphibiansTheme {
                HomeScreen(
                    uiState = viewModel.uiState,
                    retry = viewModel::loadAmphibians
                )
            }
        }
    }
}