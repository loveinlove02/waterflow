```c++
#include <SimpleTimer.h>
#include <LiquidCrystal_I2C.h>

const int flowSensorPin = 2;  // 유량 센서의 신호 핀
volatile double waterFlow;  // 유량 변수 (리터 단위)
unsigned long currentTime;
unsigned long prevTime = 0;
unsigned long elapsedTime;

LiquidCrystal_I2C lcd(0x27, 16, 2);  // LCD 화면 설정

SimpleTimer t1;

void setup() {
  pinMode(flowSensorPin, INPUT);
  digitalWrite(flowSensorPin, HIGH);  // 내부 풀업 활성화

  attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulse, RISING);
  
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
 
  t1.setInterval(1000, checkFlowRate);  // 1초마다 checkFlowRate 함수 실행
}

void loop() {
  t1.run();
}

void checkFlowRate() {
  currentTime = millis();
  elapsedTime = currentTime - prevTime;

  if (elapsedTime > 1000) {  // 1초마다 계산
    detachInterrupt(digitalPinToInterrupt(flowSensorPin));
    
    double flowRateMlPerHour = waterFlow * 60 * 1000;  // 초당 유량(L) -> 시간당 유량(ml)

    Serial.print("유량: ");
    Serial.print(flowRateMlPerHour);
    Serial.println(" ml/hour");

    if (flowRateMlPerHour >= 10.0 && flowRateMlPerHour < 20.0) {  
      Serial.println("경고1");
      lcd.setCursor(0, 0);
      lcd.print("warning 1");

      lcd.setCursor(0, 1);
      lcd.print(String(flowRateMlPerHour) + " ml/h");
    } else if (flowRateMlPerHour >= 20.0 && flowRateMlPerHour < 30.0) {
      Serial.println("경고2");
      lcd.setCursor(0, 0);
      lcd.print("warning 2");

      lcd.setCursor(0, 1);
      lcd.print(String(flowRateMlPerHour) + " ml/h");
    } else if (flowRateMlPerHour >= 30.0 && flowRateMlPerHour < 40.0) {
      Serial.println("경고3");
      lcd.setCursor(0, 0);
      lcd.print("warning 3");

      lcd.setCursor(0, 1);
      lcd.print(String(flowRateMlPerHour) + " ml/h");
    } else {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("safe");

      lcd.setCursor(0, 1);
      lcd.print("");
    }

    waterFlow = 0;  // 유량 변수 초기화
    prevTime = currentTime;

    attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulse, RISING);
  }
}


void pulse() {
  // YF-S401 센서는 특정 주파수 범위에서 물의 유량을 측정합니다. 
  // 센서의 데이터 시트에서 5880은 센서가 1리터의 물을 측정할 때 발생하는 펄스의 수를 나타냅니다.
  // 센서가 1리터의 물이 흐를 때 5880 펄스를 생성한다고 가정하면, 1펄스당 물의 양은 1.0 / 5880.0 리터가 됩니다.
  waterFlow += 1.0 / 5880.0;  // 펄스당 유량 (리터)
}

/*

유량 센서는 물이 흐를 때마다 일정한 간격으로 펄스 신호를 발생시킵니다. 
이 펄스 신호는 물의 흐름에 비례하며, 센서가 생성하는 펄스의 수를 세면 물이 얼마나 많이 흐르는지 알 수 있습니다.


유량 센서의 작동 원리

유량 센서는 물이 흐를 때마다 "펄스"라는 신호를 발생시켜요. 마치 전등 스위치를 여러 번 켜고 끄는 것처럼요. 
펄스가 발생하는 빈도를 주파수라고 해요. 
물이 빠르게 흐르면 더 많은 펄스가 발생하고, 천천히 흐르면 적은 펄스가 발생해요.
*/
```
