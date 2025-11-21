## Практическая работа 24. Загрузка и отображение изображений из Интернета

## Подготовка: добавление зависимости Coil

Открой:

app/build.gradle.kts


Добавь:

implementation("io.coil-kt:coil-compose:2.4.0")


Нажми Sync Now.

## Обновление ViewModel: отображение URL первой фотографии (временный шаг)

В MarsViewModel.kt, в getMarsPhotos():

Изменяем:

val listResult = marsPhotosRepository.getMarsPhotos()


НА:

val result = marsPhotosRepository.getMarsPhotos()[0]


И временно возвращаем URL:

marsUiState = MarsUiState.Success("First Mars image URL: ${result.imgSrc}")


Запустить → должно показывать URL.

## Добавляем отображение изображения с AsyncImage
Создаём MarsPhotoCard

Открой:

ui/screens/HomeScreen.kt


Добавь:

@Composable
fun MarsPhotoCard(photo: MarsPhoto, modifier: Modifier = Modifier) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(photo.imgSrc)
            .crossfade(true)
            .build(),
        contentDescription = stringResource(R.string.mars_photo),
        modifier = modifier.fillMaxWidth()
    )
}


Добавь ContentScale.Crop:

contentScale = ContentScale.Crop

## Обновляем MarsUiState и ViewModel на объект MarsPhoto
В MarsUiState:
sealed interface MarsUiState {
    data class Success(val photos: MarsPhoto) : MarsUiState
    object Error : MarsUiState
    object Loading : MarsUiState
}

В ViewModel:
marsUiState = try {
    MarsUiState.Success(marsPhotosRepository.getMarsPhotos()[0])
}

## Добавляем placeholder + error изображения

В MarsPhotoCard:

error = painterResource(R.drawable.ic_broken_image),
placeholder = painterResource(R.drawable.loading_img),


Полный код:

@Composable
fun MarsPhotoCard(photo: MarsPhoto, modifier: Modifier = Modifier) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(photo.imgSrc)
            .crossfade(true)
            .build(),
        error = painterResource(R.drawable.ic_broken_image),
        placeholder = painterResource(R.drawable.loading_img),
        contentDescription = stringResource(R.string.mars_photo),
        contentScale = ContentScale.Crop,
        modifier = modifier
    )
}

## Переход к сетке: создаём LazyVerticalGrid
Создаём PhotosGridScreen
@Composable
fun PhotosGridScreen(
    photos: List<MarsPhoto>,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(0.dp)
) {
    LazyVerticalGrid(
        columns = GridCells.Adaptive(minSize = 150.dp),
        modifier = modifier.padding(horizontal = 4.dp),
        contentPadding = contentPadding
    ) {
        items(photos, key = { it.id }) { photo ->
            MarsPhotoCard(
                photo = photo,
                modifier = Modifier
                    .padding(4.dp)
                    .fillMaxWidth()
                    .aspectRatio(1.5f)
            )
        }
    }
}

## Обновляем MarsUiState на список

В MarsUiState:

data class Success(val photos: List<MarsPhoto>) : MarsUiState

## ViewModel возвращает список

В getMarsPhotos():

marsUiState = try {
    MarsUiState.Success(marsPhotosRepository.getMarsPhotos())
} catch (e: IOException) {
    MarsUiState.Error
}
## Обновляем HomeScreen
@Composable
fun HomeScreen(
    marsUiState: MarsUiState,
    retryAction: () -> Unit,
    modifier: Modifier = Modifier
) {
    when (marsUiState) {
        is MarsUiState.Loading ->
            LoadingScreen(modifier.fillMaxSize())

        is MarsUiState.Success ->
            PhotosGridScreen(
                photos = marsUiState.photos,
                modifier = modifier.fillMaxSize()
            )

        is MarsUiState.Error ->
            ErrorScreen(retryAction = retryAction, modifier = modifier.fillMaxSize())
    }
}

## Добавляем Retry кнопку

В ErrorScreen:

@Composable
fun ErrorScreen(
    retryAction: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier,
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Image(
            painter = painterResource(id = R.drawable.ic_connection_error),
            contentDescription = null
        )
        Text(stringResource(R.string.loading_failed))
        Button(onClick = retryAction) {
            Text(stringResource(R.string.retry))
        }
    }
}

## Обновляем вызов HomeScreen в MarsPhotosApp
HomeScreen(
    marsUiState = marsViewModel.marsUiState,
    retryAction = marsViewModel::getMarsPhotos
)

## Обновляем тест ViewModel

Открыть:

rules/MarsViewModelTest.kt


Используем FakeDataSource.photosList:

@Test
fun marsViewModel_getMarsPhotos_verifyMarsUiStateSuccess() = runTest {
    val marsViewModel = MarsViewModel(
        marsPhotosRepository = FakeNetworkMarsPhotosRepository()
    )

    assertEquals(
        MarsUiState.Success(FakeDataSource.photosList),
        marsViewModel.marsUiState
    )
}