#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <virtuabotixRTC.h>
#include <Keypad.h>

#define I2C_ADDR 0x27 
#define BACKLIGHT_PIN 3
#define En_pin 2
#define Rw_pin 1
#define Rs_pin 0
#define D4_pin 4
#define D5_pin 5
#define D6_pin 6
#define D7_pin 7

LiquidCrystal_I2C lcd(I2C_ADDR, En_pin, Rw_pin, Rs_pin, D4_pin, D5_pin, D6_pin, D7_pin);
virtuabotixRTC myRTC(2, 3, 4); 

const byte numRows = 4; 
const byte numCols = 4;

char keymap[numRows][numCols] = {
  {'1', '2', '3', 'A'}, 
  {'4', '5', '6', 'B'}, 
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

byte rowPins[numRows] = {12, 11, 10, 9}; 
byte colPins[numCols] = {8, 7, 6, 5}; 

Keypad myKeypad = Keypad(makeKeymap(keymap), rowPins, colPins, numRows, numCols);

unsigned long startTime = 0;
unsigned long elapsedTime = 0;
bool timerRunning = false;

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2);
  lcd.setBacklightPin(BACKLIGHT_PIN, POSITIVE);
  lcd.setBacklight(HIGH);
  lcd.home();
}

void loop() {
  char key = myKeypad.getKey();
  
  if (key != NO_KEY) {
    handleKeypadInput(key);
  }
  
  displayClock();
  displayTimer();
}

void handleKeypadInput(char key) {
  if (key == 'A') { // Start/Stop timer with 'A' key
    if (timerRunning) {
      timerRunning = false;
    } else {
      startTime = millis() - elapsedTime;
      timerRunning = true;
    }
  }
}

void displayClock() {
  myRTC.updateTime();
  lcd.setCursor(0, 0);
  lcd.print("Time: ");
  lcd.print(myRTC.hours);
  lcd.print(":");
  lcd.print(myRTC.minutes);
  lcd.print(":");
  lcd.print(myRTC.seconds);
}

void displayTimer() {
  if (timerRunning) {
    elapsedTime = millis() - startTime;
    int seconds = elapsedTime / 1000;
    int minutes = seconds / 60;
    int hours = minutes / 60;
    
    lcd.setCursor(0, 1);
    lcd.print("Timer: ");
    lcd.print(hours);
    lcd.print(":");
    lcd.print(minutes % 60);
    lcd.print(":");
    lcd.print(seconds % 60);
  }
}
