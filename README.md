# arduino1
/*
 * Arduino Stopwatch System
 * Measures elapsed time in tenths of a second.
 * Button A (pin 2): Pause / Resume
 * Button B (pin 3): Reset
 *
 * Debounce logic on both buttons prevents false triggers
 * from mechanical switch bounce.
 */

#include <LiquidCrystal.h>

// ── LCD Pins ──────────────────────────────────────────────
const int LCD_PinRS = 8;
const int LCD_PinE  = 9;
const int LCD_PinD4 = 10;
const int LCD_PinD5 = 11;
const int LCD_PinD6 = 12;
const int LCD_PinD7 = 13;

// ── Button Pins ───────────────────────────────────────────
const int PIN_PAUSE_RESUME = 2;
const int PIN_RESET        = 3;

// ── Debounce Settings ─────────────────────────────────────
const unsigned long DEBOUNCE_DELAY = 50; // ms

// ── Stopwatch State ───────────────────────────────────────
enum State { RUNNING, PAUSED };
State timerState = RUNNING;

unsigned long startTime   = 0;  // millis() when timer last started/resumed
unsigned long elapsedTime = 0;  // accumulated ms before last pause

// ── Button State ──────────────────────────────────────────
int lastPauseReading  = HIGH;
int lastResetReading  = HIGH;
unsigned long lastPauseDebounce = 0;
unsigned long lastResetDebounce = 0;
int pauseButtonState  = HIGH;
int resetButtonState  = HIGH;

LiquidCrystal lcd(LCD_PinRS, LCD_PinE, LCD_PinD4, LCD_PinD5, LCD_PinD6, LCD_PinD7);

void setup() {
  pinMode(PIN_PAUSE_RESUME, INPUT_PULLUP);
  pinMode(PIN_RESET,        INPUT_PULLUP);

  lcd.begin(16, 2);
  lcd.clear();
  lcd.setCursor(3, 0);
  lcd.print("STOPWATCH");
  lcd.setCursor(0, 1);
  lcd.print("Time:   0.0  sec");

  startTime = millis();
}

void loop() {
  handlePauseButton();
  handleResetButton();
  updateDisplay();
  delay(50);
}

void handlePauseButton() {
  int reading = digitalRead(PIN_PAUSE_RESUME);

  if (reading != lastPauseReading) {
    lastPauseDebounce = millis();
  }

  if ((millis() - lastPauseDebounce) > DEBOUNCE_DELAY) {
    if (reading != pauseButtonState) {
      pauseButtonState = reading;

      if (pauseButtonState == LOW) {
        if (timerState == RUNNING) {
          elapsedTime += millis() - startTime;
          timerState = PAUSED;
        } else {
          startTime  = millis();
          timerState = RUNNING;
        }
      }
    }
  }

  lastPauseReading = reading;
}

void handleResetButton() {
  int reading = digitalRead(PIN_RESET);

  if (reading != lastResetReading) {
    lastResetDebounce = millis();
  }

  if ((millis() - lastResetDebounce) > DEBOUNCE_DELAY) {
    if (reading != resetButtonState) {
      resetButtonState = reading;

      if (resetButtonState == LOW) {
        elapsedTime = 0;
        startTime   = millis();
        timerState  = RUNNING;
      }
    }
  }

  lastResetReading = reading;
}

void updateDisplay() {
  unsigned long total;

  if (timerState == RUNNING) {
    total = elapsedTime + (millis() - startTime);
  } else {
    total = elapsedTime;
  }

  unsigned long seconds = total / 1000;
  unsigned long tenths  = (total / 100) % 10;

  lcd.setCursor(0, 0);
  if (timerState == PAUSED) {
    lcd.print("   ** PAUSED **  ");
  } else {
    lcd.print("   STOPWATCH     ");
  }

  lcd.setCursor(6, 1);
  char buf[8];
  sprintf(buf, "%4lu.%lu", seconds, tenths);
  lcd.print(buf);
}
