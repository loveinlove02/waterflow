
```C++
#include <Adafruit_NeoPixel.h>

#define PIN D7
#define NUM_LEDS 256 

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  Serial.begin(115200);
  
  strip.setBrightness(50);  
  strip.begin();
  clear_led();            
}

void loop() {
  safe();
  delay(5000);

  clear_led();

  danger();
  delay(5000);

  clear_led();
}

void safe() {
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

void danger() {
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
