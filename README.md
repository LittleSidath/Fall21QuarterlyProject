#Fall21QuaterlyProject

#include <Keypad.h>
#include <Servo.h>
#include <Wire.h>
#include "U8x8lib.h"
#include "Wire.h"

Servo myServo;
const int Hall_Sensor = 8;
const int buzzerPin=4;
const int SERVO_PIN = 10;
const int IRorMagneticSwitch = 5;
const int MAX_REFRESH = 1000;
const byte ROWS = 4;
const byte COLS = 3;
const int len_key = 5;
byte rowPins[ROWS] = {49, 48, 47, 46};
byte colPins[COLS] = {45, 44, 43};
char hexaKeys[ROWS][COLS] = {
{'1', '2', '3'},
{'4', '5', '6'},
{'7', '8', '9'},
{'*', '0', '#'}
};
char master_key[len_key] = {'1','2','3','4','*'};
char attempt_key[len_key];
String displayPass="";
unsigned long lastClear = 0;
int z=0;
Keypad customKeypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS);
U8X8_SSD1306_128X32_UNIVISION_HW_I2C oled(U8X8_PIN_NONE);

void setup(){
  Serial.begin(9600);
  pinMode (Hall_Sensor, INPUT);
  pinMode (IRorMagneticSwitch, INPUT);
  pinMode(buzzerPin, OUTPUT);
  Serial.begin(9600);
  setupDisplay();
  writeDisplay("Enter Password: ", 0,true);
}

void loop(){
   int HallStatus = digitalRead(Hall_Sensor);
   int IRorMagSwitchStatus = digitalRead(IRorMagneticSwitch);
   if(IRorMagSwitchStatus != 1){
      Serial.println("Magnetic Switch On");
      openCloseDoor();
   }

   if(HallStatus  != 1){
    Serial.println("Hall Sensor On");
    openCloseDoor();
   }

  char key = customKeypad.getKey();

  if (key){
    displayPass+="* ";
    writeDisplay(displayPass.c_str(), 1,false);
    switch(key){
      case '#':
        delay(100);
        checkKEY();
        break;
      default:
         attempt_key[z]=key;
         z++;
      }
  }

}

void setupDisplay() {   
    oled.begin();
    oled.setPowerSave(0);
    oled.setFont(u8x8_font_amstrad_cpc_extended_r);
    oled.setCursor(0, 0);
}
 
void writeDisplay(const char * message, int row, bool erase) {
    unsigned long now = millis();
    if(erase && (millis() - lastClear >= MAX_REFRESH)) {
        oled.clearDisplay();
        lastClear = now;
    }
    oled.setCursor(0, row);
    oled.print(message);
}

void writeDisplayCSV(String message, int commaCount) {
     int startIndex = 0;
     for(int i=0; i<=commaCount; i++) {
          int index = message.indexOf(',', startIndex);           
          String subMessage = message.substring(startIndex, index);
          startIndex = index + 1; 
          writeDisplay(subMessage.c_str(), i, false);
     }
}

void openCloseDoor(){
  myServo.attach(SERVO_PIN);
  Serial.println("Opening Exiting Door");
  for(int pos = 180; pos>=30; pos--){
    myServo.write(pos);
    delay(5);
  }
  delay(5000);
  Serial.println("Closing Exiting Door");
  for(int pos = 30; pos<=180; pos++){
   myServo.write(pos);
   delay(5);
   }
     
  myServo.detach(); 
}

void checkKEY()
{
   int correct=0;
   int i;
   for (i=0; i<len_key; i++) {
    if (attempt_key[i]==master_key[i]) {
      correct++;
      }
    }
   if (correct==len_key && z==len_key){
    displayPass="";
    writeDisplay("Correct Password", 2,false);
    delay(1000);
    openCloseDoor();
    z=0;
    writeDisplay("Enter Password: ", 0,true);
   }
   else
   {
    displayPass="";
    writeDisplay("Wrong Password", 2,false);
    tone(buzzerPin, 2000); 
    delay(1500); 
    noTone(buzzerPin);
    z=0;
    writeDisplay("Enter Password: ", 0,true);
   }
   for (int zz=0; zz<len_key; zz++) {
    attempt_key[zz]=0;
   }
}    
