```c
/*  simple momentary button code for Arduino or Photon
 *  
 *  Button connections:
 *  buttons have 2 leads. Connect one lead to ground, connect the other lead to a data pin on the micro-controller
 *  
 *  Using INPUT_PULLUP in pinMode:
 *  When button is not pressed, the internal pull-up resistor connects to 5 volts. 
 *  This causes the micro-controller to report "1" or HIGH. When the button is pressed, the 
 *  micro-controller pin is pulled to ground, causing it to report a "0", or LOW.
 */

int button_1 = 7;
int button_2 = 8;
// add new button variables as needed
//
boolean currentState1 = HIGH; //storage for current button state
boolean lastState1= HIGH; // storage for last button state
boolean currentState2 = HIGH; //storage for current button state
boolean lastState2= HIGH; // storage for last button state
// add new button states as needed
//
String mode; 

void setup(){
  Serial.begin(9600);
  // configure pins as input and enable the internal pull-up resistors
  pinMode(button_1, INPUT_PULLUP);
  pinMode(button_2, INPUT_PULLUP);
  // add new buttons as needed
}

void loop(){
  currentState1 = digitalRead(button_1);
  currentState2 = digitalRead(button_2);
  // add digitalRead()s as needed
  delay(15); // button debouncing
  //
  if (currentState1 == LOW && lastState1 == HIGH){ // if button has just been pressed
    mode = "happy";
    Serial.println(mode);
    delay(1); // button debouncing
  }
  lastState1 = currentState1;
  //
  if (currentState2 == LOW && lastState2 == HIGH){ // if button has just been pressed
    mode = "sad";
    Serial.println(mode);
    delay(1); // button debouncing
  }
  lastState2 = currentState2;
  //
  // add button conditionals as needed
}
```

