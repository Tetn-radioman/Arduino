## 📌 ШПАРГАЛКА: Qt + Arduino (мигание светодиодами)

### 🔷 ARDUINO (скетч)

```cpp
// ============================================
// СОСТОЯНИЕ ПО УМОЛЧАНИЮ:
//   A1 = ГОРИТ
//   A2 = НЕ ГОРИТ
//   A3 = НЕ ГОРИТ
//
// ПО КОМАНДЕ '1' (2 секунды):
//   A1 = НЕ ГОРИТ
//   A2 = ГОРИТ
//   A3 = ГОРИТ
// ============================================

const int led1 = A1;  // Всегда горит
const int led2 = A2;  // Включается по команде
const int led3 = A3;  // Включается по команде

void setup() {
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  
  // НАЧАЛЬНОЕ СОСТОЯНИЕ
  digitalWrite(led1, HIGH);   // A1 горит
  digitalWrite(led2, LOW);    // A2 не горит
  digitalWrite(led3, LOW);    // A3 не горит
  
  Serial.begin(9600);
  Serial.println("Ready");
}

void loop() {
  if (Serial.available() > 0) {
    char command = Serial.read();
    
    if (command == '1') {
      // МЕНЯЕМ СОСТОЯНИЕ НА 2 СЕКУНДЫ
      digitalWrite(led1, LOW);   // A1 гаснет
      digitalWrite(led2, HIGH);  // A2 загорается
      digitalWrite(led3, HIGH);  // A3 загорается
      
      delay(2000);  // Ждём 2 секунды
      
      // ВОЗВРАЩАЕМ НАЧАЛЬНОЕ СОСТОЯНИЕ
      digitalWrite(led1, HIGH);  // A1 горит
      digitalWrite(led2, LOW);   // A2 гаснет
      digitalWrite(led3, LOW);   // A3 гаснет
    }
  }
}
```

---

### 🔷 Qt (консольное приложение)

```cpp
// ============================================
// Qt КОНСОЛЬНОЕ ПРИЛОЖЕНИЕ
// Отправляет команду '1' в Arduino
// ============================================

#include <QCoreApplication>
#include <QSerialPort>
#include <QDebug>
#include <QThread>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    
    // 1. СОЗДАЁМ ОБЪЕКТ ПОРТА
    QSerialPort serial;
    
    // 2. НАСТРАИВАЕМ ПОРТ
    serial.setPortName("/dev/ttyUSB0");   // Имя порта (Linux)
    serial.setBaudRate(QSerialPort::Baud9600);
    
    // 3. ОТКРЫВАЕМ ПОРТ
    if (!serial.open(QIODevice::ReadWrite)) {
        qDebug() << "Ошибка:" << serial.errorString();
        return 1;
    }
    
    qDebug() << "Порт открыт";
    
    // 4. ВАЖНО! ЖДЁМ ПОКА ARDUINO ЗАГРУЗИТСЯ
    QThread::sleep(2);  // БЕЗ ЭТОГО НЕ РАБОТАЕТ!
    
    // 5. ОТПРАВЛЯЕМ КОМАНДУ
    serial.write("1");
    serial.flush();  // Принудительная отправка
    qDebug() << "Команда '1' отправлена";
    
    // 6. ЖДЁМ ВЫПОЛНЕНИЯ (3 секунды)
    QThread::sleep(3);
    
    // 7. ЗАКРЫВАЕМ ПОРТ
    serial.close();
    qDebug() << "Готово";
    
    return 0;
}
```

---

### 🔷 cmake файл для Qt

```cmake
cmake_minimum_required(VERSION 3.19)
project(SCSArduino LANGUAGES CXX)

find_package(Qt6 6.9 REQUIRED COMPONENTS Core SerialPort)

qt_standard_project_setup()

qt_add_executable(SCSArduino
    main.cpp
)

target_link_libraries(SCSArduino
    PRIVATE
        Qt6::Core
        Qt6::SerialPort
)

install(TARGETS SCSArduino
    BUNDLE  DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
```

---

### 🔷 Схема подключения

```
Arduino            Светодиоды
═══════════════════════════════════════════
  A1 ───[220 Ом]─── LED1 ─── GND
  A2 ───[220 Ом]─── LED2 ─── GND
  A3 ───[220 Ом]─── LED3 ─── GND
```

---

### 🔷 Важные моменты

| Что | Почему |
|-----|--------|
| **`QThread::sleep(2);`** | Arduino нужно время на инициализацию после открытия порта |
| **`serial.flush();`** | Принудительно отправляет данные (без этого может не уйти) |
| **`Baud9600`** | Скорость должна совпадать с Arduino (`Serial.begin(9600)`) |
| **`/dev/ttyUSB0`** | Имя порта может быть `/dev/ttyACM0` или другим |
| **`delay(2000);`** | Блокирующая задержка на Arduino (просто и надёжно) |

---

### 🔷 Полезные команды для Linux

```bash
# Узнать имя порта
ls /dev/ttyUSB* /dev/ttyACM*

# Дать права на порт
sudo chmod 666 /dev/ttyUSB0

# Посмотреть, кто занял порт
lsof /dev/ttyUSB0

# Убить процесс, занявший порт
sudo kill -9 [PID]
```

---

### 🔷 Если не работает

1. **Arduino IDE закрыта?** (она блокирует порт)
2. **Порт правильный?** (проверьте `ls /dev/tty*`)
3. **Скорость совпадает?** (9600)
4. **Ждёте 2 секунды?** (`QThread::sleep(2);`)
5. **Земля общая?** (GND подключен)

---

### 🔷 Готово! 🎉
