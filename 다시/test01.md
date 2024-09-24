```C++
// YF-S201 유량 센서와 아두이노 Uno를 사용하여 유량(리터/시간)을 계산하는 코드

const int flowSensorPin = 2;  // 유량 센서의 신호 핀
volatile int flowPulseCount = 0;  // 펄스 카운터
float flowRate = 0.0;  // 유량 (리터/시간)
unsigned int flowMilliLiters = 0;  // 유량 (밀리리터)
unsigned long currentTime;
unsigned long prevTime = 0;
unsigned long elapsedTime;

void setup() {
  pinMode(flowSensorPin, INPUT);
  digitalWrite(flowSensorPin, HIGH);  // 내부 풀업 활성화

  attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulseCounter, FALLING);
  
  Serial.begin(9600);
}

void loop() {
  currentTime = millis();
  elapsedTime = currentTime - prevTime;

  if (elapsedTime > 1000) {  // 1초마다 계산
    detachInterrupt(digitalPinToInterrupt(flowSensorPin));
    
    flowRate = ((1000.0 / elapsedTime) * flowPulseCount) / 7.5;  // 펄스 간격으로 유량을 계산, 7.5는 펄스당 리터 수

    flowMilliLiters = (flowPulseCount / 7.5) * 1000;  // 펄스 간격으로 유량을 계산, 7.5는 펄스당 리터 수

    Serial.print("유량: ");
    Serial.print(flowRate);
    Serial.print(" 리터/시간, 총 양: ");
    Serial.print(flowMilliLiters);
    Serial.println(" 밀리리터");

    flowPulseCount = 0;  // 펄스 카운터 초기화
    prevTime = currentTime;

    attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulseCounter, FALLING);
  }
}

void pulseCounter() {
  flowPulseCount++;
}
```
