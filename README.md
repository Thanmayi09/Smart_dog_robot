#include <ESP8266WiFi.h>
#include <Firebase_ESP_Client.h>
#include "addons/TokenHelper.h"
#include "addons/RTDBHelper.h"
#include <Servo.h>

#define SERVO_PIN_1 D1 // Connect servo 1 to Digital Pin D1
#define SERVO_PIN_2 D2 // Connect servo 2 to Digital Pin D2
#define SERVO_PIN_3 D3 // Connect servo 3 to Digital Pin D3
#define SERVO_PIN_4 D4 // Connect servo 4 to Digital Pin D4
String a="2";
String intValue;
int angle;

#define WIFI_SSID "123456789"
#define WIFI_PASSWORD "123456789"
#define API_KEY "AIzaSyC0gPSHesz3RxIsbFM48OkKK_zCBhfbtmc"
#define DATABASE_URL "https://test-26075-default-rtdb.firebaseio.com/"

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;
unsigned long sendDataPrevMillis = 0;
bool signupOK = false;

Servo servo1;
Servo servo2;
Servo servo3;
Servo servo4;

void setup() {
  Serial.begin(115200);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;

  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("Firebase sign-up successful");
    signupOK = true;
  } else {
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  config.token_status_callback = tokenStatusCallback; // see addons/TokenHelper.h
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
  
  servo1.attach(SERVO_PIN_1);
  servo2.attach(SERVO_PIN_2);
  servo3.attach(SERVO_PIN_3);
  servo4.attach(SERVO_PIN_4);


  Serial.println("Servo control initialized");
}

void loop() {
  // Sweep both servos from 0 to 180 degrees
  for (int angle = 0; angle <= 180; angle += 50) {
    //servo1.write(angle);
    //servo2.write(angle);
    //servo3.write(angle);
    //servo4.write(angle);


    delay(100); // Adjust the delay for a smoother motion
  }

  delay(100); // Wait for 1 second at the end of the sweep

  // Sweep both servos back from 180 to 0 degrees
  for (int angle = 180; angle >= 0; angle -= 50) {
   // servo1.write(angle);
    //servo2.write(angle);
    //servo3.write(angle);
    //servo4.write(angle);

    delay(100); // Adjust the delay for a smoother motion
  }

if (Firebase.ready() && signupOK && (millis() - sendDataPrevMillis > 1000 || sendDataPrevMillis == 0)) {
    sendDataPrevMillis = millis();

        if (Firebase.RTDB.getString(&fbdo, "/main/servo")) 
        {
          intValue = fbdo.stringData();
          String mySubString = intValue.substring(2, 3);
          Serial.println(intValue);
          Serial.println(mySubString);
          if (mySubString == "1"){
            for (int angle = 0; angle <= 90; angle += 90){
              
              servo1.write(angle);
              servo2.write(angle);
              servo3.write(angle);
              servo4.write(angle);
            
              Serial.println("Shield ON");
              delay(100);
              }  
            }
          else if (mySubString == "0"){
            for (int angle = 90; angle >= 0; angle -= 90){
              
              servo1.write(0);
              servo2.write(0);
              servo3.write(0);
              servo4.write(0);
              Serial.println("Shield OFF");
              delay(100);  
              }  
        delay(100);
        }
    else {
        Serial.println(fbdo.errorReason());
      }

  delay(100); // Wait for 1 second at the end of the sweep
  }
}
}
