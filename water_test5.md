```cpp
#include <SimpleTimer.h>
#include <LiquidCrystal_I2C.h>

#define flowsensor 2

volatile double water = 0;
double last_water = 0;

LiquidCrystal_I2C lcd(0x27,16,2);

// ----------------------------------------
long previousTime = 0;  
unsigned long cnt=1;        
long interval = 5000; 
// ----------------------------------------

SimpleTimer t1;

void setup() {
  Serial.begin(9600);
  pinMode(flowsensor, INPUT);
  
  attachInterrupt(digitalPinToInterrupt(flowsensor), flow, FALLING);

  lcd.init();
  lcd.backlight();
 
  t1.setInterval(1000,fn1);
}

void loop() {
  t1.run();
}

void fn1(void) {
  Serial.print("last water : ");
  Serial.println(last_water);
  
  Serial.print("water: ");
  Serial.println(water);

  if(last_water!=water) {
    if(water>=50.0 && water<=100.0) {  
      Serial.println("경고1");
      lcd.setCursor(0, 0);
      lcd.print("warning 1");

      lcd.setCursor(0, 1);
      lcd.print(String(water));
    } else if(water>100.0 && water<=200.0) {
      Serial.println("경고2");
      lcd.setCursor(0, 0);
      lcd.print("warning 2");

      lcd.setCursor(0, 1);
      lcd.print(String(water));
    } else if(water>200.0) {
      Serial.println("경고3");
      lcd.setCursor(0, 0);
      lcd.print("warning 3");

      lcd.setCursor(0, 1);
      lcd.print(String(water));
    }

    previousTime = millis();  // 카운트 증가
    cnt++; 
    
  } else if(last_water==water) {
    Serial.println("이전 상태와 같음");
  }
  
  last_water = water;

  if (millis() - previousTime > interval) {  
     previousTime = millis();  
     cnt = 0;
     
     lcd.clear();
     lcd.setCursor(0, 0);
     lcd.print("safe");

     lcd.setCursor(0, 1);
     lcd.print("");
  }

  if(cnt<1) {
    water = 0;
  }
  Serial.print("cnt: ");
  Serial.println(cnt); 
}


void flow(){
  water += (1/5888.0)*10000;
}
 
```
