## Краткое резюме ключевых этапов:
1. Удаление старых ресурсов

Удалили старые файлы ic_launcher_background.xml и ic_launcher_foreground.xml.

Очистили папки mipmap-* и mipmap-anydpi-v26.

2. Добавление новых иконок

Загрузили новые XML-файлы для переднего и фонового слоев.

Убедились, что передний слой содержит ключевые элементы (например, логотип), а задний — фон.

Использовали Image Asset Studio для генерации всех необходимых ресурсов.

3. Создание адаптивной иконки

В Foreground Layer загрузили ic_launcher_foreground.xml.

В Background Layer — ic_launcher_background.xml.

Убедились, что иконка выглядит корректно на превью (с разными формами масок).

4. Поддержка разных версий Android

Адаптивная иконка поддерживается с API 26+.

Для более старых версий Android автоматически сгенерированы растровые изображения (.webp, .png) в соответствующих папках mipmap-*.

5. Тестирование

Запустили приложение на эмуляторе или устройстве.

Убедились, что новая иконка отображается корректно в списке приложений и на главном экране.

## AndroidManifest.xml
<activity
    android:name=".MainActivity"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher" />

<activity-alias
    android:name=".Icon1"
    android:enabled="true"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_icon1"
    android:targetActivity=".MainActivity" />

<activity-alias
    android:name=".Icon2"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_icon2"
    android:targetActivity=".MainActivity" />

## Kotlin-код для переключения иконки
fun switchAppIcon(useIcon1: Boolean, context: Context) {
    val packageManager = context.packageManager

    val icon1 = ComponentName(context, "com.yourpackage.Icon1")
    val icon2 = ComponentName(context, "com.yourpackage.Icon2")

    if (useIcon1) {
        packageManager.setComponentEnabledSetting(
            icon1,
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
            PackageManager.DONT_KILL_APP
        )
        packageManager.setComponentEnabledSetting(
            icon2,
            PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
            PackageManager.DONT_KILL_APP
        )
    } else {
        packageManager.setComponentEnabledSetting(
            icon1,
            PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
            PackageManager.DONT_KILL_APP
        )
        packageManager.setComponentEnabledSetting(
            icon2,
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
            PackageManager.DONT_KILL_APP
        )
    }
}
