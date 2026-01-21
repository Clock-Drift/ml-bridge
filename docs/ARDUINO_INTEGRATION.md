# Arduino Integration Guide

This guide shows you how to receive ML Bridge predictions on your Arduino and control hardware based on the results.

## Quick Start

### 1. Hardware Setup
- Connect Arduino to your computer
- Connect an LED to pin 2 (or any digital pin)
- Open Serial Bridge application
- Connect to your Arduino in Serial Bridge

### 2. ML Bridge Setup
1. Open ML Bridge
2. Go to **Deploy** tab
3. Select **Serial Bridge** protocol
4. Enter your device ID (e.g., `arduino_1`)
5. Choose data format: **JSON** or **CSV**

### 3. Upload Arduino Code
Choose the example that matches your selected format:

---

## Format 1: JSON (Recommended)

**Best for:** Projects that need extensibility, multiple data fields, or professional applications.

**What you receive:**
```json
{"label":"class_1","confidence":0.85}
```

### Arduino Code (JSON)

```cpp
#include <ArduinoJson.h>

#define LED_PIN 2

void setup() {
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);
  Serial.println("Ready to receive predictions!");
}

void loop() {
  if (Serial.available()) {
    String data = Serial.readStringUntil('\n');
    data.trim();
    
    // Parse JSON
    StaticJsonDocument<200> doc;
    DeserializationError error = deserializeJson(doc, data);
    
    if (!error) {
      const char* label = doc["label"];
      float confidence = doc["confidence"];
      
      // Control LED based on prediction
      if (strcmp(label, "class_1") == 0) {
        digitalWrite(LED_PIN, HIGH);
        Serial.println("LED ON");
      } else {
        digitalWrite(LED_PIN, LOW);
        Serial.println("LED OFF");
      }
    }
  }
}
```

**Install ArduinoJson library:**
1. Arduino IDE → Tools → Manage Libraries
2. Search "ArduinoJson"
3. Install version 6.x

---

## Format 2: CSV (Simple)

**Best for:** Beginners, minimal code, or memory-constrained projects.

**What you receive:**
```
class_1,0.85
```

### Arduino Code (CSV)

```cpp
#define LED_PIN 2

void setup() {
  Serial.begin(9600);
  pinMode(LED_PIN, OUTPUT);
  Serial.println("Ready to receive predictions!");
}

void loop() {
  if (Serial.available()) {
    String data = Serial.readStringUntil('\n');
    data.trim();
    
    // Parse CSV: label,confidence
    int commaIndex = data.indexOf(',');
    String label = data.substring(0, commaIndex);
    float confidence = data.substring(commaIndex + 1).toFloat();
    
    // Control LED based on prediction
    if (label == "class_1") {
      digitalWrite(LED_PIN, HIGH);
      Serial.println("LED ON");
    } else {
      digitalWrite(LED_PIN, LOW);
      Serial.println("LED OFF");
    }
  }
}
```

**No libraries needed!** This uses only built-in Arduino functions.

---

## Advanced Examples

### Confidence Threshold
Only act on high-confidence predictions:

```cpp
if (confidence > 0.7) {  // Only if 70%+ confident
  if (label == "class_1") {
    digitalWrite(LED_PIN, HIGH);
  }
}
```

### Multiple Classes
Control different outputs:

```cpp
if (label == "class_1") {
  digitalWrite(LED_PIN, HIGH);
  digitalWrite(MOTOR_PIN, LOW);
} else if (label == "class_2") {
  digitalWrite(LED_PIN, LOW);
  digitalWrite(MOTOR_PIN, HIGH);
} else {
  // class_0 or unknown
  digitalWrite(LED_PIN, LOW);
  digitalWrite(MOTOR_PIN, LOW);
}
```

### PWM Based on Confidence
Dim LED based on confidence:

```cpp
int brightness = (int)(confidence * 255);
analogWrite(LED_PIN, brightness);
```

---

## Troubleshooting

### No data received
- Check Serial Bridge is connected to Arduino
- Verify device ID matches in ML Bridge
- Ensure baud rate is 9600 in both Arduino and Serial Bridge

### Corrupted data
- ML Bridge automatically throttles messages
- Only sends when prediction changes
- If issues persist, increase throttle in ML Bridge settings

### LED not responding
- Check LED wiring (long leg to pin, short leg to GND via resistor)
- Verify pin number matches your code
- Add `Serial.println()` to debug received data

---

## Tips

✅ **Start with CSV** if you're new to Arduino  
✅ **Use JSON** for production projects  
✅ **Add debug prints** to see what data you're receiving  
✅ **Test manually** in Serial Bridge before connecting ML Bridge  

## Next Steps

- Try controlling a servo motor
- Add multiple LEDs for different classes
- Build a gesture-controlled robot
- Create a sound-reactive light show

Happy making! 🚀
