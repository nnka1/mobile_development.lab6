# Лабораторная работа №6. Уведомления. 

Выполнила: Усова Валентина, ИСП-221С
## Что делает приложение?

Приложение позволяет создавать и удалять напоминания с заданной датой и временем. При наступлении заданного времени, приложение отображает уведомление. Данные напоминаний хранятся в локальной базе данных.

## Работа приложения

(кликабельно)

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/4dXUo3m3PiA/default.jpg)](https://www.youtube.com/shorts/4dXUo3m3PiA)

## 1. Обзор MainActivity
![](https://github.com/nnka1/mobile_development.lab6/blob/main/photo_2024-12-04_17-32-08.jpg)

![](https://github.com/nnka1/mobile_development.lab6/blob/main/photo_2024-12-04_17-33-51.jpg)

Основные функции:

* Взаимодействие с пользователем: Обрабатывает ввод заголовка и текста напоминания в полях titleEditText и messageEditText. Обрабатывает нажатия кнопок "Добавить напоминание" (addButton), "Выбрать дату" (dateButton) и "Выбрать время" (timeButton). Также обрабатывает нажатия на элементы списка напоминаний для их удаления. Пример обработчика кнопки "Добавить напоминание":
```
addButton.setOnClickListener(v -> addReminder());

```
* Управление напоминаниями: Использует DatabaseHelper для добавления, получения и удаления напоминаний из локальной базы данных.

* Планирование уведомлений: После добавления напоминания вызывает метод scheduleNotification() для планирования уведомления с помощью AlarmManager. 
```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    if (alarmManager.canScheduleExactAlarms()) {
        alarmManager.setExact(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
    } else {
        startActivity(new Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM, Uri.parse("package:" + getPackageName())));
    }
} else {
    alarmManager.setExact(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pendingIntent);
}

```
* Обновление интерфейса: После добавления или удаления напоминаний обновляет список напоминаний в ListView с помощью ReminderAdapter.

* Обработка даты и времени: Использует DatePickerDialog и TimePickerDialog для удобного выбора даты и времени напоминания.

## 2. Элементы UI

Пользовательский интерфейс MainActivity состоит из:

• EditText titleEditText: Поле для ввода заголовка напоминания.

• EditText messageEditText: Поле для ввода текста напоминания.

• Button dateButton: Кнопка для выбора даты напоминания.

• Button timeButton: Кнопка для выбора времени напоминания.

• Button addButton: Кнопка для добавления нового напоминания.

• ListView reminderList: Список, отображающий все созданные напоминания. Каждый элемент списка содержит заголовок, текст и кнопку удаления.


## 3. Добавление напоминания (addReminder() метод)

Метод addReminder() выполняет следующие шаги:

1. Получение данных из полей ввода:
```
String title = titleEditText.getText().toString();
String message = messageEditText.getText().toString();

```
2. Проверка на пустые поля:
```
if (title.isEmpty() || message.isEmpty()) {
    Toast.makeText(this, "Заполните все поля", Toast.LENGTH_SHORT).show();
    return;
}

```
3. Форматирование даты и времени:
```
String date = String.format(Locale.getDefault(), "%04d-%02d-%02d %02d:%02d",
        selectedCalendar.get(Calendar.YEAR),
        selectedCalendar.get(Calendar.MONTH) + 1,
        selectedCalendar.get(Calendar.DAY_OF_MONTH),
        selectedCalendar.get(Calendar.HOUR_OF_DAY),
        selectedCalendar.get(Calendar.MINUTE));

```
4. Добавление в базу данных:
```
 if (dbHelper.addReminder(title, message, date)) {
            scheduleNotification(title, message, date);
            loadReminders();
            Toast.makeText(this, "Напоминание добавлено", Toast.LENGTH_SHORT).show();
            titleEditText.setText("");
            messageEditText.setText("");
        } else {
            Toast.makeText(this, "Ошибка при добавлении напоминания", Toast.LENGTH_SHORT).show();
        }

```
5. Планирование уведомления: Вызов scheduleNotification(title, message, date);

6. Обновление списка: Вызов loadReminders();

7. Очистка полей:
```
titleEditText.setText("");
messageEditText.setText("");

```
8. Отображение сообщения: Отображение Toast-сообщения об успехе или неудаче.

## 4. Удаление напоминания (deleteReminder() метод)

Метод deleteReminder() удаляет напоминание из базы данных по его ID:
```
public boolean deleteReminder(int id) {
    SQLiteDatabase db = this.getWritableDatabase();
    return db.delete(TABLE_NAME, COL_1 + "=?", new String[]{String.valueOf(id)}) > 0;
}

```
## 5. DatabaseHelper

Пример добавления напоминания в базу данных:
```
public boolean addReminder(String title, String message, String date) {
    SQLiteDatabase db = this.getWritableDatabase();
    ContentValues contentValues = new ContentValues();
    contentValues.put(COL_2, title);
    contentValues.put(COL_3, message);
    contentValues.put(COL_4, date);
    long result = db.insert(TABLE_NAME, null, contentValues);
    return result != -1;
}

```
## 6. NotificationReceiver 

Пример создания уведомления:
```
NotificationCompat.Builder builder = new NotificationCompat.Builder(context, channelId)
    .setSmallIcon(R.drawable.cat)
    .setContentTitle(title)
    .setContentText(message)
    .setPriority(NotificationCompat.PRIORITY_HIGH)
    .setContentIntent(contentIntent)
    .setAutoCancel(true);
notificationManager.notify((int) System.currentTimeMillis(), builder.build());

```
## 7. Требуемые разрешения

Приложение запрашивает следующие разрешения (в манифесте):
```
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />

```

## Как собрать проект?
1. Загрузка или клонирование репозитория:
* Скачайте файлы проекта (ZIP-архив) и разархивируйте их в удобную папку или клонируйте репозиторий с помощью команды git clone [URL репозитория].

2. Открытие проекта в Android Studio:
* Откройте Android Studio.
* В главном окне выберите "Open an existing Android Studio project".
* Найдите папку, в которую вы скачали или клонировали проект, и выберите файл build.gradle.

3. Запуск приложения:
* В Android Studio, нажмите кнопку "Run" (зеленый треугольник) на панели инструментов.
* Выберите эмулятор или подключенное Android-устройство, на котором вы хотите запустить приложение.
* Дождитесь завершения сборки и запуска приложения.
