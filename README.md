# Smart-Home-Automation-using-IoT-and-ESP32
This repository consist of code for Smart Home Automation using IoT and ESP32 Project.
/************************************************************
      SmartShield IoT Home Automation & Security System
 ************************************************************/

#define BLYNK_TEMPLATE_ID "TMPL31BUZIMu9"
#define BLYNK_TEMPLATE_NAME "SmartShield"
#define BLYNK_AUTH_TOKEN "kG2E2N0OZbVo2TBeFbJQOX09QTXxTk_Y"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>

/**************** WIFI DETAILS ****************/

char ssid[] = "samsung";
char pass[] = "9812345";

/**************** PIN DEFINITIONS ****************/

#define RELAY_PIN      23
#define IR_SENSOR_PIN  34
#define BUZZER_PIN     19
#define RED_LED_PIN    4
#define GREEN_LED_PIN  2

/**************** VARIABLES ****************/

BlynkTimer timer;

bool alertSent = false;

/************************************************************
                    RELAY CONTROL
 ************************************************************/

BLYNK_WRITE(V0)
{
  int relayState = param.asInt();

  digitalWrite(RELAY_PIN, relayState);

  Serial.print("Relay State: ");
  Serial.println(relayState);
}

/************************************************************
                 SECURITY SYSTEM FUNCTION
 ************************************************************/

void checkSecurity()
{
  int sensorValue = digitalRead(IR_SENSOR_PIN);

  Serial.print("IR Sensor Value: ");
  Serial.println(sensorValue);

  /********************************************************
      MOST IR SENSORS:
      HIGH = Object Detected
      LOW  = No Object
  ********************************************************/

  if(sensorValue == HIGH)
  {
    digitalWrite(BUZZER_PIN, HIGH);
    digitalWrite(RED_LED_PIN, HIGH);

    Blynk.virtualWrite(V1, 255);

    if(!alertSent)
    {
      Serial.println("INTRUDER DETECTED!");

      Blynk.logEvent("intruder",
      "🚨 Intruder Detected in SmartShield System!");

      alertSent = true;
    }
  }
  else
  {
    digitalWrite(BUZZER_PIN, LOW);
    digitalWrite(RED_LED_PIN, LOW);

    Blynk.virtualWrite(V1, 0);

    alertSent = false;
  }
}

/************************************************************
                    BLYNK STATUS
 ************************************************************/

void checkConnection()
{
  if(Blynk.connected())
  {
    digitalWrite(GREEN_LED_PIN, HIGH);
  }
  else
  {
    digitalWrite(GREEN_LED_PIN, LOW);
  }
}

/************************************************************
                          SETUP
 ************************************************************/

void setup()
{
  Serial.begin(115200);

  pinMode(RELAY_PIN, OUTPUT);
  pinMode(IR_SENSOR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(RED_LED_PIN, OUTPUT);
  pinMode(GREEN_LED_PIN, OUTPUT);

  digitalWrite(RELAY_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);
  digitalWrite(RED_LED_PIN, LOW);
  digitalWrite(GREEN_LED_PIN, LOW);

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  timer.setInterval(300L, checkSecurity);
  timer.setInterval(2000L, checkConnection);

  Serial.println("=================================");
  Serial.println(" SmartShield System Started ");
  Serial.println("=================================");
}

/************************************************************
                           LOOP
 ************************************************************/

void loop()
{
  Blynk.run();
  timer.run();
}
