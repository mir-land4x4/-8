Код:
#include <Wire.h>
#include <Adafruit_BMP085.h>
#include <LiquidCrystal_I2C.h>

Adafruit_BMP085 bmp;
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Энкодер
#define CLK 2
#define DT 3

int counter = 0;
int currentStateCLK;
int lastStateCLK;

float zeroAltitude = 0;

void setup() {

  pinMode(CLK, INPUT);
  pinMode(DT, INPUT);

  lcd.init();
  lcd.backlight();

  // Проверка BMP
  if (!bmp.begin()) {
    lcd.print("BMP ERROR");
    while (1);
  }

  // Начальная высота
  zeroAltitude = bmp.readAltitude();

  lastStateCLK = digitalRead(CLK);

  lcd.clear();
  lcd.print("Altimeter");
  delay(1000);
}

void loop() {

  currentStateCLK = digitalRead(CLK);

  // Работа энкодера
  if (currentStateCLK != lastStateCLK &&
      currentStateCLK == HIGH) {

    if (digitalRead(DT) != currentStateCLK) {
      counter++;
    } else {
      counter--;
    }
  }

  lastStateCLK = currentStateCLK;

  // Абсолютная высота
  float altitude = bmp.readAltitude();

  // Относительная высота
  float relativeAlt = altitude - zeroAltitude;

  // Поправка энкодером
  float finalAlt = relativeAlt + counter;

  // Давление в Па
  long pressurePa = bmp.readPressure();

  // Перевод в мм рт. ст.
  float pressureMM = pressurePa / 133.3;

  // ===== LCD =====
  lcd.clear();

  // 1 строка — высота
  lcd.setCursor(0, 0);
  lcd.print("H:");
  lcd.print(finalAlt, 1);
  lcd.print("m");

  // 2 строка — давление
  lcd.setCursor(0, 1);
  lcd.print("P:");
  lcd.print(pressureMM, 0);
  lcd.print(" ENC:");
  lcd.print(counter);

  delay(100);
}
