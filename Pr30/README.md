## Практическая работа 30. Views в Compose

## Spinner в Compose

Создаем класс SpinnerAdapter, который реализует AdapterView.OnItemSelectedListener:

class SpinnerAdapter(val onColorChange: (Int) -> Unit) : AdapterView.OnItemSelectedListener {
    override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
        onColorChange(position)
    }

    override fun onNothingSelected(parent: AdapterView<*>?) {
        onColorChange(0)
    }
}


Создаем Composable ColorSpinnerRow:

@Composable
fun ColorSpinnerRow(
    colorSpinnerPosition: Int,
    onColorChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    val juiceColorArray = JuiceColor.values().map { stringResource(it.label) }

    InputRow(inputLabel = stringResource(R.string.color), modifier = modifier) {
        AndroidView(
            modifier = Modifier.fillMaxWidth(),
            factory = { context ->
                Spinner(context).apply {
                    adapter = ArrayAdapter(
                        context,
                        android.R.layout.simple_spinner_dropdown_item,
                        juiceColorArray
                    )
                }
            },
            update = { spinner ->
                spinner.setSelection(colorSpinnerPosition)
                spinner.onItemSelectedListener = SpinnerAdapter(onColorChange)
            }
        )
    }
}


Использование: в EntryBottomSheet.kt после TextInputRow для описания.

## 2. RatingBar в Compose

Создаем Composable RatingInputRow:

@Composable
fun RatingInputRow(
    rating: Int,
    onRatingChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    InputRow(inputLabel = stringResource(R.string.rating), modifier = modifier) {
        AndroidView(
            factory = { context ->
                RatingBar(context).apply { stepSize = 1f }
            },
            update = { ratingBar ->
                ratingBar.rating = rating.toFloat()
                ratingBar.setOnRatingBarChangeListener { _, _, _ ->
                    onRatingChange(ratingBar.rating.toInt())
                }
            }
        )
    }
}


Использование: в EntryBottomSheet после ColorSpinnerRow и перед кнопками.

## 3. AdView в Compose

Создаем Composable AdBanner:

@Composable
fun AdBanner(modifier: Modifier = Modifier) {
    AndroidView(
        modifier = modifier,
        factory = { context ->
            AdView(context).apply {
                setAdSize(AdSize.BANNER)
                adUnitId = "ca-app-pub-3940256099942544/6300978111"
            }
        },
        update = { adView ->
            adView.loadAd(AdRequest.Builder().build())
        }
    )
}


Использование: в JuiceTrackerApp.kt перед списком соков:

AdBanner(
    Modifier
        .fillMaxWidth()
        .padding(top = dimensionResource(R.dimen.padding_medium),
                 bottom = dimensionResource(R.dimen.padding_small))
)
