---
title: "[Prototype] Smart Greenhouse"
date: 2025-05-05T20:11:08+07:00
lastmod: 2025-05-05T20:11:08+07:00
author: ["Zuharman"]
categories: 
- IoT
- Hardware
tags: 
- IoT
- Hardware
description: ""
weight: # 1 means pin the article, sort articles according to this number
slug: ""
draft: false # draft or not
showToc: true # show contents
TocOpen: true # open contents automantically
hidemeta: false # hide information (author, create date, etc.)
disableShare: true	# do not show share button
showbreadcrumbs: true # show current path
cover:
    image: "images/greenhouse.png"
    caption: ""
    alt: ""
    relative: false
---

A smart IoT device to help run a hydroponic greenhouse more smoothly. The device comes with sensors that check temperature, humidity, and the level of dissolved solids (TDS) in the water that’s used to feed the plants. If the temperature inside the greenhouse gets too high, the device will automatically turn on a fan to cool it down. All of this runs on its own—no need to constantly check things manually. It’s basically like having an automatic assistant that helps keep the greenhouse environment just right, making things easier and more efficient for the people running it.

Link: https://github.com/zxrmxn/SmartGreenhouse

---

### **Tools**

| **Hardware** | **Description**             |
| ------------ | --------------------------- |
| NodeMCU      | Microcontroller             |
| DHT22        | Temp and Moisture Sensor    |
| TDS Sensor   | Dissolved Solid Water Level |

| **Software** | **Description**                    |
| ------------ | --------------------------------   |
| Arduino IDE  | Integrated Development Environment |
| ThingSpeak   | Devices Data Stream and Store It   |
| Weebly       | Web Builder                        |

---

### **Codes**

#### Include libraries
``` arduino
#include <DHT.h>
#include <WiFi.h>
```

#### Initializing Internet Connection
``` arduino
const char 'ssid' = "WiFi SSID"
const char 'pass' = "WiFi Password"
const char 'server' = "ThingSpeak API"

WiFi.begin (ssid, pass);

while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(.):
}
Serial.println("");
Serial.println("WiFi Connected");
```

#### Define Pin to connect to DHT Sensor
``` arduino
#define DHTPIN 2 // Pin connected to DHT Sensor
```

#### Define DHT type
``` arduino
// Uncomment whatever type you're using!
//#define DHTTYPE DHT11   // DHT 11
#define DHTTYPE DHT22   // DHT 22  (AM2302), AM2321
//#define DHTTYPE DHT21   // DHT 21 (AM2301)
```

#### Initialize pin for relay
``` arduino
const int relay1 = D0; // Pin2
const int relay2 = D1; // Pin3
const int relay3 = D2; // Pin4
const int relay4 = D3; // Pin5

int relayON = LOW;
int relayOFF = HIGH;
```

#### Set relay off when startup
``` arduino
pinMode (relay1, OUTPUT);
pinMode (relay2, OUTPUT);
pinMode (relay3, OUTPUT);
pinMode (relay4, OUTPUT);
digitalWrite (relay1, relayOFF);
digitalWrite (relay2, relayOFF);
digitalWrite (relay3, relayOFF);
digitalWrite (relay4, relayOFF);
```

#### Read data from DHT sensor
``` arduino
void loop() {
    delay (2000);

    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();
    float hix = dht.computeHeatIndex(temperature, humidity, false);

}
```

#### Relay Condition
``` arduino
if (temperature > 30) {
    digitalWrite (relay1, relayON);
    delay(1000);
}
else{
    digitalWrite (relay1, relayOFF);
    delay(1000);
}
```

#### Print DHT22 value
``` arduino
(isnan(h) || isnan(t) || isnan(f)) {
Serial.println(F("Failed to read from DHT sensor!"));
return;

Serial.print(F("Humidity: "));
Serial.print(humidity);
Serial.print(F("%  Temperature: "));
Serial.print(temperature);
Serial.print(F("°C "));
Serial.print(f);
Serial.print(F("°F  Heat index: "));
Serial.print(hix);
Serial.print(F("°C "));
}
```

