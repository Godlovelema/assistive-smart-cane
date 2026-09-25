# assistive-smart-cane
An IoT-based smart cane project designed to assist visually impaired individuals navigate safely using ultrasonic sensors.
#include <SoftwareSerial.h>

// SIM800L Pins (GSM/GPS)
SoftwareSerial gsmSerial(2, 3); // RX=2, TX=3 (Unganisha TX ya SIM800L kwenye pin 2, RX kwenye pin 3)

// Ultrasonic Sensor ya Juu (Vizuizi vya juu)
const int trigJuu = 4;
const int echoJuu = 5;

// Ultrasonic Sensor ya Chini (Vizuizi vya chini/Mashimo)
const int trigChini = 6;
const int echoChini = 7;

// Water Sensor na Buzzer
const int waterSensorPin = A0; 
const int buzzerPin = 8;
const int buttonEmergency = 9; // Kitufe cha dharura

// Namba ya simu ya ndugu/mlezi (Badilisha weka ya kwako hapa)
String nambaYaSimu = "+2557XXXXXXXX"; 

void setup() {
  Serial.begin(9600);
  gsmSerial.begin(9600);
  
  pinMode(trigJuu, OUTPUT);   pinMode(echoJuu, INPUT);
  pinMode(trigChini, OUTPUT); pinMode(echoChini, INPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(buttonEmergency, INPUT_PULLUP); // Inatumia internal resistor

  Serial.println("Mfumo umeanza...");
  delay(1000);
}

void loop() {
  // 1. Kupima Vikwazo
  long umbaliJuu = pimaUmbali(trigJuu, echoJuu);
  long umbaliChini = pimaUmbali(trigChini, echoChini);
  int kiwangoMaji = analogRead(waterSensorPin);

  // 2. Kuchakata Taarifa za Vikwazo (Maji na Vizuizi chini ya 50cm)
  if (umbaliJuu < 50 && umbaliJuu > 0) {
    pigaKingora(100, 1); // Vikwazo vya juu - mlio wa haraka
    Serial.println("Kikwazo cha JUU kipo karibu!");
  }
  else if (umbaliChini < 50 && umbaliChini > 0) {
    pigaKingora(300, 1); // Vikwazo vya chini - mlio wa wastani
    Serial.println("Kikwazo cha CHINI/SHIMO kipo karibu!");
  }
  else if (kiwangoMaji > 200) { // Thamani ikizidi 200 kuna maji
    pigaKingora(50, 3); // Maji - milio mitatu mifupi ya haraka
    Serial.println("Kuna MAJI mbele yako!");
  }

  // 3. Kitufe cha Dharura (Emergency Push Button)
  if (digitalWrite(buttonEmergency) == LOW) { 
    Serial.println("Kitufe cha Dharura kimebonyezwa!");
    pigaSimuNaSms();
    delay(5000); // Zuia kutuma sms nyingi mfululizo
  }

  delay(200);
}

// Function ya Kupima Umbali
long pimaUmbali(int trig, int echo) {
  digitalWrite(trig, LOW); delayMicroseconds(2);
  digitalWrite(trig, HIGH); delayMicroseconds(10);
  digitalWrite(trig, LOW);
  long muda = pulseIn(echo, HIGH, 30000); // Timeout ya 30ms usizuie mfumo
  return muda * 0.034 / 2;
}

// Function ya Kupiga King'ora
void pigaKingora(int muda, int marudio) {
  for(int i=0; i<marudio; i++){
    digitalWrite(buzzerPin, HIGH); delay(muda);
    digitalWrite(buzzerPin, LOW);  delay(muda);
  }
}

// Function ya Kutuma SMS na Kupiga Simu ya Dharura
void pigaSimuNaSms() {
  // Kupiga Simu ya Kawaida ya Sauti
  gsmSerial.println("ATD" + nambaYaSimu + ";"); 
  delay(100);
  
  // Kutuma SMS (Ujumbe wa Dharura)
  gsmSerial.println("AT+CMGF=1"); // Weka format ya SMS kuwa text
  delay(500);
  gsmSerial.println("AT+CMGS=\"" + nambaYaSimu + "\"");
  delay(500);
  // Hapa unaweza kuweka link ya Google Maps kama unatumia GPS module, au ujumbe wa kawaida
  gsmSerial.print("MSAADA! Fimbo Janja imegundua dharura. Tafadhali nisaidie."); 
  delay(500);
  gsmSerial.write(26); // Kufunga ujumbe na kuutuma (Ctrl+Z)
  delay(500);
}
