#include <Wire.h>
#define BLYNK_PRINT Serial

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>

// Blynk Credentials
#define BLYNK_TEMPLATE_ID "TMPL3HPG_ReGc"
#define BLYNK_TEMPLATE_NAME "Quickstart Template"
#define BLYNK_AUTH_TOKEN "Your_Auth_Token"

// WiFi Credentials
char ssid[] = "Your_WiFi_Name";
char pass[] = "Your_WiFi_Password";

// MPU6050 Address
const int MPU_addr = 0x68;

// Sensor variables
int16_t AcX, AcY, AcZ;
int minVal = 265;
int maxVal = 402;

double x, y, z;

void setup() {
  Wire.begin();

  // Wake up MPU6050
  Wire.beginTransmission(MPU_addr);
  Wire.write(0x6B);
  Wire.write(0);
  Wire.endTransmission(true);

  Serial.begin(9600);

  // Connect to Blynk
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
}

void loop() {
  Blynk.run();

  // Read MPU6050 data
  Wire.beginTransmission(MPU_addr);
  Wire.write(0x3B);
  Wire.endTransmission(false);
  Wire.requestFrom(MPU_addr, 14, true);

  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();

  int xAng = map(AcX, minVal, maxVal, -90, 90);
  int yAng = map(AcY, minVal, maxVal, -90, 90);
  int zAng = map(AcZ, minVal, maxVal, -90, 90);

  // Calculate tilt angles
  x = RAD_TO_DEG * (atan2(-yAng, -zAng) + PI);
  y = RAD_TO_DEG * (atan2(-xAng, -zAng) + PI);
  z = RAD_TO_DEG * (atan2(-yAng, -xAng) + PI);

  Serial.print("AngleX= "); Serial.println(x);
  Serial.print("AngleY= "); Serial.println(y);
  Serial.print("AngleZ= "); Serial.println(z);
  Serial.println("------------------------");

  // Send data to Blynk
  Blynk.virtualWrite(V2, x);
  Blynk.virtualWrite(V3, y);
  Blynk.virtualWrite(V4, z);

  delay(1000);
}

