```cpp
#include <SimpleTimer.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>

SoftwareSerial myserial(7, 8);

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
  Serial.begin(115200);
  myserial.begin(115200);

  delay(3000);
  myserial.println("AT");
  delay(1000);

  // SIM 카드 핀(전원 핀) 설정
  pinMode(9, OUTPUT);
  digitalWrite(9, HIGH);
  
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

      sendSMS("1025029532", 1);
    } else if(water>100.0 && water<=200.0) {
      Serial.println("경고2");
      lcd.setCursor(0, 0);
      lcd.print("warning 2");

      lcd.setCursor(0, 1);
      lcd.print(String(water));

      sendSMS("1025029532", 2);
    } else if(water>200.0) {
      Serial.println("경고3");
      lcd.setCursor(0, 0);
      lcd.print("warning 3");

      lcd.setCursor(0, 1);
      lcd.print(String(water));

      sendSMS("1025029532", 3);
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


void sendSMS(const char* phoneNumber, int a) {
  // SMS 메시지 보내기 AT 명령 전송
  myserial.print("AT+CMGF=1\r");
  delay(1000);
  
  myserial.print("AT+CMGS=\"");
  myserial.print(phoneNumber);
  myserial.print("\"\r");
  delay(1000);

  if(a==1) {
    myserial.write("007700610072006E0069006E00670031");   
  } else if(a==2) {
    myserial.write("007700610072006E0069006E00670032"); 
  } else if(a==3) {
    myserial.write("007700610072006E0069006E00670033"); 
  }
  // 007700610072006E0069006E00670032
  // B3C4C6C0C7740020D544C694D569B2C8B2E4002E
  delay(1000);
  
  myserial.write(26);
  delay(10000);
 
  // 응답 확인
  while (!myserial.available()) {
    // 응답을 기다리는 동안 대기
  }
 
  // 응답 출력
  while (myserial.available()) {
    Serial.write(myserial.read());
  }
}
 
```
