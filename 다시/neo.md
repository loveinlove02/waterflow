```C++
#include <Adafruit_NeoPixel.h>
#define PIN 6
#define NUM_LEDS 256 

Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.setBrightness(50);  // 밝기 50은 최대
  strip.begin();
 // strip.show(); 
}
void loop() {
  for(int i=0; i<strip.numPixels(); i++) {
    strip.setPixelColor(i, strip.Color(255,0,0));
    strip.show();
  }
}

```
