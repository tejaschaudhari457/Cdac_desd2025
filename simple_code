//simple code bare-metal approach 


#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <TinyGPS++.h>
#include <WiFi.h>
#include <HTTPClient.h>

// Wi-Fi credentials
const char* ssid = "one";
const char* password = "11111111";

// ThingSpeak API details
const char* server = "http://api.thingspeak.com/update";
const char* apiKey = "LVGMOI4EKKHA4CIO";

// GSM module pins and initialization
#define GSM_TX_PIN 26
#define GSM_RX_PIN 27
HardwareSerial gsmSerial(2);  // Use UART2 for GSM communication

// MPU6050 sensor object
Adafruit_MPU6050 mpu;

// Threshold for accident detection
const float ACCEL_THRESHOLD = 15.0;
const float GYRO_THRESHOLD = 150.0;

// Vibration sensor
#define VIBRATION_SENSOR_DO 13
#define VIBRATION_SENSOR_AO 32
const int ACCIDENT_THRESHOLD = 3000;

// TinyGPS++ and GPS setup
TinyGPSPlus gps;
HardwareSerial gpsSerial(1);  // Use UART1 for GPS
#define GPS_RX_PIN 16
#define GPS_TX_PIN 17

bool accidentHappened = false;

void setup() {
  Serial.begin(115200);

  // Initialize I2C for MPU6050
  Wire.begin();

  // Initialize MPU6050
  if (!mpu.begin()) {
    Serial.println("MPU6050 not found!");
    while (1) { delay(10); }
  }
  mpu.setAccelerometerRange(MPU6050_RANGE_16_G);
  mpu.setGyroRange(MPU6050_RANGE_500_DEG);
  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);

  // Initialize GPS
  gpsSerial.begin(9600, SERIAL_8N1, GPS_RX_PIN, GPS_TX_PIN);
  Serial.println("GPS initialized.");

  // Initialize GSM
  gsmSerial.begin(9600, SERIAL_8N1, GSM_RX_PIN, GSM_TX_PIN);
  delay(2000);

  // Connect to Wi-Fi
  connectToWiFi();
  
  // Vibration sensor setup
  pinMode(VIBRATION_SENSOR_DO, INPUT);
}

void loop() {
  if (!accidentHappened) {
    if (accidentDetected()) {
      accidentHappened = true;

      if (gps.location.isUpdated()) {
        float latitude = gps.location.lat();
        float longitude = gps.location.lng();

        Serial.print("Latitude: ");
        Serial.println(latitude, 6);
        Serial.print("Longitude: ");
        Serial.println(longitude, 6);

        sendToThingSpeak(latitude, longitude);
        sendSMS(latitude, longitude);
      } else {
        Serial.println("Using default GPS location: 12.8421, 77.6631");
      }
    }
  }

  // Read GPS data
  while (gpsSerial.available() > 0) {
    gps.encode(gpsSerial.read());
  }

  delay(500);
}

bool accidentDetected() {
  sensors_event_t a, g, temp;
  mpu.getEvent(&a, &g, &temp);

  float ax = a.acceleration.x;
  float ay = a.acceleration.y;
  float az = a.acceleration.z;

  float totalAccel = sqrt(ax * ax + ay * ay + az * az);

  float gx = g.gyro.x;
  float gy = g.gyro.y;
  float gz = g.gyro.z;

  int analogValue = analogRead(VIBRATION_SENSOR_AO);

  Serial.print("Vibration Intensity: ");
  Serial.println(analogValue);
  Serial.print("Total Acceleration: ");
  Serial.println(totalAccel);

  return (totalAccel > ACCEL_THRESHOLD || abs(gx) > GYRO_THRESHOLD ||
          abs(gy) > GYRO_THRESHOLD || abs(gz) > GYRO_THRESHOLD || 
          analogValue > ACCIDENT_THRESHOLD);
}

void connectToWiFi() {
  Serial.print("Connecting to Wi-Fi...");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected to Wi-Fi.");
}

void sendToThingSpeak(float latitude, float longitude) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = String(server) + "?api_key=" + apiKey + 
                 "&field1=" + String(latitude, 6) + 
                 "&field2=" + String(longitude, 6);

    http.begin(url);
    int httpResponseCode = http.GET();

    if (httpResponseCode > 0) {
      Serial.println("Data sent successfully.");
    } else {
      Serial.print("Error sending data: ");
      Serial.println(httpResponseCode);
    }

    http.end();
  } else {
    Serial.println("Wi-Fi Disconnected.");
  }
}

void sendSMS(float latitude, float longitude) {
  String googleMapsLink = "https://www.google.com/maps?q=" + String(latitude, 6) + "," + String(longitude, 6);
  String message = "Accident detected! Check location: " + googleMapsLink;

  gsmSerial.print("AT+CMGS=\"7558358463\"\r");  // Replace with your phone number
  delay(100);
  gsmSerial.print(message);
  delay(100);
  gsmSerial.write(26);  // ASCII for Ctrl+Z to send the SMS
  Serial.println("SMS sent successfully.");
}
