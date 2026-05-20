#include <Wire.h>
#include <Adafruit_BMP085.h>
#include <LiquidCrystal_I2C.h>

Adafruit_BMP085 bmp;
LiquidCrystal_I2C lcd(0x27, 16, 2);

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

  if (!bmp.begin()) {
    lcd.print("BMP ERROR");
    while (1);
  }

  zeroAltitude = bmp.readAltitude();

  lastStateCLK = digitalRead(CLK);

  lcd.clear();
  lcd.print("Altimeter");
  delay(1000);
}

void loop() {

  currentStateCLK = digitalRead(CLK);

  if (currentStateCLK != lastStateCLK &&
      currentStateCLK == HIGH) {

  if (digitalRead(DT) != currentStateCLK) {
      counter++;
    } else {
      counter--;
    }
  }

  lastStateCLK = currentStateCLK;

  float altitude = bmp.readAltitude();

  float relativeAlt = altitude - zeroAltitude;

  float finalAlt = relativeAlt + counter;

  long pressurePa = bmp.readPressure();

  float pressureMM = pressurePa / 133.3;

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("H:");
  lcd.print(finalAlt, 1);
  lcd.print("m");
  
  lcd.setCursor(0, 1);
  lcd.print("P:");
  lcd.print(pressureMM, 0);
  lcd.print(" ENC:");
  lcd.print(counter);

  delay(100);
}
