## Лекция 27. Advanced WorkManager и тестирование

Dependencies — app/build.gradle.kts
dependencies {
    implementation("androidx.activity:activity-compose:1.9.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.0")

    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")

    // Coil
    implementation("io.coil-kt:coil-compose:2.6.0")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

    // Testing
    testImplementation("junit:junit:4.13.2")
}

## 2. Data Layer
2.1 Модели JSON — data/BooksModels.kt
package com.example.bookshelf.data

data class BooksResponse(
    val items: List<BookItem>
)

data class BookItem(
    val id: String,
    val volumeInfo: VolumeInfo
)

data class VolumeInfo(
    val title: String,
    val imageLinks: ImageLinks?
)

data class ImageLinks(
    val thumbnail: String?
)

2.2 Retrofit API — network/BooksApiService.kt
package com.example.bookshelf.network

import com.example.bookshelf.data.BookItem
import com.example.bookshelf.data.BooksResponse
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query

interface BooksApiService {

    @GET("volumes")
    suspend fun searchBooks(
        @Query("q") query: String
    ): BooksResponse

    @GET("volumes/{id}")
    suspend fun getBook(
        @Path("id") id: String
    ): BookItem
}

2.3 Retrofit Instance — network/RetrofitInstance.kt
package com.example.bookshelf.network

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance {

    val api: BooksApiService by lazy {
        Retrofit.Builder()
            .baseUrl("https://www.googleapis.com/books/v1/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(BooksApiService::class.java)
    }
}

2.4 Repository interface — data/BooksRepository.kt
package com.example.bookshelf.data

interface BooksRepository {
    suspend fun loadBookThumbnails(query: String): List<String>
}

2.5 Repository implementation — data/NetworkBooksRepository.kt
package com.example.bookshelf.data

import com.example.bookshelf.network.BooksApiService

class NetworkBooksRepository(
    private val api: BooksApiService
) : BooksRepository {

    override suspend fun loadBookThumbnails(query: String): List<String> {
        val response = api.searchBooks(query)

        val thumbnails = mutableListOf<String>()

        for (item in response.items) {
            val book = api.getBook(item.id)

            val link = book.volumeInfo.imageLinks?.thumbnail
            if (link != null) {
                thumbnails.add(
                    link.replace("http://", "https://")
                )
            }
        }
        return thumbnails
    }
}

## 3. Dependency Injection — Application Container
3.1 Container — di/AppContainer.kt
package com.example.bookshelf.di

import com.example.bookshelf.data.BooksRepository
import com.example.bookshelf.data.NetworkBooksRepository
import com.example.bookshelf.network.RetrofitInstance

class AppContainer {
    val booksRepository: BooksRepository by lazy {
        NetworkBooksRepository(RetrofitInstance.api)
    }
}

3.2 Application class — BookshelfApplication.kt
package com.example.bookshelf

import android.app.Application
import com.example.bookshelf.di.AppContainer

class BookshelfApplication : Application() {
    lateinit var container: AppContainer

    override fun onCreate() {
        super.onCreate()
        container = AppContainer()
    }
}


AndroidManifest.xml:

<application
    android:name=".BookshelfApplication"

## 4. ViewModel — ui/BookshelfViewModel.kt
package com.example.bookshelf.ui

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.bookshelf.data.BooksRepository
import kotlinx.coroutines.launch

sealed interface BooksUiState {
    object Loading : BooksUiState
    data class Success(val thumbnails: List<String>) : BooksUiState
    data class Error(val message: String) : BooksUiState
}

class BookshelfViewModel(
    private val repository: BooksRepository
) : ViewModel() {

    var uiState: BooksUiState = BooksUiState.Loading
        private set

    fun load(query: String) {
        uiState = BooksUiState.Loading

        viewModelScope.launch {
            try {
                val urls = repository.loadBookThumbnails(query)
                uiState = BooksUiState.Success(urls)
            } catch (e: Exception) {
                uiState = BooksUiState.Error(e.message ?: "Error")
            }
        }
    }
}

## 5. UI Layer
5.1 Bookshelf Screen — ui/BookshelfScreen.kt
package com.example.bookshelf.ui

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyVerticalGrid
import androidx.compose.foundation.lazy.grid.items
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage

@Composable
fun BookshelfScreen(state: BooksUiState) {
    when (state) {
        BooksUiState.Loading -> CircularProgressIndicator()
        is BooksUiState.Error -> Text("Ошибка загрузки: ${state.message}")
        is BooksUiState.Success -> BooksGrid(state.thumbnails)
    }
}

@Composable
fun BooksGrid(urls: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Adaptive(120.dp),
        modifier = Modifier.fillMaxSize().padding(8.dp)
    ) {
        items(urls) { url ->
            AsyncImage(
                model = url,
                contentDescription = null,
                modifier = Modifier
                    .padding(4.dp)
                    .height(180.dp)
            )
        }
    }
}

## 6. MainActivity — подключаем ViewModel к DI
package com.example.bookshelf

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.bookshelf.ui.*

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val appContainer = (application as BookshelfApplication).container

        setContent {
            val viewModel: BookshelfViewModel = viewModel(
                factory = BookshelfViewModelFactory(appContainer.booksRepository)
            )

            viewModel.load("android")

            BookshelfScreen(viewModel.uiState)
        }
    }
}

6.1 Factory — ui/BookshelfViewModelFactory.kt
package com.example.bookshelf.ui

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.bookshelf.data.BooksRepository

class BookshelfViewModelFactory(
    private val repository: BooksRepository
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return BookshelfViewModel(repository) as T
    }
}

## 7. Тестирование
Fake Service — test/FakeBooksApiService.kt
package com.example.bookshelf

import com.example.bookshelf.data.*
import com.example.bookshelf.network.BooksApiService

class FakeBooksApiService : BooksApiService {

    override suspend fun searchBooks(query: String): BooksResponse {
        return BooksResponse(
            items = listOf(
                BookItem("id1", VolumeInfo("Book1", null)),
                BookItem("id2", VolumeInfo("Book2", null))
            )
        )
    }

    override suspend fun getBook(id: String): BookItem {
        return BookItem(id, VolumeInfo(
            "Book $id",
            ImageLinks("http://example.com/$id.jpg")
        ))
    }
}

Repository Test — test/NetworkBooksRepositoryTest.kt
package com.example.bookshelf

import com.example.bookshelf.data.NetworkBooksRepository
import kotlinx.coroutines.runBlocking
import org.junit.Assert.assertEquals
import org.junit.Test

class NetworkBooksRepositoryTest {

    @Test
    fun `repository converts http to https`() = runBlocking {
        val repo = NetworkBooksRepository(FakeBooksApiService())

        val result = repo.loadBookThumbnails("android")

        assertEquals("https://example.com/id1.jpg", result[0])
        assertEquals("https://example.com/id2.jpg", result[1])
    }
}