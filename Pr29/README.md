## Практическая работа 29. Добавление Compose в приложение, основанное на представлении

build.gradle.kts — что нужно добавить
buildFeatures {
    viewBinding = true
    compose = true
}

composeOptions {
    kotlinCompilerExtensionVersion = "1.5.1"
}

dependencies {
    implementation(platform("androidx.compose:compose-bom:2023.06.01"))

    implementation("androidx.activity:activity-compose:1.7.2")
    implementation("androidx.compose.material3:material3")
    implementation("com.google.accompanist:accompanist-themeadapter-material3:0.28.0")

    debugImplementation("androidx.compose.ui:ui-tooling")
}

❗ Удалить файл:
res/layout/list_item.xml

❗ Удалить в JuiceListAdapter:

ListItemBinding

LayoutInflater

всё, что связано с XML

## 2. Полный код JuiceListAdapter.kt

Вставьте весь этот файл. Он полностью готов, ничто не отсутствует.

package com.example.juicetracker.juicelist

import android.view.ViewGroup
import androidx.compose.foundation.Image
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.ComposeView
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.pluralStringResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Column
import androidx.compose.material3.Surface
import androidx.compose.ui.semantics.contentDescription
import androidx.compose.ui.semantics.semantics
import androidx.recyclerview.widget.RecyclerView
import com.example.juicetracker.R
import com.example.juicetracker.data.Juice
import com.example.juicetracker.data.JuiceColor
import com.google.accompanist.themeadapter.material3.Mdc3Theme


class JuiceListAdapter(
    private val onEdit: (Juice) -> Unit,
    private val onDelete: (Juice) -> Unit
) : RecyclerView.Adapter<JuiceListViewHolder>() {

    private var juiceList: List<Juice> = listOf()

    fun submitList(list: List<Juice>) {
        juiceList = list
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): JuiceListViewHolder {
        return JuiceListViewHolder(
            ComposeView(parent.context),
            onEdit,
            onDelete
        )
    }

    override fun getItemCount(): Int = juiceList.size

    override fun onBindViewHolder(holder: JuiceListViewHolder, position: Int) {
        holder.bind(juiceList[position])
    }
}



class JuiceListViewHolder(
    private val composeView: ComposeView,
    private val onEdit: (Juice) -> Unit,
    private val onDelete: (Juice) -> Unit
) : RecyclerView.ViewHolder(composeView) {

    fun bind(input: Juice) {
        composeView.setContent {
            Mdc3Theme {
                ListItem(
                    input = input,
                    onEdit = onEdit,
                    onDelete = onDelete,
                    modifier = Modifier
                        .fillMaxWidth()
                        .clickable { onEdit(input) }
                        .padding(vertical = 8.dp, horizontal = 16.dp)
                )
            }
        }
    }
}

## 3. Все composable-функции (в этом же файле)
ListItem()
@Composable
fun ListItem(
    input: Juice,
    onEdit: (Juice) -> Unit,
    onDelete: (Juice) -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier,
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.Top
    ) {
        JuiceIcon(color = input.color)
        JuiceDetails(
            juice = input,
            modifier = Modifier
                .weight(1f)
                .padding(start = 12.dp)
        )
        DeleteButton(
            onDelete = {
                onDelete(input)
            },
            modifier = Modifier.align(Alignment.Top)
        )
    }
}

JuiceIcon()
@Composable
fun JuiceIcon(color: String, modifier: Modifier = Modifier) {

    val colorLabelMap = JuiceColor.values().associateBy { stringResource(it.label) }
    val selectedColor = colorLabelMap[color]?.let { Color(it.color) }
    val juiceDesc = stringResource(R.string.juice_color, color)

    Box(
        modifier = modifier.semantics { contentDescription = juiceDesc }
    ) {
        Icon(
            painter = painterResource(R.drawable.ic_juice_color),
            contentDescription = null,
            tint = selectedColor ?: Color.Red,
            modifier = Modifier.align(Alignment.Center)
        )
        Icon(
            painter = painterResource(R.drawable.ic_juice_clear),
            contentDescription = null
        )
    }
}

JuiceDetails()
@Composable
fun JuiceDetails(juice: Juice, modifier: Modifier = Modifier) {
    Column(
        modifier = modifier,
        verticalArrangement = Arrangement.Top
    ) {
        Text(
            text = juice.name,
            style = MaterialTheme.typography.headlineSmall.copy(fontWeight = FontWeight.Bold),
        )
        Text(text = juice.description)
        RatingDisplay(
            rating = juice.rating,
            modifier = Modifier.padding(top = 8.dp)
        )
    }
}

RatingDisplay()
@Composable
fun RatingDisplay(rating: Int, modifier: Modifier = Modifier) {

    val desc = pluralStringResource(R.plurals.number_of_stars, rating, rating)

    Row(
        modifier = modifier.semantics {
            contentDescription = desc
        }
    ) {
        repeat(rating) {
            Image(
                painter = painterResource(R.drawable.star),
                contentDescription = null,
                modifier = Modifier.size(32.dp)
            )
        }
    }
}

DeleteButton()
@Composable
fun DeleteButton(onDelete: () -> Unit, modifier: Modifier = Modifier) {
    IconButton(
        onClick = onDelete,
        modifier = modifier
    ) {
        Icon(
            painter = painterResource(R.drawable.ic_delete),
            contentDescription = stringResource(R.string.delete)
        )
    }
}

## 4. Все Previews
@Preview
@Composable
fun PreviewJuiceIcon() {
    Mdc3Theme { JuiceIcon("Yellow") }
}

@Preview
@Composable
fun PreviewJuiceDetails() {
    Mdc3Theme {
        JuiceDetails(
            Juice(1, "Sweet Beet", "Apple, carrot, beet, lemon", "Red", 4)
        )
    }
}

@Preview
@Composable
fun PreviewDeleteIcon() {
    Mdc3Theme { DeleteButton({}) }
}

@Preview
@Composable
fun PreviewListItem() {
    Mdc3Theme {
        ListItem(
            Juice(1, "Sweet Beet", "Apple, carrot, beet, lemon", "Red", 4),
            onEdit = {},
            onDelete = {}
        )
    }
}