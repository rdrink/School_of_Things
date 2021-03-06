```c
// This #include statement was automatically added by the Particle IDE.
#include <SparkFunPhant.h>

// This #include statement was automatically added by the Particle IDE.
//#include <neopixel.h>

/* simple momentary button and neopixel code for Arduino or Photon

    Botton connections:
    buttons have 2 leads. Connect one lead to ground, connect the other lead to a data pin

    When using INPUT_PULLUP in pinMode:
    When button is not pressed, the internal pull-up resistor connects to 5 volts.
    This causes the micro-controller to report "1" or HIGH. When the button is pressed, the
    micro-controller pin is pulled to ground, causing it to report a "0", or LOW.

    Neopixel strip connected to pin 6
*/

// Includes for NeoPixel LED's
//#include <Adafruit_NeoPixel.h>

// define PIN 6 as output, for NeoPixel Library
//#define PIN 6 
// This #include statement was automatically added by the Particle IDE.

#include "Particle.h"
#include "neopixel.h"
#include <Wire.h>

//phant code
const char server[] = "data.sparkfun.com"; // Phant destination server
const char publicKey[] = "xxx"; // Phant public key - HAS TO BE CHANGED
const char privateKey[] = "xxx"; // Phant private key  - HAS TO BE CHANGED
Phant phant(server, publicKey, privateKey); // Create a Phant object

const int POST_RATE = 3000; // Time between posts, in ms.
unsigned long lastPost = 0; // global variable to keep track of last post time


SYSTEM_MODE(AUTOMATIC);

// IMPORTANT: Set pixel COUNT, PIN and TYPE
#define PIXEL_PIN D0
#define PIXEL_COUNT 1
#define PIXEL_TYPE WS2812B
// Instantiate the NeoPixel Object - this is the engine of the N-Pix Library
Adafruit_NeoPixel strip = Adafruit_NeoPixel(1, D0,  WS2812B);

// bitwise variable declaration for each color (boolean)
uint32_t red = strip.Color(0, 255, 0);  // Red
uint32_t green = strip.Color(255, 0, 0);  // Green
uint32_t blue = strip.Color(0, 0, 255);  // Blue
uint32_t cyan = strip.Color(255, 0, 255);  // Cyan
uint32_t magenta = strip.Color(0, 255, 255);  // Magenta
uint32_t yellow = strip.Color(255, 255, 0);  // Yellow

// neopix / color variables
//#define arraySize 6 //this was giving an error on IDE, so manually put in 6
String colors[6] = { "red", "green", "blue", "cyan", "magenta", "yellow" };
String currentColor[6] = {  "red", "green", "blue", "cyan", "magenta", "yellow"};

int colorCount = 0;
unsigned long myTimer;
unsigned long colorChange_interval = 3000; // <--- this determines, in milliseconds, the time each color is on, change as needed
unsigned long last_colorChange = 0;

// button variables
int button = 1;
boolean currentState = HIGH; // storage for current button state
boolean lastState = HIGH; // storage for last button state

void setup() {
  Serial.begin(9600);
  //configure pins as input and enable the internal pull-up resistors
  pinMode(button, INPUT_PULLUP);
  strip.begin();
  strip.setBrightness(50);  // range = 0 to 255 (255 is full brightness)
  strip.show(); // Initialize all pixels to 'off'
}

void loop() {

  buttonPushState();

  myTimer = millis();
  if ((myTimer - last_colorChange) > colorChange_interval) {
    if (colorCount == 0) {
      allColorOn(red);
    } else if (colorCount == 1) {
      allColorOn(green);
    } else if (colorCount == 2) {
      allColorOn(blue);
    } else if (colorCount == 3) {
      allColorOn(cyan);
    } else if (colorCount == 4) {
      allColorOn(magenta);
    } else if (colorCount == 5) {
      allColorOn(yellow);
    }
    
     //buttonPushState();
    Serial.println(colors[colorCount]); //<--- this prints the current cycling color
    colorCount++;
    if (colorCount == 6) {
      colorCount = 0;
    }
    last_colorChange = myTimer;
  }

}

// function: checks the button and prints the current color
//-------------------------------------------------------------
void buttonPushState() {
  currentState = digitalRead(button);
  delay(15); // button debouncing
  if (currentState == LOW && lastState == HIGH) { // if button has just been pressed
    Serial.println("BUTTON PRESSED!") ;
    Serial.println("Sending " + currentColor[colorCount-1] + " to postToPhant"); // <--- this prints the current color on each button press
    postToPhant(currentColor[colorCount-1]) ; //send the current color to phant function

    delay(1); // button debouncing
  }
  lastState = currentState;
}

// function: makes all neopixels a single color
//-------------------------------------------------------------
void allColorOn(uint32_t c) {
  strip.setPixelColor(0, c);
  strip.setPixelColor(1, c);
  strip.setPixelColor(2, c);
  strip.show();
}

//post to phant code
int postToPhant(String currentColor)
{    
    // Use phant.add(<field>, <value>) to add data to each field.
    // Phant requires you to update each and every field before posting,
    // make sure all fields defined in the stream are added here.
    phant.add("color", currentColor);
    
Serial.println("Pushing " + currentColor + " to phant") ;

    TCPClient client;
    char response[512];
    int i = 0;
    int retVal = 0;
    
    if (client.connect(server, 80)) // Connect to the server
    {
		// Post message to indicate connect success
        Serial.println("Posting!"); 
		
		// phant.post() will return a string formatted as an HTTP POST.
		// It'll include all of the field/data values we added before.
		// Use client.print() to send that string to the server.
        client.print(phant.post());
        delay(1000);
		// Now we'll do some simple checking to see what (if any) response
		// the server gives us.
        while (client.available())
        {
            char c = client.read();
            Serial.print(c);	// Print the response for debugging help.
            if (i < 512)
                response[i++] = c; // Add character to response string
        }
		// Search the response string for "200 OK", if that's found the post
		// succeeded.
        if (strstr(response, "200 OK"))
        {
            Serial.println("Post success!");
            retVal = 1;
        }
        else if (strstr(response, "400 Bad Request"))
        {	// "400 Bad Request" means the Phant POST was formatted incorrectly.
			// This most commonly ocurrs because a field is either missing,
			// duplicated, or misspelled.
            Serial.println("Bad request");
            retVal = -1;
        }
        else
        {
			// Otherwise we got a response we weren't looking for.
            retVal = -2;
        }
    }
    else
    {	// If the connection failed, print a message:
        Serial.println("connection failed");
        retVal = -3;
    }
    client.stop();	// Close the connection to server.
    return retVal;	// Return error (or success) code.
}
```

