## Разделим UI на 2 основные секции:
## Верхняя секция:
## Логотип
## Имя
## Должность
## Нижняя секция:
## Телефон
## Email
## Линк (или соцсеть)
@Composable
fun BusinessCardApp() {
    Surface(
        color = Color(0xFF073042),
        modifier = Modifier.fillMaxSize()
    ) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(32.dp),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            ProfileSection()
            Spacer(modifier = Modifier.height(32.dp))
            ContactSection()
        }
    }
}

@Composable
fun ProfileSection() {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally
    ) {

        Image(
            painter = painterResource(id = R.drawable.android_logo),
            contentDescription = "Android Logo",
            modifier = Modifier
                .size(120.dp)
                .clip(CircleShape)
                .border(2.dp, Color.White, CircleShape)
        )
        Text(
            text = "Ржевский Кирилл",
            fontSize = 32.sp,
            fontWeight = FontWeight.Bold,
            color = Color.White,
            modifier = Modifier.padding(top = 16.dp)
        )
        Text(
            text = "Android Developer",
            fontSize = 20.sp,
            color = Color(0xFF3ddc84)
        )
    }
}

@Composable
fun ContactSection() {
    Column(
        modifier = Modifier.fillMaxWidth()
    ) {
        Divider(color = Color.Gray, thickness = 1.dp)
        ContactRow(icon = Icons.Default.Phone, contactText = "+7 (999) 123-45-67")
        ContactRow(icon = Icons.Default.Email, contactText = "Kirill@example.com")
        ContactRow(icon = Icons.Default.Share, contactText = "@Kirilldev")
    }
}

@Composable
fun ContactRow(icon: ImageVector, contactText: String) {
    Row(
        verticalAlignment = Alignment.CenterVertically,
        modifier = Modifier
            .padding(vertical = 8.dp)
    ) {
        Icon(
            imageVector = icon,
            contentDescription = null,
            tint = Color(0xFF3ddc84),
            modifier = Modifier.padding(end = 16.dp)
        )
        Text(
            text = contactText,
            color = Color.White,
            fontSize = 16.sp
        )
    }
}
