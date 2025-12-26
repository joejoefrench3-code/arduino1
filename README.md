# arduino1
This board runs a stopwatch and measures time (in tenths of a second) from when a button is clicked. The timer can be reset or paused at will.  I had helped assemble several wires onto the board in the appropriate manner.
#include <LiquidCrystal.h>

// LCD Pin Definitions
int LCD_PinRS = 8;   // Register Select
int LCD_PinE  = 9;   // Enable
int LCD_PinD4 = 10;  // Data 4
int LCD_PinD5 = 11;  // Data 5
int LCD_PinD6 = 12;  // Data 6
int LCD_PinD7 = 13;  // Data 7

// Time Variables
int seconds = 0;
int tenths = 0;

// Initialize LCD
LiquidCrystal lcd(LCD_PinRS, LCD_PinE, LCD_PinD4, LCD_PinD5, LCD_PinD6, LCD_PinD7);

void setup() {
  lcd.begin(16, 2);
  lcd.clear();
  lcd.setCursor(5, 0);
  lcd.print("ARDUINO");
  lcd.setCursor(0, 1);
  lcd.print("Time: ");
  lcd.setCursor(13, 1);
  lcd.print("sec");
}

void loop() {
  seconds = millis() / 1000;           // Total seconds
  tenths = (millis() / 100) - (seconds * 10); // Tenths of a second

  lcd.setCursor(6, 1);
  lcd.print(seconds);
  lcd.print(".");
  lcd.print(tenths);

  delay(100); // Update every 0.1 second
}