#### Define TDS Sensor
``` arduino
#define TdsSensorPin A0
#define VREF 5.0 // analog reference voltage(Volt) of the ADC
#define SCOUNT 30 // sum of sample point
```

#### Create buffer
``` arduino
int analogBuffer[SCOUNT]; // store the analog value in the array, read from ADC
int analogBufferTemp[SCOUNT]; // a temporary copy of analogBuffer, used for processing.
int analogBufferIndex = 0,copyIndex = 0;
float averageVoltage = 0,tdsValue = 0,temperature = 25;
// averageVoltage: Stores the filtered average voltage value from the sensor (in volts).
// tdsValue: The final computed Total Dissolved Solids value (in ppm).
// temperature: Used for temperature compensation of the TDS reading. It's fixed at 25°C by default (you can replace it with a real-time temperature sensor if available).
```

#### Analog Sampling and Data Processing
``` arduino
void setup()
{
    Serial.begin(115200);
    pinMode(TdsSensorPin,INPUT);
}

void loop()
{
   static unsigned long analogSampleTimepoint = millis();
   if(millis()-analogSampleTimepoint > 40U) //every 40 milliseconds,read the analog value from the ADC
   {
     analogSampleTimepoint = millis();
     analogBuffer[analogBufferIndex] = analogRead(TdsSensorPin); //read the analog value and store into the buffer
     analogBufferIndex++;
     if(analogBufferIndex == SCOUNT) 
         analogBufferIndex = 0;
   }   
   static unsigned long printTimepoint = millis();
   if(millis()-printTimepoint > 800U)
   {
      printTimepoint = millis();
      for(copyIndex=0;copyIndex<SCOUNT;copyIndex++)
        analogBufferTemp[copyIndex]= analogBuffer[copyIndex];
      averageVoltage = getMedianNum(analogBufferTemp,SCOUNT) * (float)VREF / 1024.0; // read the analog value more stable by the median filtering algorithm, and convert to voltage value
      float compensationCoefficient=1.0+0.02*(temperature-25.0); //temperature compensation formula: fFinalResult(25^C) = fFinalResult(current)/(1.0+0.02*(fTP-25.0));
      float compensationVolatge=averageVoltage/compensationCoefficient; //temperature compensation
      tdsValue=(133.42*compensationVolatge*compensationVolatge*compensationVolatge - 255.86*compensationVolatge*compensationVolatge + 857.39*compensationVolatge)*0.5; //convert voltage value to tds value
      //Serial.print("voltage:");
      //Serial.print(averageVoltage,2);
      //Serial.print("V   ");
      Serial.print("TDS Value:");
      Serial.print(tdsValue,0);
      Serial.println("ppm");
   }
}
```

#### Calculate the median using bubble sort
``` arduino
int getMedianNum(int bArray[], int iFilterLen) 
{
      int bTab[iFilterLen];
      for (byte i = 0; i<iFilterLen; i++)
      bTab[i] = bArray[i];
      int i, j, bTemp;
      for (j = 0; j < iFilterLen - 1; j++) 
      {
      for (i = 0; i < iFilterLen - j - 1; i++) 
          {
        if (bTab[i] > bTab[i + 1]) 
            {
        bTemp = bTab[i];
            bTab[i] = bTab[i + 1];
        bTab[i + 1] = bTemp;
         }
      }
      }
      if ((iFilterLen & 1) > 0)
    bTemp = bTab[(iFilterLen - 1) / 2];
      else
    bTemp = (bTab[iFilterLen / 2] + bTab[iFilterLen / 2 - 1]) / 2;
      return bTemp;
}
```

---

### References
 - DHT Sensor Library: https://github.com/adafruit/DHT-sensor-library
 - Adafruit Unified Sensor Lib: https://github.com/adafruit/Adafruit_Sensor
 - TDS Sensor: https://wiki.dfrobot.com/Gravity__Analog_TDS_Sensor___Meter_For_Arduino_SKU__SEN0244