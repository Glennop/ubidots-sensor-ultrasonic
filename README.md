#include "UbidotsESPMQTT.h"

#define TOKEN "BBUS-6SNn4yhzukMcclT9jaNlqmaubmdpQH"   // Ganti dengan Ubidots TOKEN kamu
#define WIFINAME "ADNAN"            // SSID WiFi
#define WIFIPASS "15071970"         // Password WiFi

#define DEVICE_LABEL "esp32_ultrasonic" // Nama device di Ubidots
#define VARIABLE_LABEL "distance_cm"    // Variable di Ubidots

const int trigPin = 33;
const int echoPin = 32;

Ubidots client(TOKEN);

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (unsigned int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

float readDistanceCM() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000); // timeout 30 ms
  float distance = duration * 0.034 / 2; // cm
  return distance;
}

unsigned long lastPublish = 0;

void setup() {
  Serial.begin(115200);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  client.wifiConnection(WIFINAME, WIFIPASS);
  client.begin(callback);
}

void loop() {
  if (!client.connected()) {
    client.reconnect();
  }
  client.loop();

  // Publish tiap 2 detik
  if (millis() - lastPublish > 2000) {
    lastPublish = millis();

    float jarak = readDistanceCM();

    Serial.print("Distance: ");
    Serial.print(jarak);
    Serial.println(" cm");

    client.add(VARIABLE_LABEL, jarak);   // tambah variable
    client.ubidotsPublish(DEVICE_LABEL); // kirim ke Ubidots
  }
}
