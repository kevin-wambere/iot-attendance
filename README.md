# iot-attendance
#include <Servo.h>
#include <SPI.h>
#include <MFRC522.h>
#include <DHT.h>

#define SS_PIN 53
#define RST_PIN 5
MFRC522 rfid(SS_PIN, RST_PIN);

Servo doorServo;

#define DHTPIN 7
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

#define buzzer 6
#define ledGreen 3
#define ledRed 4

String authorizedUID = "A1B2C3D4";

void setup() {
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();

  doorServo.attach(10);
  doorServo.write(0);

  dht.begin();

  pinMode(buzzer, OUTPUT);
  pinMode(ledGreen, OUTPUT);
  pinMode(ledRed, OUTPUT);

  digitalWrite(buzzer, LOW);
  digitalWrite(ledGreen, LOW);
  digitalWrite(ledRed, LOW);
}

void loop() {

  // Default OFF state
  digitalWrite(buzzer, LOW);
  digitalWrite(ledGreen, LOW);
  digitalWrite(ledRed, LOW);

  if (rfid.PICC_IsNewCardPresent() && rfid.PICC_ReadCardSerial()) {

    String uid = "";
    for (byte i = 0; i < rfid.uid.size; i++) {
      uid += String(rfid.uid.uidByte[i], HEX);
    }

    uid.toUpperCase();

    if (uid == authorizedUID) {
      digitalWrite(ledGreen, HIGH);
      doorServo.write(90);
      delay(3000);
      doorServo.write(0);
    } else {
      digitalWrite(ledRed, HIGH);
      digitalWrite(buzzer, HIGH);
      delay(2000);
    }

    rfid.PICC_HaltA();
  }

  float temp = dht.readTemperature();
  float hum = dht.readHumidity();

  Serial.print("Temp: ");
  Serial.print(temp);
  Serial.print(" C | Hum: ");
  Serial.print(hum);
  Serial.println(" %");

  delay(500);
}
