This is for the 4 Button Membrane Keypad as used in a student project

```c
// Based off of http://www.instructables.com/id/1x4-Membrane-Keypad-w-Arduino/?ALLSTEPS 
//Group 808's SY1617 project
//Note: Phant no longer works. Replace with webhook to handle a push

#include <SparkFun-Spark-Phant.h>
#include <application.h>

int buttonPin1 = D0;
int buttonPin2 = D1;
int buttonPin3 = D2;
int buttonPin4 = D3;

int moodHappy = 0;
int moodSad = 0;
int moodNeutral = 0;
int moodMad = 0;


double value1 = 0; 
double value2 = 0; 

const char server[] = "https://data.sparkfun.com"; // Phant destination server
const char publicKey[] = "XXXX"; // Phant public key - HAS TO BE CHANGED
const char privateKey[] = "XXXX"; // Phant private key  - HAS TO BE CHANGED
Phant phant(server, publicKey, privateKey); // Create a Phant object

const int POST_RATE = 3000; // Time between posts, in ms.
unsigned long lastPost = 0; // global variable to keep track of last post time

void setup() {
    pinMode(buttonPin1, INPUT_PULLDOWN);
    pinMode(buttonPin2, INPUT_PULLDOWN);
    pinMode(buttonPin3, INPUT_PULLDOWN);
    pinMode(buttonPin4, INPUT_PULLDOWN);
    Serial.begin(9600); 
}

void loop() 
{
    moodHappy = digitalRead(buttonPin1);
    moodSad = digitalRead(buttonPin2);
    moodNeutral = digitalRead(buttonPin3);
    moodMad = digitalRead(buttonPin4);

    if(digitalRead(buttonPin1) == HIGH) 
    { 
        Serial.println("happy");
        postToPhant();
        Particle.publish("mood","happy",60,PRIVATE);
        while(digitalRead(buttonPin1) == HIGH);
    }
    
    if (digitalRead(buttonPin2) == HIGH) 
    {
        Serial.println("sad");
        postToPhant();
        Particle.publish("mood","sad",60,PRIVATE);
         while(digitalRead(buttonPin2) == HIGH);
    }
    
    if (digitalRead(buttonPin3) == HIGH) 
    {
        Serial.println("neutral");
        postToPhant();
        Particle.publish("mood","neutral",60,PRIVATE);
         while(digitalRead(buttonPin3) == HIGH);
    }  
    
    if (digitalRead(buttonPin4) == HIGH) 
    {
        Serial.println("mad");
        postToPhant();
        Particle.publish("mood","mad",60,PRIVATE);
         while(digitalRead(buttonPin4) == HIGH);
    }
    value1++;
    value2++;
}
int postToPhant()
{    
    // Use phant.add(<field>, <value>) to add data to each field.
    // Phant requires you to update each and every field before posting,
    // make sure all fields defined in the stream are added here.
    phant.add("mood", moodHappy);
    phant.add("mood", moodSad);
    phant.add("mood", moodMad);
    phant.add("mood", moodNeutral);
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
            Serial.print(c);    // Print the response for debugging help.
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
        {   // "400 Bad Request" means the Phant POST was formatted incorrectly.
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
    {   // If the connection failed, print a message:
        Serial.println("connection failed");
        retVal = -3;
    }
    client.stop();  // Close the connection to server.
    return retVal;  // Return error (or success) code.
}
```
