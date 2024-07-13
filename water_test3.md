```cpp
#include <SimpleTimer.h>

#define flowsensor 2

volatile double water = 0;
double last_water = 0;

// ----------------------------------------
long previousTime = 0;  
unsigned long cnt=1;        
long interval = 5000; 
// ----------------------------------------

SimpleTimer t1;
SimpleTimer t2;

void setup() {
  Serial.begin(9600);
  pinMode(flowsensor, INPUT);
  
  attachInterrupt(digitalPinToInterrupt(flowsensor), flow, FALLING);

  t1.setInterval(1000,fn1);
  t2.setInterval(1000,fn2);
}

void loop() {
  t1.run();
  t2.run();
}

void fn1(void) {
  Serial.print("last water : ");
  Serial.println(last_water);
  
  Serial.print("water: ");
  Serial.println(water);

  if(last_water!=water) {
    if(water>=2000.0 && water<=4000.0) {  
      Serial.println("경고1");
    } else if(water>4000.0 && water<=5000.0) {
      Serial.println("경고2");
    } else if(water>5000.0) {
      Serial.println("경고3");
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
  }

  if(cnt<1) {
    water = 0;
  }
  Serial.print("cnt: ");
  Serial.println(cnt); 
}

void fn2(void) {
  
}

void flow(){
  water += (1/5888.0)*10000;
}
```
