```C++
#include <SimpleTimer.h>

#define flowsensor D4

volatile double water = 0;
double last_water = 0;

// ----------------------------------------
long previousTime = 0;  
unsigned long cnt=1;        
long interval = 1000; 
// ----------------------------------------

SimpleTimer t1;

void setup() {
  Serial.begin(115200);
    
  pinMode(flowsensor, INPUT);
  attachInterrupt(digitalPinToInterrupt(flowsensor), flow, FALLING);
  
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
}


ICACHE_RAM_ATTR void flow(){
  water += (1/5888.0)*10000;
}
```
