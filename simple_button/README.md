```c
// Wire one side of the button to D0 (or another digital pin and edit code), then the other side of the button to ground

int led = D7; // onboard LED
int pushButton = D2; // Push button is connected to D2
int counter=0;

// This routine runs only once upon reset
void setup() 
{
  pinMode(led, OUTPUT); // Initialize D0 pin as output
  pinMode(pushButton, INPUT_PULLUP); 
  // Initialize D2 pin as input with an internal pull-up resistor
}

// This routine loops forever
void loop() 
{
  int pushButtonState; 

  pushButtonState = digitalRead(pushButton);

  if(pushButtonState == LOW)
  { // If we push down on the push button
    digitalWrite(led, HIGH);  // Turn ON the LED
    counter++; // print couter to terminal
    Serial.println(counter); 
    delay(250); //allow the user to release button
  }
  else
  {
    digitalWrite(led, LOW);   // Turn OFF the LED 
  }

}
```
