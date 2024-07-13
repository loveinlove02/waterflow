```cpp
#define flowsensor 2

volatile double water = 0;
double last_water = 0;

void setup() {
  Serial.begin(9600);
  pinMode(flowsensor, INPUT);
  
  attachInterrupt(digitalPinToInterrupt(flowsensor), flow, FALLING);
}

void loop() {
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
  } else if(last_water==water) {
    Serial.println("이전 상태와 같음");
  }
  
  last_water = water;
  
  delay(1000);
}

void flow(){
  water += (1/5888.0)*10000;
}
```
