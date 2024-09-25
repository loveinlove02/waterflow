```C++
#include <SimpleTimer.h>

#define flowsensor 2

volatile int flowPulseCount = 0;  // 펄스 카운터. 센서가 감지한 펄스의 개수를 저장.
double flowRate = 0.0;            // 현재 유량 (mL/min). 현재 유량을 분당 밀리리터로 저장.
double waterVolume = 0.0;         // 누적된 물의 양 (mL). 누적된 물의 양을 밀리리터로 저장.
unsigned long lastTime = 0;       
SimpleTimer timer;

// YF-S201 유량 센서의 펄스당 리터 변환 계수 (5888 펄스 = 1리터)
const float calibrationFactor = 5888.0;

// 유량 임계값 설정 (경고 기준)
const double warningLevel1 = 1000.0;  // 1000 mL/min
const double warningLevel2 = 3000.0;  // 3000 mL/min
const double warningLevel3 = 5000.0;  // 5000 mL/min
const double suddenIncreaseThreshold = 2000.0; // 갑작스러운 유량 증가 기준

void setup() {
  Serial.begin(9600);

  pinMode(flowsensor, INPUT); // 핀 모드 설정

  // 센서가 펄스를 발생시킬 때마다 pulseCounter 함수를 호출합니다. 이 함수(pulseCounter)는 펄스의 개수를 증가시킵니다.
  // 펄스 발생 시 인터럽트
  attachInterrupt(digitalPinToInterrupt(flowsensor), pulseCounter, FALLING); 
  
  // 1초마다 calculateFlowRate 함수를 호출하여 유량을 계산합니다
  timer.setInterval(1000, calculateFlowRate);                                
}

void loop() {
  timer.run(); // 타이머 실행
}

// 센서가 펄스를 발생시킬 때마다 호출되어 flowPulseCount를 증가시킵니다.
void pulseCounter() {
  flowPulseCount++;  // 펄스가 발생할 때마다 카운트 증가 (펄스의 개수를 증가)
}

void calculateFlowRate() {
  unsigned long currentTime = millis();
  unsigned long timeInterval = currentTime - lastTime;  // 지난 시간 (밀리초)
  
  // 펄스 카운트로 유량(mL/min) 계산. 펄스 수를 사용하여 분당 유량을 계산합니다.
  flowRate = ((flowPulseCount / calibrationFactor) * 60000 * 1000) / timeInterval; // mL/min

  /*
   
   flowPulseCount: 센서가 측정한 펄스의 총 수입니다. 물이 흐를 때마다 센서가 펄스를 발생시키므로, 이 수치는 물의 흐름을 나타냅니다.
   calibrationFactor : 센서의 캘리브레이션 계수입니다. 센서의 사양에 따라 설정된 값으로, 보통 펄스당 물의 양(리터)을 나타냅니다. 
                       예를 들어, 5888 펄스가 1리터에 해당한다고 가정합니다.
   60000 : 1분은 60,000 밀리초이기 때문에, 유량을 분당으로 변환하기 위해 사용됩니다.
   timeInterval : 마지막 유량 계산 이후 경과된 시간(밀리초)입니다. 이 값을 사용하여 유량을 현재 시간 간격으로 조정합니다.


  flowPulseCount / calibrationFactor : 펄스 수를 calibrationFactor로 나누면, 이 펄스 수가 얼마나 많은 리터에 해당하는지를 계산합니다.
                                       예를 들어, flowPulseCount가 5888 펄스이고, calibrationFactor가 5888이라면, 
                                       flowPulseCount / calibrationFactor는 1리터가 됩니다.
                                  
  (flowPulseCount / calibrationFactor) * 60000 : 위에서 계산한 리터를 1분으로 변환하기 위해, 60000을 곱합니다.
                                                 예를 들어, flowPulseCount / calibrationFactor가 1 리터라면, 
                                                 1분 동안의 유량을 나타내기 위해 60,000을 곱해줍니다.
   
  ((flowPulseCount / calibrationFactor) * 60000) / timeInterval : 시간 간격 조정.
                      timeInterval은 계산에 사용된 실제 시간(밀리초)입니다. 이를 나누어 실제 시간 간격에 맞는 유량을 계산합니다.
                      예를 들어, timeInterval이 1000 밀리초(1초)라면, 유량을 1분 단위로 조정하기 위해 60000을 나누어 1분 동안의 유량을 계산합니다.
  
  */

  

  // 흐른 물의 양을 누적 (밀리리터로 계산)
  waterVolume += (flowPulseCount / calibrationFactor) * 1000; // mL
  
  // 유량에 따른 경고 메시지 출력
  if (flowRate == 0.0) {
    Serial.println("Level 0: 물이 흐르지 않습니다.");
  } else if (flowRate > warningLevel3) {
    Serial.println("경고: 유량이 매우 높습니다! (Level 3)");
  } else if (flowRate > warningLevel2) {
    Serial.println("경고: 유량이 높습니다! (Level 2)");
  } else if (flowRate > warningLevel1) {
    Serial.println("경고: 유량이 중간 수준입니다! (Level 1)");
  }
  
  // 유량이 갑자기 급격히 증가하면 경고 출력
  static double lastFlowRate = 0.0;  // 이전 유량을 저장
  if ((flowRate - lastFlowRate) > suddenIncreaseThreshold) {
    Serial.println("경고: 유량이 갑자기 급격히 증가했습니다!");
  }
  
  // 결과 출력
  Serial.print("현재 유량 (mL/min): ");
  Serial.println(flowRate);
  
  Serial.print("누적된 물의 양 (mL): ");
  Serial.println(waterVolume);
  
  // 카운트 리셋
  flowPulseCount = 0;
  lastFlowRate = flowRate;  // 이번 유량을 이전 유량으로 저장
  lastTime = currentTime;
}

/*
 YF-S201 유량 센서를 사용하여 물의 유량을 측정하고, 특정 유량에 따라 경고 메시지를 출력하는 역할을 합니다. 
 유량은 시간 단위로 흐르는 물의 양을 나타내며, 여기서는 분당(mL/min)으로 측정됩니다.

 (1) 펄스의 개념

  . 펄스는 유량 센서가 물의 흐름을 측정하기 위해 발생시키는 작은 전기 신호입니다.
  . 센서 내부에는 회전하는 날개가 있으며, 물이 흐르면 날개가 회전합니다. 
    이 회전에 따라 센서가 전기 신호를 발생시키고, 이 신호가 펄스라고 불립니다.

  . 더 많은 펄스가 발생하면 더 많은 물이 흐르고 있다는 의미입니다.

  
*/


/*
유량을 분당(mL/min) 단위로 계산했습니다. 이는 대부분의 유량 센서와 관련된 시스템에서 유량을 측정하는 표준 방식입니다. 
분당 단위는 시간 단위로 계산된 물의 양을 쉽게 이해할 수 있게 해줍니다. 

분당으로 측정하는 이유는 다음과 같습니다:

이유 1: 표준 측정 단위

분당(mL/min)은 많은 유량 센서와 관련 시스템에서 표준으로 사용되는 측정 단위입니다. 
센서의 사양이나 매뉴얼에서도 이 단위를 기준으로 유량을 측정하도록 설계된 경우가 많습니다.

이유 2: 편리한 시간 단위

유량을 분당으로 계산하면, 일상적인 시간 단위(분)를 기준으로 물의 흐름을 쉽게 이해할 수 있습니다. 
예를 들어, 배수로에서 물이 3000 mL/min로 흐를 때, 이는 1분 동안 3리터의 물이 흐른다는 의미입니다. 
이 정보는 물의 흐름을 빠르게 파악하는 데 유용합니다.


이유 3: 센서의 특성에 맞춤

대부분의 유량 센서는 분당으로 유량을 계산하여 값을 출력합니다. 
이는 센서가 측정한 펄스 수를 분 단위로 환산하는 방식으로 유량을 나타내기 때문입니다. 
따라서 센서의 출력을 직접적으로 사용하기 위해 분당 단위로 변환합니다.
*/



// 시간 단위에 따른 유량 계산 : 센서의 펄스 수를 초 단위로 계산한 후, 이를 분 단위로 변환하여 mL/min으로 계산
// flowRate = ((flowPulseCount / calibrationFactor) * 60000) / timeInterval;
  
/*
유량 : 물이나 공기 같은 유체가 일정 시간 동안 어느 한 지점을 통과하는 양

유량은 "얼마나 많은 물(또는 공기)이 한 곳을 지나가는가"를 측정하는 것

예를 들어, 강에서 흐르는 물의 양을 생각해 볼 수 있는데, 
유량이 클수록 물이 많이 흐르고, 유량이 작을수록 물이 적게 흐릅니다. 
보통 유량은 시간당 통과하는 물의 부피(예: 리터/초 또는 m³/초)로 나타냅니다.

유량센서 

https://blog.naver.com/ubicomputing/220523467220

*/
```
