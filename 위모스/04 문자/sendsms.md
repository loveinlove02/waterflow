```C++
#include <SoftwareSerial.h>

SoftwareSerial myserial(D5, D6);

void setup() {
  Serial.begin(115200);
  myserial.begin(115200);

  delay(3000);
  myserial.println("AT");
  delay(1000);

  // SIM 카드 핀(전원 핀) 설정
  pinMode(9, OUTPUT);
  digitalWrite(9, HIGH);
}

void loop() {
  sendSMS("1025029532", 1);
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
