## Практическая работа 26. Создание приложения "Книжная полка"
## Создание проекта

Empty Activity → Bookshelf

## 2. Добавление зависимостей

В app/build.gradle.kts:

plugins {
    kotlin("plugin.serialization") version "1.9.0"
}

dependencies {
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")

    // Gson
    implementation("com.google.code.gson:gson:2.10")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.1")

    // Coil
    implementation("io.coil-kt:coil-compose:2.4.0")

    // ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.6.2")
}

## 3. Модель данных (Gson)

Структура Google Books API:

items → [ { id, volumeInfo { title, authors[], imageLinks{ thumbnail } } } ]


Создаём модели:

data/model/Book.kt
package com.example.bookshelf.data.model

data class BooksResponse(
    val items: List<BookItem>?
)

data class BookItem(
    val id: String,
    val volumeInfo: VolumeInfo
)

data class VolumeInfo(
    val title: String?,
    val authors: List<String>?,
    val imageLinks: ImageLinks?
)

data class ImageLinks(
    val thumbnail: String?
)

## 4. Слой сети (Retrofit)
data/network/BooksApiService.kt
package com.example.bookshelf.data.network

import com.example.bookshelf.data.model.BooksResponse
import com.example.bookshelf.data.model.BookItem
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query

interface BooksApiService {

    @GET("volumes")
    suspend fun searchBooks(
        @Query("q") query: String
    ): BooksResponse

    @GET("volumes/{id}")
    suspend fun getBookInfo(
        @Path("id") id: String
    ): BookItem
}

data/network/BooksApi.kt
package com.example.bookshelf.data.network

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

private const val BASE_URL = "https://www.googleapis.com/books/v1/"

object BooksApi {
    val retrofitService: BooksApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(BooksApiService::class.java)
    }
}

## 5. Репозиторий (с последовательными запросами!)

Загрузка миниатюр:

выполнить searchBooks

для каждого id — выполнить getBookInfo

собрать все thumbnail

data/repository/BooksRepository.kt
package com.example.bookshelf.data.repository

import com.example.bookshelf.data.model.BookItem

interface BooksRepository {
    suspend fun loadBooks(query: String): List<BookItem>
}

data/repository/NetworkBooksRepository.kt
package com.example.bookshelf.data.repository

import com.example.bookshelf.data.model.BookItem
import com.example.bookshelf.data.network.BooksApi

class NetworkBooksRepository : BooksRepository {

    override suspend fun loadBooks(query: String): List<BookItem> {
        val searchResult = BooksApi.retrofitService.searchBooks(query)

        val resultItems = mutableListOf<BookItem>()

        searchResult.items?.forEach { item ->
            val details = BooksApi.retrofitService.getBookInfo(item.id)
            resultItems.add(details)
        }

        return resultItems
    }
}

## 6. DI-контейнер
data/AppContainer.kt
package com.example.bookshelf.data

import com.example.bookshelf.data.repository.BooksRepository
import com.example.bookshelf.data.repository.NetworkBooksRepository

interface AppContainer {
    val booksRepository: BooksRepository
}

class DefaultAppContainer : AppContainer {
    override val booksRepository = NetworkBooksRepository()
}

## 7. Application-класс
BookshelfApplication.kt
package com.example.bookshelf

import android.app.Application
import com.example.bookshelf.data.AppContainer
import com.example.bookshelf.data.DefaultAppContainer

class BookshelfApplication : Application() {

    lateinit var container: AppContainer
        private set

    override fun onCreate() {
        super.onCreate()
        container = DefaultAppContainer()
    }
}


AndroidManifest.xml:

android:name=".BookshelfApplication"

## 8. ViewModel
ui/BookshelfViewModel.kt
package com.example.bookshelf.ui

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.bookshelf.data.model.BookItem
import com.example.bookshelf.data.repository.BooksRepository
import kotlinx.coroutines.launch

sealed interface BooksUiState {
    object Loading : BooksUiState
    object Error : BooksUiState
    data class Success(val books: List<BookItem>) : BooksUiState
}

class BookshelfViewModel(
    private val repository: BooksRepository
) : ViewModel() {

    var uiState: BooksUiState = BooksUiState.Loading
        private set

    init {
        loadBooks("android development")
    }

    fun loadBooks(query: String) {
        uiState = BooksUiState.Loading
        viewModelScope.launch {
            uiState = try {
                val books = repository.loadBooks(query)
                BooksUiState.Success(books)
            } catch (e: Exception) {
                BooksUiState.Error
            }
        }
    }
}

## 9. UI: карточка книги + сетка
⚠ Важно: заменяем http → https
val thumbnail = book.volumeInfo.imageLinks?.thumbnail?.replace("http://", "https://")

ui/BookCard.kt
@Composable
fun BookCard(book: BookItem) {
    val thumbnail = book.volumeInfo.imageLinks?.thumbnail?.replace("http://", "https://")

    Card(
        modifier = Modifier.padding(8.dp),
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column {
            AsyncImage(
                model = thumbnail,
                contentDescription = book.volumeInfo.title,
                contentScale = ContentScale.Crop,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(180.dp)
            )

            Text(
                text = book.volumeInfo.title ?: "No title",
                style = MaterialTheme.typography.titleMedium,
                modifier = Modifier.padding(8.dp)
            )
        }
    }
}

Список книг в виде сетки
@Composable
fun BooksGrid(books: List<BookItem>) {
    LazyVerticalGrid(columns = GridCells.Adaptive(150.dp)) {
        items(books, key = { it.id }) { book ->
            BookCard(book)
        }
    }
}

## 10. Экран HomeScreen
@Composable
fun HomeScreen(uiState: BooksUiState, retry: () -> Unit) {
    when (uiState) {
        is BooksUiState.Loading -> LoadingScreen()
        is BooksUiState.Error -> ErrorScreen(retry)
        is BooksUiState.Success -> BooksGrid(uiState.books)
    }
}

## 11. MainActivity
создаём ViewModel с DI
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val app = application as BookshelfApplication

        val viewModel: BookshelfViewModel = viewModel(
            factory = BookshelfViewModelFactory(app.container.booksRepository)
        )

        setContent {
            BookshelfTheme {
                HomeScreen(
                    uiState = viewModel.uiState,
                    retry = { viewModel.loadBooks("android")} 
                )
            }
        }
    }
}

## 12. Тестирование: фальшивый репозиторий
test/FakeBooksRepository.kt
class FakeBooksRepository : BooksRepository {
    override suspend fun loadBooks(query: String): List<BookItem> {
        return listOf(
            BookItem(
                id = "1",
                volumeInfo = VolumeInfo(
                    title = "Fake Book",
                    authors = listOf("Tester"),
                    imageLinks = ImageLinks("https://example.com/image.jpg")
                )
            )
        )
    }
}

BookshelfViewModelTest.kt
@Test
fun bookshelfViewModel_loadBooks_success() = runTest {
    val vm = BookshelfViewModel(FakeBooksRepository())

    assertTrue(vm.uiState is BooksUiState.Success)
}
