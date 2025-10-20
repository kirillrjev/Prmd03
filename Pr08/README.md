## Пример реализации
@Composable
fun FourQuadrantsScreen() {
    Column(modifier = Modifier.fillMaxSize()) {
        Row(modifier = Modifier.weight(1f)) {
            Quadrant(
                title = stringResource(R.string.composable_text),
                description = stringResource(R.string.composable_text_desc),
                backgroundColor = Color(0xFFEADDFF),
                modifier = Modifier.weight(1f)
            )
            Quadrant(
                title = stringResource(R.string.image_composable),
                description = stringResource(R.string.image_composable_desc),
                backgroundColor = Color(0xFFD0BCFF),
                modifier = Modifier.weight(1f)
            )
        }
        Row(modifier = Modifier.weight(1f)) {
            Quadrant(
                title = stringResource(R.string.composable_row),
                description = stringResource(R.string.composable_row_desc),
                backgroundColor = Color(0xFFB69DF8),
                modifier = Modifier.weight(1f)
            )
            Quadrant(
                title = stringResource(R.string.composable_column),
                description = stringResource(R.string.composable_column_desc),
                backgroundColor = Color(0xFFF6EDFF),
                modifier = Modifier.weight(1f)
            )
        }
    }
}

@Composable
fun Quadrant(
    title: String,
    description: String,
    backgroundColor: Color,
    modifier: Modifier = Modifier
) {
    Box(
        modifier = modifier
            .fillMaxSize()
            .background(backgroundColor)
            .padding(16.dp)
    ) {
        Column(
            modifier = Modifier.fillMaxSize(),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = title,
                fontWeight = FontWeight.Bold,
                // Можно использовать style = MaterialTheme.typography.titleMedium etc.
                modifier = Modifier.padding(bottom = 16.dp)
            )
            Text(text = description)
        }
    }
}
## strings.xml
<string name="composable_text">Композитный текст</string>
<string name="composable_text_desc">Отображает текст и следует рекомендованным рекомендациям Material Design.</string>

<string name="image_composable">Image composable</string>
<string name="image_composable_desc">Создает композитное изображение, которое выстраивает и рисует заданный объект класса Painter.</string>

<string name="composable_row">Составная строка</string>
<string name="composable_row_desc">Составной элемент макета, который размещает свои дочерние элементы в горизонтальной последовательности.</string>

<string name="composable_column">Составной столбец</string>
<string name="composable_column_desc">Композит макета, который размещает свои дочерние элементы в вертикальной последовательности.</string>
