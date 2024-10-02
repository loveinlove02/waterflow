```C++
#if defined(ESP32)
#include <WiFi.h>
#include <FirebaseESP32.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#include <FirebaseESP8266.h>
#endif

#include <SimpleTimer.h>
#include <Adafruit_NeoPixel.h>
#include <SoftwareSerial.h>

//Provide the token generation process info.
#include <addons/TokenHelper.h>

//Provide the RTDB payload printing info and other helper functions.
#include <addons/RTDBHelper.h>

/* 1. Define the WiFi credentials */
#define WIFI_SSID "soft"
#define WIFI_PASSWORD "ss2420052"

//#define WIFI_SSID "PISnet_4BDCD0"
//#define WIFI_PASSWORD "leeinhwan9532"

//#define WIFI_SSID "android9532"
//#define WIFI_PASSWORD "12345678"

//For the following credentials, see examples/Authentications/SignInAsUser/EmailPassword/EmailPassword.ino

/* 2. Define the API Key */
#define API_KEY "AIzaSyDYC8LuGNK3pSCIRJzrgf4vVA9gnqR_IFY"

/* 3. Define the RTDB URL */
#define DATABASE_URL "https://cho1-a8f77.firebaseio.com" //<databaseName>.firebaseio.com or <databaseName>.<region>.firebasedatabase.app

/* 4. Define the user Email and password that alreadey registerd or added in your project */
#define USER_EMAIL "softplay008@gmail.com"
#define USER_PASSWORD "swplay!@#"

//Define Firebase Data object
FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;

#define PIN D7
#define NUM_LEDS 256 

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, PIN, NEO_GRB + NEO_KHZ800);

SoftwareSerial myserial(D5, D6);

#define flowsensor D4

volatile double water = 0;
double last_water = 0;

// ----------------------------------------
long previousTime = 0;  
unsigned long cnt=1;        
long interval = 1000; 
// ----------------------------------------

SimpleTimer t1;

int isClear = 1;

void setup() {
  Serial.begin(115200);
  myserial.begin(115200);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  Serial.printf("Firebase Client v%s\n\n", FIREBASE_CLIENT_VERSION);

  /* Assign the api key (required) */
  config.api_key = API_KEY;

  /* Assign the user sign in credentials */
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;

  /* Assign the RTDB URL (required) */
  config.database_url = DATABASE_URL;

  /* Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons/TokenHelper.h

  Firebase.begin(&config, &auth);

  //Comment or pass false value when WiFi reconnection will control by your code or third party library
  Firebase.reconnectWiFi(true);
  
  delay(3000);
  myserial.println("AT");
  delay(1000);

  // SIM 카드 핀(전원 핀) 설정
  pinMode(9, OUTPUT);
  digitalWrite(9, HIGH);
  
  pinMode(flowsensor, INPUT);
  
  attachInterrupt(digitalPinToInterrupt(flowsensor), flow, FALLING);

  strip.setBrightness(50);  // 밝기 50은 최대
  strip.begin();

  clear_led();              // 모두 끄기
  isClear = -1;              
  
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

  Serial.print("isClear: ");
  Serial.println(isClear);
  
  if(water==0.0) {
    Serial.println("안전");
    safe();    
  }
  
  if(last_water!=water) {
    if(water>=100.0) {  
      Serial.println("위험");      
      
      clear_led();
      danger();

      isClear = 1;

      sendSMS("1025029532", 2);
    }
//    } else if(water>30.0) {
//      Serial.println("위험");
//      
//      
//      clear_led();
//      danger();
//      
//      isClear = 1;
//
//      sendSMS("1025029532", 2);
//    }

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

    if(isClear!=-1) {  // 1이면 끈다
      clear_led();
      isClear = -1;
    }
  }
  
  Serial.print("cnt: ");
  Serial.println(cnt); 
}


ICACHE_RAM_ATTR void flow(){
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
    myserial.write("ACBDACE0");   
    // 007700610072006E0069006E00670031
  } else if(a==2) {
    myserial.write("C704D5D8"); 
  } 
  // 위험 : C704D5D8
  // 경고 : ACBDACE0
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

  // 추가
  myserial.println("AT+RST");  // 모듈 리셋
  delay(1000);  // 모듈 재부팅 시간
}

void safe() {
  Firebase.setString(fbdo, "water/sensor1", "safe");
  
  strip.setPixelColor(49, strip.Color(0,255,0));
  strip.setPixelColor(62, strip.Color(0,255,0));
  strip.setPixelColor(65, strip.Color(0,255,0));

  strip.setPixelColor(45, strip.Color(0,255,0));
  strip.setPixelColor(44, strip.Color(0,255,0));

  strip.setPixelColor(77, strip.Color(0,255,0));
  strip.setPixelColor(76, strip.Color(0,255,0));

  strip.setPixelColor(52, strip.Color(0,255,0));
  strip.setPixelColor(59, strip.Color(0,255,0));
  strip.setPixelColor(68, strip.Color(0,255,0));

  strip.setPixelColor(97, strip.Color(0,255,0));
  strip.setPixelColor(98, strip.Color(0,255,0));
  strip.setPixelColor(99, strip.Color(0,255,0));
  strip.setPixelColor(100, strip.Color(0,255,0));
  strip.setPixelColor(101, strip.Color(0,255,0));
  
  strip.setPixelColor(108, strip.Color(0,255,0));

  strip.setPixelColor(54, strip.Color(0,255,0));
  strip.setPixelColor(55, strip.Color(0,255,0));
  strip.setPixelColor(56, strip.Color(0,255,0));
  strip.setPixelColor(71, strip.Color(0,255,0));
  strip.setPixelColor(72, strip.Color(0,255,0));
  strip.setPixelColor(87, strip.Color(0,255,0));
  strip.setPixelColor(88, strip.Color(0,255,0));
  strip.setPixelColor(103, strip.Color(0,255,0));
  
  strip.setPixelColor(158, strip.Color(0,255,0));
  strip.setPixelColor(161, strip.Color(0,255,0));
  strip.setPixelColor(174, strip.Color(0,255,0));
  strip.setPixelColor(177, strip.Color(0,255,0));
  strip.setPixelColor(190, strip.Color(0,255,0));

  strip.setPixelColor(173, strip.Color(0,255,0));
  strip.setPixelColor(163, strip.Color(0,255,0));
  strip.setPixelColor(179, strip.Color(0,255,0));
  strip.setPixelColor(155, strip.Color(0,255,0));
  strip.setPixelColor(187, strip.Color(0,255,0));

  strip.setPixelColor(195, strip.Color(0,255,0));
  strip.setPixelColor(204, strip.Color(0,255,0));

  strip.setPixelColor(209, strip.Color(0,255,0));
  strip.setPixelColor(210, strip.Color(0,255,0));
  strip.setPixelColor(211, strip.Color(0,255,0));
  strip.setPixelColor(212, strip.Color(0,255,0));
  strip.setPixelColor(213, strip.Color(0,255,0));


  strip.setPixelColor(166, strip.Color(0,255,0));
  strip.setPixelColor(167, strip.Color(0,255,0));
  strip.setPixelColor(168, strip.Color(0,255,0));
  strip.setPixelColor(183, strip.Color(0,255,0));
  strip.setPixelColor(184, strip.Color(0,255,0));
  strip.setPixelColor(199, strip.Color(0,255,0));
  strip.setPixelColor(200, strip.Color(0,255,0));
  strip.setPixelColor(215, strip.Color(0,255,0));

  strip.show();  
}

void warring() {
  Firebase.setString(fbdo, "water/sensor1", "warring");

  strip.setPixelColor(63, strip.Color(255,0,0));
  strip.setPixelColor(64, strip.Color(255,0,0));
  strip.setPixelColor(79, strip.Color(255,0,0));
  strip.setPixelColor(80, strip.Color(255,0,0));
  
  strip.setPixelColor(81, strip.Color(255,0,0));
  strip.setPixelColor(82, strip.Color(255,0,0));
  strip.setPixelColor(83, strip.Color(255,0,0));
  strip.setPixelColor(84, strip.Color(255,0,0));

  strip.setPixelColor(97, strip.Color(255,0,0));
  strip.setPixelColor(110, strip.Color(255,0,0));

  strip.setPixelColor(99, strip.Color(255,0,0));
  strip.setPixelColor(108, strip.Color(255,0,0));

  strip.setPixelColor(112, strip.Color(255,0,0));
  strip.setPixelColor(113, strip.Color(255,0,0));
  strip.setPixelColor(114, strip.Color(255,0,0));
  strip.setPixelColor(115, strip.Color(255,0,0));
  strip.setPixelColor(116, strip.Color(255,0,0));

  strip.setPixelColor(86, strip.Color(255,0,0));
  strip.setPixelColor(89, strip.Color(255,0,0));
  strip.setPixelColor(102, strip.Color(255,0,0));
  strip.setPixelColor(105, strip.Color(255,0,0));
  strip.setPixelColor(118, strip.Color(255,0,0));

  strip.setPixelColor(87, strip.Color(255,0,0));
  strip.setPixelColor(88, strip.Color(255,0,0));
  strip.setPixelColor(103, strip.Color(255,0,0));
  strip.setPixelColor(104, strip.Color(255,0,0));
  strip.setPixelColor(119, strip.Color(255,0,0));

  strip.setPixelColor(159, strip.Color(255,0,0));
  strip.setPixelColor(160, strip.Color(255,0,0));
  strip.setPixelColor(175, strip.Color(255,0,0));
  strip.setPixelColor(176, strip.Color(255,0,0));
  strip.setPixelColor(191, strip.Color(255,0,0));
  strip.setPixelColor(192, strip.Color(255,0,0));

  strip.setPixelColor(193, strip.Color(255,0,0));
  strip.setPixelColor(194, strip.Color(255,0,0));
  strip.setPixelColor(195, strip.Color(255,0,0));
  strip.setPixelColor(196, strip.Color(255,0,0));


  strip.setPixelColor(173, strip.Color(255,0,0));
  strip.setPixelColor(172, strip.Color(255,0,0));
  strip.setPixelColor(171, strip.Color(255,0,0));
  strip.setPixelColor(170, strip.Color(255,0,0));

  strip.setPixelColor(153, strip.Color(255,0,0));
  strip.setPixelColor(166, strip.Color(255,0,0));
  strip.setPixelColor(169, strip.Color(255,0,0));
  strip.setPixelColor(182, strip.Color(255,0,0));
  strip.setPixelColor(185, strip.Color(255,0,0));
  strip.setPixelColor(198, strip.Color(255,0,0));
  
  strip.show();  
}

void danger() {
  Firebase.setString(fbdo, "water/sensor1", "danger");
  
    strip.setPixelColor(48, strip.Color(255,0,0));
  strip.setPixelColor(63, strip.Color(255,0,0));
  strip.setPixelColor(64, strip.Color(255,0,0));

  strip.setPixelColor(46, strip.Color(255,0,0));
  strip.setPixelColor(45, strip.Color(255,0,0));

  strip.setPixelColor(78, strip.Color(255,0,0));
  strip.setPixelColor(77, strip.Color(255,0,0));

  strip.setPixelColor(51, strip.Color(255,0,0));
  strip.setPixelColor(60, strip.Color(255,0,0));
  strip.setPixelColor(67, strip.Color(255,0,0));


  strip.setPixelColor(42, strip.Color(255,0,0));
  strip.setPixelColor(53, strip.Color(255,0,0));
  strip.setPixelColor(58, strip.Color(255,0,0));
  strip.setPixelColor(69, strip.Color(255,0,0));
  strip.setPixelColor(74, strip.Color(255,0,0));
  strip.setPixelColor(85, strip.Color(255,0,0));
  
  strip.setPixelColor(57, strip.Color(255,0,0));
  strip.setPixelColor(56, strip.Color(255,0,0));

  strip.setPixelColor(96, strip.Color(255,0,0));
  strip.setPixelColor(97, strip.Color(255,0,0));
  strip.setPixelColor(98, strip.Color(255,0,0));
  strip.setPixelColor(99, strip.Color(255,0,0));
  strip.setPixelColor(100, strip.Color(255,0,0));
  strip.setPixelColor(101, strip.Color(255,0,0));
  strip.setPixelColor(102, strip.Color(255,0,0));
  strip.setPixelColor(103, strip.Color(255,0,0));


  strip.setPixelColor(175, strip.Color(255,0,0));
  strip.setPixelColor(145, strip.Color(255,0,0));
  strip.setPixelColor(158, strip.Color(255,0,0));
  strip.setPixelColor(161, strip.Color(255,0,0));
  strip.setPixelColor(174, strip.Color(255,0,0));
  strip.setPixelColor(177, strip.Color(255,0,0));
  strip.setPixelColor(190, strip.Color(255,0,0));
  strip.setPixelColor(193, strip.Color(255,0,0));

  strip.setPixelColor(162, strip.Color(255,0,0));
  strip.setPixelColor(173, strip.Color(255,0,0));
  strip.setPixelColor(178, strip.Color(255,0,0));

  strip.setPixelColor(156, strip.Color(255,0,0));
  strip.setPixelColor(188, strip.Color(255,0,0));

  strip.setPixelColor(164, strip.Color(255,0,0));
  strip.setPixelColor(171, strip.Color(255,0,0));
  strip.setPixelColor(180, strip.Color(255,0,0));
  strip.setPixelColor(211, strip.Color(255,0,0));
  
  strip.setPixelColor(223, strip.Color(255,0,0));
  strip.setPixelColor(222, strip.Color(255,0,0));
  strip.setPixelColor(221, strip.Color(255,0,0));
  strip.setPixelColor(220, strip.Color(255,0,0));
  strip.setPixelColor(219, strip.Color(255,0,0));
  strip.setPixelColor(218, strip.Color(255,0,0));

  strip.setPixelColor(166, strip.Color(255,0,0));
  strip.setPixelColor(169, strip.Color(255,0,0));
  strip.setPixelColor(182, strip.Color(255,0,0));
  strip.setPixelColor(185, strip.Color(255,0,0));
  strip.setPixelColor(198, strip.Color(255,0,0));
  strip.setPixelColor(201, strip.Color(255,0,0));

  strip.setPixelColor(167, strip.Color(255,0,0));
  strip.setPixelColor(168, strip.Color(255,0,0));
  strip.setPixelColor(183, strip.Color(255,0,0));
  strip.setPixelColor(184, strip.Color(255,0,0));
  strip.setPixelColor(199, strip.Color(255,0,0));
  strip.setPixelColor(200, strip.Color(255,0,0));
  
  strip.show();
}

void clear_led() {
  for(int i=0; i<strip.numPixels(); i++) {
    strip.setPixelColor(i, strip.Color(0,0,0));      
  }
  strip.show();
}
```
