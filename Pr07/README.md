## 
package com.example.lemonade

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.Image
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.lemonade.ui.theme.LemonadeTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            LemonadeTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    LemonApp()
                }
            }
        }
    }
}

@Composable
fun LemonApp() {
    var currentStep by remember { mutableStateOf(1) }
    var squeezeCount by remember { mutableStateOf(0) }

    when (currentStep) {
        1 -> {
            LemonTextAndImage(
                textResId = R.string.lemon_select,
                drawableResId = R.drawable.lemon_tree,
                contentDescriptionResId = R.string.lemon_tree_content_description,
                onImageClick = {
                    squeezeCount = (2..4).random()
                    currentStep = 2
                }
            )
        }
        2 -> {
            LemonTextAndImage(
                textResId = R.string.lemon_squeeze,
                drawableResId = R.drawable.lemon_squeeze,
                contentDescriptionResId = R.string.lemon_content_description,
                onImageClick = {
                    squeezeCount--
                    if (squeezeCount <= 0) {
                        currentStep = 3
                    }
                }
            )
        }
        3 -> {
            LemonTextAndImage(
                textResId = R.string.lemon_drink,
                drawableResId = R.drawable.lemon_drink,
                contentDescriptionResId = R.string.lemon_drink_content_description,
                onImageClick = {
                    currentStep = 4
                }
            )
        }
        4 -> {
            LemonTextAndImage(
                textResId = R.string.lemon_restart,
                drawableResId = R.drawable.lemon_restart,
                contentDescriptionResId = R.string.lemon_restart_content_description,
                onImageClick = {
                    currentStep = 1
                }
            )
        }
    }
}

@Composable
fun LemonTextAndImage(
    textResId: Int,
    drawableResId: Int,
    contentDescriptionResId: Int,
    onImageClick: () -> Unit
) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
    ) {
        Text(
            text = stringResource(id = textResId),
            fontSize = 18.sp
        )
        Spacer(modifier = Modifier.height(16.dp))
        Image(
            painter = painterResource(id = drawableResId),
            contentDescription = stringResource(id = contentDescriptionResId),
            modifier = Modifier
                .wrapContentSize()
                .clickable { onImageClick() }
        )
    }
}

@Composable
fun DefaultPreview() {
    LemonadeTheme {
        LemonApp()
    }
}
