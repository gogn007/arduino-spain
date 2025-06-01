#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <SD.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

int totalWords = 0, currentLine = 0;
bool trans = false, mode = true;
String currentSpanish = "", currentCzech = "";

void setup() {
  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);
  
  lcd.init();
  lcd.backlight();
  lcd.print("Pocitam slova...");
  
  if (!SD.begin(10)) {
    lcd.clear();
    lcd.print("CHYBA SD!");
    while(1);
  }
  
  // Spočítej kolik je slov v souboru
  File f = SD.open("slovnik.txt");
  if (!f) {
    lcd.clear();
    lcd.print("slovnik.txt?");
    while(1);
  }
  
  while (f.available()) {
    String line = f.readStringUntil('\n');
    line.trim();
    if (line.length() > 0 && !line.startsWith("#") && line.indexOf('|') > 0) {
      totalWords++;
    }
  }
  f.close();
  
  if (totalWords == 0) {
    lcd.clear();
    lcd.print("Zadna slova!");
    while(1);
  }
  
  lcd.clear();
  lcd.print("Nalezeno:");
  lcd.setCursor(0, 1);
  lcd.print(String(totalWords) + " slov");
  delay(1500);
  
  randomSeed(analogRead(0));
  currentLine = random(totalWords);
  loadWord(currentLine);
  show();
}

void loop() {
  static unsigned long t1 = 0, t2 = 0;
  
  if (!digitalRead(2) && millis() - t1 > 300) {
    t1 = millis();
    if (!trans) {
      trans = true;
      show();
    } else {
      currentLine = random(totalWords);
      loadWord(currentLine);
      trans = false;
      show();
    }
  }
  
  if (!digitalRead(3) && millis() - t2 > 300) {
    t2 = millis();
    mode = !mode;
    trans = false;
    lcd.clear();
    lcd.print(mode ? "ES->CZ" : "CZ->ES");
    delay(500);
    show();
  }
}

void loadWord(int lineNumber) {
  File f = SD.open("slovnik.txt");
  if (!f) return;
  
  int currentWordLine = 0;
  while (f.available()) {
    String line = f.readStringUntil('\n');
    line.trim();
    
    if (line.length() > 0 && !line.startsWith("#") && line.indexOf('|') > 0) {
      if (currentWordLine == lineNumber) {
        int pos = line.indexOf('|');
        currentSpanish = line.substring(0, pos);
        currentCzech = line.substring(pos + 1);
        
        // Zkrať pokud je moc dlouhé
        if (currentSpanish.length() > 16) currentSpanish = currentSpanish.substring(0, 16);
        if (currentCzech.length() > 16) currentCzech = currentCzech.substring(0, 16);
        break;
      }
      currentWordLine++;
    }
  }
  f.close();
}

void show() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(trans ? (mode ? "CESKY:" : "SPANELSKY:") : (mode ? "SPANELSKY:" : "CESKY:"));
  lcd.setCursor(0, 1);
  lcd.print(trans ? (mode ? currentCzech : currentSpanish) : (mode ? currentSpanish : currentCzech));
}
