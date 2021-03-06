```c
/*
For use with this chip:
https://www.sparkfun.com/products/9868
Sample rate will determine how often an new value is added to the array
The array size divided by the sample rate will tell you how much 
time the array represents
snd_avg is a constantly recalculated average of the values in the array
snd_min and snd_max are reset every minute, but keep in mind that 
very large and very small values will remain in the array until shifted
out, in this example, over the course of 1 minute.

*/

const long sample_rate = 50; // time between samples (in miliseconds)
const int array_size = 1200; // 1000/50=20 * 60=1200
int snd_array[array_size] = {};
int snd_max = 0 ;
int snd_min = 4096 ;
double snd_avg = 0;

const int blink_thresh = 3000;

unsigned long broadcast_interval = 1000;
unsigned long last_broadcast = 0;
unsigned long lastSample = 0 ;
unsigned long lastReset = 0 ;

void averageReading(int value) 
{
    unsigned long now = millis();

    if((now - lastSample) > sample_rate) 
    {
        // Shift all the values right by 1
        for(int i = array_size-1; i >= 1; i--) 
        {
            snd_array[i] = snd_array[i-1]; 
            if((snd_array[i] < snd_min) && (snd_array[i] > 0))
            {
                snd_min = snd_array[i];
            }
            if(snd_array[i] > snd_max)
            {
                snd_max = snd_array[i];
                
            }
        }
    
        snd_array[0] = value; 
    
        // Average of all non-zero elements in the array
        double avg_sum = 0; 
        int size = 0 ;
        for (int a=0; a < array_size; a++) 
        {
            if(snd_array[a] > 0)
            {
                size++ ;
                avg_sum += snd_array[a];
            }
        }
        snd_avg = avg_sum / size;
        
        lastSample = now ;
    }
}

//blink the onboard LED (D7) if reading is louder than threshold
void blinkMic(int reading) 
{
    if(reading > blink_thresh) 
    {
        digitalWrite(D7, HIGH);
    } 
    else 
    {
        digitalWrite(D7, LOW);
    }
}

//check to see if we should reset min and max
void checkReset() 
{
    unsigned long now = millis();
    
    //if it has been one minute since resetting min and max, reset
    if((now - lastReset) >= 60000) 
    {
        snd_max = 0 ;
        snd_min = 4095 ;
        lastReset = now ;

    }
}

//print out the values of avg, min and max to terminal
void broadcast() 
{
    unsigned long now = millis();
    if((now - last_broadcast) > broadcast_interval) {
        Serial.print("Avg: "); Serial.println(snd_avg);
        Serial.print("Min: "); Serial.println(snd_min);
        Serial.print("Max: "); Serial.println(snd_max);

        last_broadcast = now;
    }
}

void setup() 
{
    Serial.begin(9600);
    pinMode(A0, INPUT); // mic AUD connected to Analog pin 0
    pinMode(D7, OUTPUT); // flash on-board LED
    Particle.variable("avg",snd_avg);
    Particle.variable("min",snd_min);
    Particle.variable("max",snd_max);
}

void loop() 
{
    int mic_reading = analogRead(0); //Serial.println(mic_reading);
    blinkMic(mic_reading) ;
    averageReading(mic_reading) ; 
    broadcast() ;
    
    //this will reset the min and max, but keep in mind that those
    //large and small values will remain in the array for up to one
    //minute depending on the sample_rate
    checkReset() ;
}
```

