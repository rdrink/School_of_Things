### Library
https://github.com/sparkfun/VKey_Voltage_Keypad/tree/master/Libraries/Arduino/src


### Voltage Keypad (discontinued so hard to find)
https://www.sparkfun.com/products/12080


### Code is in Hookup guide
https://learn.sparkfun.com/tutorials/vkey-voltage-keypad-hookup-guide?_ga=1.129174789.884128913.1488570632

For Particle Photon, these changes had to be made to the SparkFunVKeyVoltageKeypad.cpp file. Sample code was used to try the keypad. 

```c
const PROGMEM VKey::VKeyScale VKey::scales[MAX] =
{
   // min, step, max
   {115,    230,   2875},
   {115,    230,   2875}
};
```
