# Arduino UNO + Processing — Basic Serial Communication

## Overview

This guide covers how to set up two-way communication between a **Processing sketch** and an **Arduino UNO** over USB serial. The Processing window sends commands to the Arduino, which controls the built-in LED on pin 13 and replies with its status.

```
Processing sketch  ──(sends 'H'/'L' via serial)──►  Arduino UNO
                   ◄──(replies "LED ON"/"LED OFF")──  (controls LED pin 13)
```

---

## Arduino Sketch

Upload this first via the Arduino IDE:

```cpp
// Arduino UNO - Serial LED control
// Upload this to the board before running the Processing sketch

const int LED_PIN = 13;  // Built-in LED

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(9600);
  Serial.println("Arduino ready");
}

void loop() {
  if (Serial.available() > 0) {
    char cmd = Serial.read();

    if (cmd == 'H') {
      digitalWrite(LED_PIN, HIGH);
      Serial.println("LED ON");
    } else if (cmd == 'L') {
      digitalWrite(LED_PIN, LOW);
      Serial.println("LED OFF");
    }
  }
}
```

---

## Processing Sketch

Open this in the Processing IDE:

```java
// Processing - Arduino serial communication
// Make sure to set the correct serial port below

import processing.serial.*;

Serial arduinoPort;
String receivedData = "";
boolean ledOn = false;

void setup() {
  size(400, 300);

  // List available ports in the console — find your Arduino's port
  printArray(Serial.list());

  // Change index [0] to match your Arduino port
  // Mac/Linux: looks like "/dev/cu.usbmodem..." or "/dev/ttyUSB0"
  // Windows:   looks like "COM3", "COM4", etc.
  String portName = Serial.list()[0];
  arduinoPort = new Serial(this, portName, 9600);
  arduinoPort.bufferUntil('\n');
}

void draw() {
  background(30);

  // Status circle
  fill(ledOn ? color(80, 220, 120) : color(100));
  noStroke();
  ellipse(200, 120, 80, 80);

  // Labels
  fill(220);
  textAlign(CENTER);
  textSize(16);
  text(ledOn ? "LED ON" : "LED OFF", 200, 200);

  textSize(13);
  fill(160);
  text("Click to toggle  |  Press SPACE to toggle", 200, 240);

  // Show last received message
  fill(120);
  textSize(12);
  text("Arduino says: " + receivedData.trim(), 200, 270);
}

// Toggle on mouse click
void mousePressed() {
  toggleLED();
}

// Toggle on spacebar
void keyPressed() {
  if (key == ' ') toggleLED();
}

void toggleLED() {
  ledOn = !ledOn;
  arduinoPort.write(ledOn ? 'H' : 'L');
}

// Read incoming data from Arduino
void serialEvent(Serial port) {
  receivedData = port.readStringUntil('\n');
}
```

---

## How to Run

1. **Upload the Arduino sketch** first using the Arduino IDE, then close the IDE (it holds the serial port).
2. Open the Processing sketch and **run it once** — it will print all available serial ports in the console.
3. **Check the console output** and update `Serial.list()[0]` to the index of your Arduino's port (e.g. `Serial.list()[1]` if it's the second one listed).
4. Run again — clicking the window or pressing `Space` will toggle the built-in LED on pin 13 and show the Arduino's reply.

> **Tip:** If you get a port error, the most common cause is the Arduino IDE still being open and holding the port. Close it first.

---

## Communication Protocol

| Direction | Value | Meaning |
|---|---|---|
| Processing → Arduino | `'H'` | Turn LED on |
| Processing → Arduino | `'L'` | Turn LED off |
| Arduino → Processing | `"LED ON\n"` | Confirmation LED is on |
| Arduino → Processing | `"LED OFF\n"` | Confirmation LED is off |

---

## Baud Rate

Both sketches use **9600 baud**. Make sure the value matches in both files if you change it.
