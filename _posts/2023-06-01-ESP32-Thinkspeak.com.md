---
layout: post
title: ESP32 IoT Thinkspeak.com
author: [Sean Su]
category: [Lecture]
tags: [jekyll, ai]
---

## IMU 實作
This project use DHT11 as a sensor to detech the Temperature and Hunidety. And the data upload to ThinkSpeak.

## IMU度感測器
### 應用功能說明
1. Use DHT11 as a sensor.
2. Use ESP32 as a MCU.
3. The data upload to ThinkSpeak.
4. ESP32 每個週期會從 DHT11 接收溫濕度數值，轉成一串網址並執行。



**所需相關技術：**
1. HTML
2. MCU


### 系統方塊圖
![](https://github.com/sean195114/MCU-project/blob/main/images/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202023-06-01%20221142.png?raw=true)


## Code: 
```C++
#include <WiFi.h> 
#include "DHT.h"

#define DHTPIN 23     // NodeMCU pin D6 connected to DHT11 pin Data
DHT dht(DHTPIN, DHT11, 15);

const char* ssid     = "Beta";
const char* password = "";


const char* host = "api.thingspeak.com";
const char* thingspeak_key = "";

void turnOff(int pin) {
  pinMode(pin, OUTPUT);
  digitalWrite(pin, 1);
}

void setup() {
  Serial.begin(115200);

  // disable all output to save power
  turnOff(0);
  turnOff(2);
  turnOff(4);
  turnOff(5);
  turnOff(12);
  turnOff(13);
  turnOff(14);
  turnOff(15);

  dht.begin();
  delay(10);
  

  // We start by connecting to a WiFi network

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");  
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

int value = 0;

void loop() {
  delay(100);
  ++value;

  Serial.print("connecting to ");
  Serial.println(host);
  
  // Use WiFiClient class to create TCP connections
  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
    Serial.println("connection failed");
    return;
  }

  String temp = String(dht.readTemperature());
  String humidity = String(dht.readHumidity());
  String url = "/update?key=";
  url += thingspeak_key;
  url += "&field1=";
  url += temp;
  url += "&field2=";
  url += humidity;
  
  Serial.print("Requesting URL: ");
  Serial.println(url);
  
  // This will send the request to the server
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" + 
               "Connection: close\r\n\r\n");
  delay(10);
  
  // Read all the lines of the reply from server and print them to Serial
  while(client.available()){
    String line = client.readStringUntil('\r');
    Serial.print(line);
  }
  
  Serial.println();
  Serial.println("closing connection. going to sleep...");
  delay(1000);
  // go to deepsleep for 1 minutes
  //system_deep_sleep_set_option(0);
  //system_deep_sleep(1 * 60 * 1000000);
  delay(1*10*1000);
}
```

## Project Review:
![](https://github.com/sean195114/MCU-project/blob/main/images/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202023-06-01%20221212.png?raw=true)
<br>
<br>

This site was last updated {{ site.time | date: "%B %d, %Y" }}.

