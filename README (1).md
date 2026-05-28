## Структура проєкту
Весь код програми знаходиться в одному файлі `MainActivity.kt`. Та написаний на Jetpack Compose.

## Детальний опис коду для пояснення

### Початок файлу: імпорт бібліотек

```kotlin
package com.example.lab2_krichun

import android.Manifest
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Bundle
import android.widget.Toast
// ... та інші імпорти
```
### Основний клас програми: MainActivity

```kotlin
class MainActivity : ComponentActivity() {
    private val CAMERA_PERMISSION_CODE = 100

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    SelfieScreen(
                        checkAndRequestCameraPermission = { checkAndRequestCameraPermission() }
                    )
                }
            }
        }
    }
    
    // Інші методи класу...
}
```

**Пояснення:**
- `MainActivity` - це головний екран нашого додатку
- `CAMERA_PERMISSION_CODE = 100` - це просто номер для ідентифікації запиту на доступ до камери
- `onCreate` - перший метод, який виконується при запуску додатку:
  - `setContent` встановлює, що буде відображатися на екрані
  - `MaterialTheme` задає сучасний стиль для всіх елементів
  - `Surface` створює основну поверхню для розміщення всіх елементів
  - `SelfieScreen` - це наш головний екран з кнопками і місцем для фото

### Метод для перевірки та запиту дозволу на камеру

```kotlin
private fun checkAndRequestCameraPermission(): Boolean {
    if (ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.CAMERA
        ) != PackageManager.PERMISSION_GRANTED
    ) {
        ActivityCompat.requestPermissions(
            this,
            arrayOf(Manifest.permission.CAMERA),
            CAMERA_PERMISSION_CODE
        )
        return false
    }
    return true
}
```

**Опис:**
- Цей метод перевіряє, чи дозволив користувач використовувати камеру:
  - Якщо НЕ дозволив (`!= PackageManager.PERMISSION_GRANTED`), то:
    - Запитуємо дозвіл (`requestPermissions`)
    - Повертаємо `false` (дозволу ще немає)
  - Якщо дозвіл вже є, повертаємо `true`

### Метод для обробки відповіді користувача на запит дозволу

```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    if (requestCode == CAMERA_PERMISSION_CODE) {
        if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            Toast.makeText(this, "Дозвіл на камеру надано", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "Потрібен дозвіл на камеру для зйомки селфі", Toast.LENGTH_LONG).show()
        }
    }
}
```

**Пояснення коду:**
- Цей метод викликається системою, коли користувач відповідає на запит дозволу камери
- Якщо користувач дозволив (`grantResults[0] == PackageManager.PERMISSION_GRANTED`), показуємо повідомлення "Дозвіл на камеру надано"
- Якщо не дозволив, показуємо повідомлення "Потрібен дозвіл на камеру для зйомки селфі"

### Функція SelfieScreen - наш користувацький інтерфейс

```kotlin
@Composable
fun SelfieScreen(checkAndRequestCameraPermission: () -> Boolean) {
    // Багато коду тут...
}
```

**Пояснення:**
- `@Composable` означає, що ця функція створює елементи інтерфейсу
- Функція створює весь екран з кнопками та місцем для фото
- `checkAndRequestCameraPermission` - це посилання на метод, який перевіряє дозвіл камери

### Створення місця для збереження фотографії

```kotlin
val context = LocalContext.current
var photoUri by remember { mutableStateOf<Uri?>(null) }
var permissionGranted by remember { mutableStateOf(false) }

val photoFile = remember {
    File.createTempFile(
        "JPEG_${SimpleDateFormat("yyyyMMdd_HHmmss", Locale.US).format(Date())}_",
        ".jpg",
        context.cacheDir
    ).apply {
        createNewFile()
        deleteOnExit()
    }
}

val imageUri = remember {
    FileProvider.getUriForFile(
        context,
        "${context.packageName}.fileprovider",
        photoFile
    )
}
```

**Опис:**
- `context` - це доступ до ресурсів системи Android
- `photoUri` - змінна, яка зберігатиме посилання на зроблене фото (спочатку пуста - `null`)
- `permissionGranted` - змінна, що зберігає стан дозволу на використання камери
- `photoFile` - створюємо тимчасовий файл для збереження фото:
  - Ім'я файлу містить поточну дату і час, щоб було унікальним
  - Файл буде видалено при закритті програми (`deleteOnExit()`)
- `imageUri` - спеціальне посилання на файл, яке можна передати камері

### Підготовка до запуску камери та відправки пошти

```kotlin
val cameraLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.TakePicture()
) { success ->
    if (success) {
        photoUri = imageUri
    }
}

val requestPermissionLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) {
        permissionGranted = true
        cameraLauncher.launch(imageUri)
    } else {
        Toast.makeText(
            context,
            "Потрібен дозвіл на камеру для зйомки селфі",
            Toast.LENGTH_LONG
        ).show()
    }
}

val emailLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.StartActivityForResult()
) { }
```

**Опис:**
- `cameraLauncher` - це інструмент для запуску камери:
  - Якщо фото успішно зроблено (`success`), зберігаємо його адресу (`photoUri = imageUri`)
- `requestPermissionLauncher` - інструмент для запиту дозволу на камеру:
  - Якщо дозвіл отримано (`isGranted`), запускаємо камеру
  - Якщо ні, показуємо повідомлення про необхідність дозволу
