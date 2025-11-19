## Практическая работа 17. Адаптация под разные размеры. Динамической навигация
Необходимые условия

    Знакомство с программированием на Kotlin, включая классы, функции и условия
    Опыт использования классов ViewModel
    Опыт создания композитных функций
    Опыт создания макетов с помощью Jetpack Compose
    Опыт запуска приложений на устройстве или эмуляторе

Что вы узнаете

    Как создать навигацию между экранами без Navigation Graph для простых приложений
    Как создать адаптивный макет навигации с помощью Jetpack Compose
    Как создать пользовательский обработчик возврата

## Часть 1. Переключение экранов без Navigation Graph
ReplyHomeScreen.kt (полное решение фрагмента)
@Composable
fun ReplyHomeScreen(
    navigationType: ReplyNavigationType,
    replyUiState: ReplyUiState,
    onTabPressed: (MailboxType) -> Unit,
    onEmailCardPressed: (Email) -> Unit,
    onDetailScreenBackPressed: () -> Unit,
    modifier: Modifier = Modifier
) {

    val navigationItemContentList = ReplyNavigationContentList

    if (navigationType == ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER
        && replyUiState.isShowingHomepage
    ) {
        PermanentNavigationDrawer(
            drawerContent = {
                PermanentDrawerSheet(Modifier.width(dimensionResource(R.dimen.drawer_width))) {
                    NavigationDrawerContent(
                        selectedDestination = replyUiState.currentMailbox,
                        onTabPressed = onTabPressed,
                        navigationItemContentList = navigationItemContentList,
                        modifier = Modifier
                            .wrapContentWidth()
                            .fillMaxHeight()
                            .background(MaterialTheme.colorScheme.inverseOnSurface)
                            .padding(dimensionResource(R.dimen.drawer_padding_content))
                    )
                }
            }
        ) {
            ReplyAppContent(
                navigationType = navigationType,
                replyUiState = replyUiState,
                onTabPressed = onTabPressed,
                onEmailCardPressed = onEmailCardPressed,
                navigationItemContentList = navigationItemContentList,
                modifier = modifier
            )
        }
    } else {
        if (replyUiState.isShowingHomepage) {
            ReplyAppContent(
                navigationType = navigationType,
                replyUiState = replyUiState,
                onTabPressed = onTabPressed,
                onEmailCardPressed = onEmailCardPressed,
                navigationItemContentList = navigationItemContentList,
                modifier = modifier
            )
        } else {
            ReplyDetailsScreen(
                replyUiState = replyUiState,
                onBackPressed = onDetailScreenBackPressed,
                modifier = modifier
            )
        }
    }
}

## Часть 2. Экран деталей + BackHandler
ReplyDetailsScreen.kt
import androidx.activity.compose.BackHandler

@Composable
fun ReplyDetailsScreen(
    replyUiState: ReplyUiState,
    onBackPressed: () -> Unit,
    modifier: Modifier = Modifier
) {
    BackHandler { onBackPressed() }

    // ... остальная верстка экрана деталей
}

## Часть 3. Подключение WindowSizeClass
build.gradle.kts
implementation("androidx.compose.material3:material3-window-size-class")

MainActivity.kt
@OptIn(ExperimentalMaterial3WindowSizeClassApi::class)
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    setContent {
        ReplyTheme {
            Surface {
                val windowSize = calculateWindowSizeClass(this)

                ReplyApp(
                    windowSize = windowSize.widthSizeClass
                )
            }
        }
    }
}

## Часть 4. ReplyApp получает WindowWidthSizeClass и выбирает навигацию
ReplyApp.kt
@Composable
fun ReplyApp(
    windowSize: WindowWidthSizeClass,
    modifier: Modifier = Modifier
) {
    val viewModel: ReplyViewModel = viewModel()
    val replyUiState by viewModel.uiState.collectAsState()

    val navigationType: ReplyNavigationType =
        when (windowSize) {
            WindowWidthSizeClass.Compact ->
                ReplyNavigationType.BOTTOM_NAVIGATION

            WindowWidthSizeClass.Medium ->
                ReplyNavigationType.NAVIGATION_RAIL

            WindowWidthSizeClass.Expanded ->
                ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER

            else -> ReplyNavigationType.BOTTOM_NAVIGATION
        }

    ReplyHomeScreen(
        navigationType = navigationType,
        replyUiState = replyUiState,
        onTabPressed = {
            viewModel.updateCurrentMailbox(it)
            viewModel.resetHomeScreenStates()
        },
        onEmailCardPressed = {
            viewModel.updateDetailsScreenStates(it)
        },
        onDetailScreenBackPressed = {
            viewModel.resetHomeScreenStates()
        },
        modifier = modifier
    )
}

## Часть 5. Добавление ReplyNavigationType enum
WindowStateUtils.kt
package com.example.reply.ui.utils

enum class ReplyNavigationType {
    BOTTOM_NAVIGATION,
    NAVIGATION_RAIL,
    PERMANENT_NAVIGATION_DRAWER
}

## Часть 6. Адаптивный контент — ReplyAppContent
ReplyHomeScreen.kt → ReplyAppContent
@Composable
private fun ReplyAppContent(
    navigationType: ReplyNavigationType,
    replyUiState: ReplyUiState,
    onTabPressed: (MailboxType) -> Unit,
    onEmailCardPressed: (Email) -> Unit,
    navigationItemContentList: List<NavigationItemContent>,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier) {

        // Navigation Rail
        AnimatedVisibility(visible = navigationType == ReplyNavigationType.NAVIGATION_RAIL) {
            ReplyNavigationRail(
                currentTab = replyUiState.currentMailbox,
                onTabPressed = onTabPressed,
                navigationItemContentList = navigationItemContentList
            )
        }

        // Main content
        Column(
            modifier = Modifier
                .fillMaxSize()
                .background(MaterialTheme.colorScheme.inverseOnSurface)
        ) {
            ReplyListOnlyContent(
                replyUiState = replyUiState,
                onEmailCardPressed = onEmailCardPressed,
                modifier = Modifier
                    .weight(1f)
                    .padding(
                        horizontal = dimensionResource(R.dimen.email_list_only_horizontal_padding)
                    )
            )

            // Bottom nav (only compact)
            AnimatedVisibility(visible = navigationType == ReplyNavigationType.BOTTOM_NAVIGATION) {
                ReplyBottomNavigationBar(
                    currentTab = replyUiState.currentMailbox,
                    onTabPressed = onTabPressed,
                    navigationItemContentList = navigationItemContentList,
                    modifier = Modifier.fillMaxWidth()
                )
            }
        }
    }
}
