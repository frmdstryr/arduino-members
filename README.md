# Arduino Members

A library for defining `observable members` which are properties that can be  watched or `observed` for changes to properly keep state in sync within an application. 

Orignally forked from [https://github.com/tomstewart89/Callback](https://github.com/tomstewart89/Callback)


### Usage

Within your class define state variables using the `Member` directive. 

```cpp

#include <Member.h>
#include <ArduinoJson.h>

#define LED_PIN 13

class Demo {
        Demo();

protected:

    // State
    Member(bool,ledActive,false);
    Member(float,temp,0.0);
    Member(float,humidity,0.0);
    // etc...

    // Init
    void bindObservers();        
    
    // Observers
    void onWeatherChanged(JsonObject &change);
    void onLedChanged(JsonObject &change);


    
  }
    
```

Next bind `observer` or callback functions / methods to be fired when the state of the member changes

```cpp

Demo::Demo() {
    Demo &self = *(this);
    self.bindObservers();
}


void Demo::bindObservers() {

    MethodObserver<Demo> onWeatherChanged(this, &Demo::onWeatherChanged);
    temp.observe(onWeatherChanged);
    humidity.observe(onWeatherChanged);
    
    MethodObserver<Demo> onLedChanged(this, &Demo::onLedChanged);
    ledActive.observe(onLedChanged);
}

```

Finally, in your callbacks, handle the changes.  A change is a JsonObject with the format 

```json
{
    "name":"property name",
    "type":"update", 
    "value": "<current value>", // Type is same as member type   
    "old":"<previous value>",  // Type is same as member type 
} 

```

Which allows you to do interesting things with a simple interface. It eliminates the need to do repeated loops to check and react to changes in state all the time.

```cpp

// Only update LED state when a change to ledActive occurs
void onLedChanged(JsonObject &change) {
    change.printTo(Serial);
    digitalWrite(LED_PIN,(ledActive)? HIGH: LOW);
}

// When either temp or humidity changes, update the led state if needed.
void onWeatherChanged(JsonObject &change) {
    change.printTo(Serial);
    String prop = change['name'];
    
   if (prop.equals("temp") {
        // We know the temp changed
        Serial.print("Temp changed to: ");
        Serial.println(temp);
        if (temp>75) {
            Serial.println("Temp is hot");
            ledActive = true; // Triggers onLedChanged  if it was previously false, otherwise does nothing
        } else if (temp < 60) {
            ledActive = false; // Triggers onLedChanged if it was previously true, otherwise does nothing
        }
   } else if (prop.equals("humidity") {
        Serial.print("Humidity changed to: ");
        Serial.println(humidity);
   }
}


```


And of course you must update the properties however you normally do.

```cpp

Demo demo;

setup() {
    // Configure pins
    pinMode(LED_PIN,OUTPUT);
}

loop() {
    // Read temp and humidity from sensor, in loop, interupts, however you roll
    // Note: Even though these are is always being set, the callback only fires when the value actually changes
    demo.temp = TempSensor.read(); 
    demo.humidity = HumiditySensor.read();
    // etc...
}
```

This library really helps simplify code that must set a system to a specific state, simply update the member values and let the callbacks do the rest.

#### Dependencies

1. [ArduinoJson](https://github.com/bblanchon/ArduinoJson)