- `emailLauncher` - інструмент для запуску програми електронної пошти

### Створення інтерфейсу користувача

```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(16.dp),
    horizontalAlignment = Alignment.CenterHorizontally,
    verticalArrangement = Arrangement.Center
) {
    // Тут буде вміст екрану...
}
```

**Пояснення:**
- `Column` - це вертикальний контейнер, який розміщує елементи один під одним
- `fillMaxSize()` - займає весь доступний простір екрану
- `padding(16.dp)` - додає відступи 16 пікселів з усіх боків
- `horizontalAlignment = Alignment.CenterHorizontally` - вирівнює всі елементи по центру горизонтально
- `verticalArrangement = Arrangement.Center` - вирівнює всі елементи по центру вертикально

### Відображення фотографії або заповнювача

```kotlin
Box(
    modifier = Modifier
        .fillMaxWidth()
        .height(400.dp)
        .padding(bottom = 16.dp),
    contentAlignment = Alignment.Center
) {
    if (photoUri != null) {
        Image(
            painter = rememberAsyncImagePainter(photoUri),
            contentDescription = "Selfie Preview",
            modifier = Modifier.fillMaxSize()
        )
    } else {
        Box(
            modifier = Modifier.size(200.dp).background(Color.Gray)
        )
    }
}
```

**Опис блоку коду:**
- `Box` - це контейнер для розміщення елементів
- Якщо фото вже зроблено (`photoUri != null`):
  - Показуємо зображення з цього фото за допомогою `Image`
  - `rememberAsyncImagePainter` завантажує зображення у фоновому режимі
- Якщо фото ще не зроблено:
  - Показуємо сірий квадрат розміром 200 на 200 пікселів

### Кнопка "Зробити селфі"

```kotlin
Button(
    onClick = {
        if (ContextCompat.checkSelfPermission(
                context,
                Manifest.permission.CAMERA
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            cameraLauncher.launch(imageUri)
        } else {
            requestPermissionLauncher.launch(Manifest.permission.CAMERA)
        }
    },
    modifier = Modifier
        .fillMaxWidth()
        .padding(vertical = 8.dp)
) {
    Text("Зробити селфі")
}
```

**Пояснення:**
- `Button` - створює кнопку
- `onClick` - визначає, що станеться при натисканні:
  - Перевіряємо, чи є дозвіл на камеру
  - Якщо є, запускаємо камеру (`cameraLauncher.launch`)
  - Якщо немає, запитуємо дозвіл (`requestPermissionLauncher.launch`)
- `fillMaxWidth()` - кнопка займає всю ширину екрану
- `padding(vertical = 8.dp)` - додає відступи зверху і знизу
- `Text("Зробити селфі")` - текст на кнопці

### Кнопка "Відіслати селфі"

```kotlin
Button(
    onClick = {
        val emailIntent = Intent(Intent.ACTION_SEND).apply {
            type = "image/*"
            putExtra(Intent.EXTRA_EMAIL, arrayOf("hodovychenko@op.edu.ua"))
            putExtra(Intent.EXTRA_SUBJECT, "ANDROID Кричун Артем")
            putExtra(Intent.EXTRA_TEXT, "Ось моє селфі!\n\nРепозиторій проєкту: ")
            putExtra(Intent.EXTRA_STREAM, photoUri)
            flags = Intent.FLAG_GRANT_READ_URI_PERMISSION
        }
        emailLauncher.launch(Intent.createChooser(emailIntent, "Відправити селфі через..."))
    },
    enabled = photoUri != null,
    modifier = Modifier
        .fillMaxWidth()
        .padding(vertical = 8.dp)
) {
    Text("Відіслати селфі")
}
```

**Пояснення:**
- Ще одна кнопка для відправки фото електронною поштою
- `onClick` - при натисканні:
  - Створюємо `emailIntent` - інструкцію для відправки пошти
  - Вказуємо тип вмісту - зображення (`type = "image/*"`)
  - Додаємо адресу отримувача (`EXTRA_EMAIL`)
  - Додаємо тему листа (`EXTRA_SUBJECT`)
  - Додаємо текст листа (`EXTRA_TEXT`)
  - Додаємо наше фото як вкладення (`EXTRA_STREAM`)
  - Дозволяємо іншим програмам читати наш файл (`FLAG_GRANT_READ_URI_PERMISSION`)
  - Запускаємо вибір програми для відправки пошти
- `enabled = photoUri != null` - кнопка активна тільки якщо фото вже зроблено
- `Text("Відіслати селфі")` - текст на кнопці

## Додаткові файли, які потрібні для роботи додатку

### AndroidManifest.xml

```xml
<uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

<provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
```

**Простими словами:**
- Перші рядки повідомляють Android, що наш додаток потребує доступ до камери
- Блок `<provider>` дозволяє нашому додатку ділитися файлами (фотографіями) з іншими додатками
- `android:authorities="${applicationId}.fileprovider"` - це унікальне ім'я для нашого провайдера файлів
- `android:resource="@xml/file_paths"` - вказує на файл з налаштуваннями

### file_paths.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="external_files" path="." />
    <cache-path name="cache_files" path="." />
</paths>
```

**Пояснення:**
- Цей файл вказує, які каталоги нашого додатку можуть бути доступні іншим додаткам
- `<cache-path>` - дозволяє доступ до кеш-директорії, де ми зберігаємо фотографії
