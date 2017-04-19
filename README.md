# Arduino Members

A library for defining `observable members` which are properties can be `observed` for changes to properly keep state in sync within an application. 

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
    'name':<property name>,
    'type':<change that occured>, 
    'value': <current value>, 
    'old':<previous value>,
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



#### Dependencies

1. [ArduinoJson](https://github.com/bblanchon/ArduinoJson)
