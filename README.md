-->Assistive Glove using ESP32

-->Overview

This project converts hand gestures into predefined messages using flex sensors and ESP32.

-->Features

* 5 flex sensor integration
* Gesture → message conversion
* Wireless communication using Blynk

-->Components Used

* ESP32
* Flex Sensors (5)
* Resistors
* WiFi (IoT)

-->Code

#define BLYNK_TEMPLATE_ID "TMPL30ZJUwCld"
#define BLYNK_TEMPLATE_NAME "ASSISTIVE GLOVE"
#define BLYNK_AUTH_TOKEN "K2_ROh54CDwUOIKgo60s-IBgofnxjCpa"

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>
#include <PulseSensorPlayground.h>

// --- WiFi Credentials ---
char ssid[] = "hi";
char pass[] = "12345678";

// --- Pin Definitions ---
#define DHTPIN 13
#define DHTTYPE DHT11
#define PULSE_PIN 26
#define BUZZER_PIN 14
#define THUMB_PIN 34
#define INDEX_PIN 35
#define MIDDLE_PIN 32
#define RING_PIN 33
#define PINKY_PIN 25

// --- Blynk Virtual Pins ---
#define VPIN_TEMP V0
#define VPIN_HUMID V1
#define VPIN_PULSE V2
#define VPIN_BUZZER_SWITCH V3
#define VPIN_THUMB V4
#define VPIN_INDEX V5
#define VPIN_MIDDLE V6
#define VPIN_RING V7
#define VPIN_PINKY V8
#define VPIN_MESSAGE V9

// --- Sensor Objects ---
DHT dht(DHTPIN, DHTTYPE);
PulseSensorPlayground pulseSensor;

// --- Variables ---
int buzzerState = 0; // Controlled from Blynk
String lastMessage = "";
int highPulseLimit = 120; // bpm threshold

void setup() {
  Serial.begin(115200);
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
  dht.begin();

  // Initialize Pulse Sensor
  pulseSensor.analogInput(PULSE_PIN);
  pulseSensor.begin();

  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW); // buzzer off by default
}

BLYNK_WRITE(VPIN_BUZZER_SWITCH) {
  buzzerState = param.asInt(); // 1 = ON, 0 = OFF
}

void loop() {
  Blynk.run();

  // --- Flex Sensor Reading ---
  checkFlexGesture();

  // --- Temperature & Humidity ---
  float h = dht.readHumidity();
  float t = dht.readTemperature(true); // Fahrenheit

  if (!isnan(t) && !isnan(h)) {
    Blynk.virtualWrite(VPIN_TEMP, t);
    Blynk.virtualWrite(VPIN_HUMID, h);
  }

  // --- Pulse Reading ---
  int BPM = pulseSensor.getBeatsPerMinute();
  if (pulseSensor.sawStartOfBeat()) {
    Serial.print("❤️ BPM: ");
    Serial.println(BPM);
    Blynk.virtualWrite(VPIN_PULSE, BPM);

    // Warning if pulse > normal
    if (BPM > highPulseLimit) {
      Blynk.virtualWrite(VPIN_MESSAGE, "⚠️ High Pulse Detected! Please check the patient.");
    }
  }

  // --- Buzzer Control (Continuous ON/OFF) ---
  if (buzzerState == 1) {
    digitalWrite(BUZZER_PIN, LOW);  // Switch ON → buzzer ON continuously
  } else {
    digitalWrite(BUZZER_PIN, HIGH);   // Switch OFF → buzzer OFF
  }

  delay(500);
}

void checkFlexGesture() {
  int thumbVal = analogRead(THUMB_PIN);
  int indexVal = analogRead(INDEX_PIN);
  int middleVal = analogRead(MIDDLE_PIN);
  int ringVal = analogRead(RING_PIN);
  int pinkyVal = analogRead(PINKY_PIN);

  String message = "";
 // --- Combination Gestures ---
  if (indexVal > 1500 && middleVal > 1500) {
    message = "🤚 let them go out I want to take rest\n";
  } 
  else if (thumbVal > 1500 && indexVal > 1500) {
    message = "👍 I'm OK\n";
  } 
  else if (thumbVal > 1500 && middleVal > 1500) {
    message = "🤕 I need to take my medicine\n ";
  }
  // --- Single Finger Gestures ---
  else if (thumbVal > 1500) message = "✋ I’m feeling pain, please come\n";
  else if (indexVal > 1500) message = "👉 I need water, please.\n";
  else if (middleVal > 1500) message = "🖐 I am hungry / need food..\n";
  else if (ringVal > 1500) message = "👌 Please call the doctor!.\n";
  else if (pinkyVal > 1500) message = "🤚 I want to use washroom\n";
  else message = "";

  if (message != "" && message != lastMessage) {
    lastMessage = message;
    Serial.println(message);
    Blynk.virtualWrite(VPIN_MESSAGE, message);
  }
}
-->Project Images

<img width="1280" height="720" alt="WhatsApp Image 2026-03-15 at 7 41 24 PM (1)" src="https://github.com/user-attachments/assets/4ba10208-f263-4fb6-97b7-546786139ed7" />

<img width="1280" height="720" alt="WhatsApp Image 2026-03-15 at 7 48 26 PM" src="https://github.com/user-attachments/assets/def9473c-fe3a-4a29-a2ce-b0f2be0ea967" />

<img width="720" height="1280" alt="WhatsApp Image 2026-03-15 at 7 41 24 PM" src="https://github.com/user-attachments/assets/6d6e600d-b873-4553-8417-8410c2ff1d0f" />

-->Presentation


[assistive glove ppt.pptx](https://github.com/user-attachments/files/26864150/assistive.glove.ppt.pptx)


-->Certificate

<img width="1252" height="888" alt="Screenshot 2026-04-19 092610" src="https://github.com/user-attachments/assets/9d7bbc69-c982-4972-807b-09ac767eeab7" />




