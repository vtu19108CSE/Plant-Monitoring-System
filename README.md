# Plant-Monitoring-System
The primary aim of the Plant Monitoring System (PMS) project is to develop and implement an advanced technology-driven solution for precision agriculture. The project seeks to address the challenges faced by modern agriculture, focusing on the accurate monitoring and management of soil conditions to optimize crop yield.


Arduino Code:

#include "DHT.h"
#define DHTPIN 13
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

int SoilMoistSensor = 11;
int SoilMoistState;

const int TrigPin = A2;
const int EchoPin = A1;

long period, interval;

String readstringdata = "";

float temp = 0.0;

#define relay 12

//#define serial

void setup()
{
  Serial.begin(9600);//sets the baud rate

  pinMode(SoilMoistSensor, INPUT);

  pinMode(TrigPin,OUTPUT);
  pinMode(EchoPin,INPUT);

  pinMode(relay, OUTPUT);
    digitalWrite(relay, HIGH);
 
  dht.begin();
}

void loop()
{
  readstringdata = "";
 
  delay(500);
 
  /*****Ultrasonic Sensor*****/
  digitalWrite(TrigPin,LOW);
  delayMicroseconds(2);
  digitalWrite(TrigPin,HIGH);
  delayMicroseconds(10);
  digitalWrite(TrigPin,LOW);  
  period = pulseIn(EchoPin,HIGH);
  interval = period/58.2;
  //Serial.println(interval);
 
  /*..........Soil Moisture Sensor..........*/
  int SoilMoistState = digitalRead(SoilMoistSensor);
  //Serial.println(SoilMoistState);
  //Serial.println("");
 
  if(SoilMoistState == 1)
  {
    digitalWrite(relay, LOW);
   
    delay(2000);
  }
  else
  {
    digitalWrite(relay, HIGH);

    delay(100);
  }

  delay(500);
 
  /*..........DHT11 TEMPERATURE AND HUMIDITY SENSOR..........*/
  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
  float h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  float t = dht.readTemperature();
  // Read temperature as Fahrenheit (isFahrenheit = true)
  float f = dht.readTemperature(true);

  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t) || isnan(f)) {
    #ifdef serial
    Serial.println("Failed to read from DHT sensor!");
    #endif
    return;
  }

  // Compute heat index in Fahrenheit (the default)
  float hif = dht.computeHeatIndex(f, h);
  // Compute heat index in Celsius (isFahreheit = false)
  float hic = dht.computeHeatIndex(t, h, false);

  #ifdef serial
  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print(" %\t");
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.print(" *C ");
  Serial.print(f);
  Serial.print(" *F\t");
  Serial.print("Heat index: ");
  Serial.print(hic);
  Serial.print(" *C ");
  Serial.print(hif);
  Serial.println(" *F");
  #endif
 
  readstringdata += String(h);
  readstringdata += String(",");
  readstringdata += String(t);
  readstringdata += String(",");
  readstringdata += String(SoilMoistState);  
  readstringdata += String(",");
  readstringdata += String(interval);    
  readstringdata += String('#');  
  Serial.println(readstringdata);
  delay(500);
 
  #ifdef serial
  Serial.println("******************************************************");
  Serial.println("         ");
  #endif
 
  readstringdata = "";
 
  delay(5000);
}

ESP8266 NodeMCU Code:

#include <Arduino.h>
#if defined(ESP32)
#include <WiFi.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#endif

#include <WiFiClient.h>  //Client wifi connection library

#include <ThingSpeak.h>  //ThingSpeak Cloud library

#define WIFI_SSID "TP-Link_8E98"
#define WIFI_PASSWORD "86427920"

WiFiClient client;  //client configuration

unsigned long myChannelNumber = 2391971;  //Thingspeak channel number
const char * myWriteAPIKey = "X9YFUDZGEBWJ3Q5Z";  //Thingspeak Write API key

String readstring = "";

String h, t;
String soilmoist;
String waterlevel;

int ind1; // , locations
int ind2;
int ind3;
int ind4;

void setup()
{
  Serial.begin(9600);
  Serial.println();

  Serial.print("Connecting to AP");

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(200);
  }

  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  ThingSpeak.begin(client);
}

void loop()
{
  readstring = "";  //Reset the variable
 
  while (Serial.available())
  {  //Check if there is an available byte to read
  delay(10); //Delay added to make thing stable
  char c = Serial.read(); //Conduct a serial read
  if (c == '#') {break;} //Exit the loop when the # is detected after the word
  readstring += c; //build the string
  }

  if (readstring.length() > 0)
  {
    Serial.println(readstring);

    ind1  = readstring.indexOf(',');
    h     = readstring.substring(0, ind1);
    ind2  = readstring.indexOf(',', ind1+1);//finds location of second ,
    t     = readstring.substring(ind1+1, ind2);
    ind3  = readstring.indexOf(',',ind2+1);
    soilmoist   = readstring.substring(ind2+1, ind3);
    ind4  = readstring.indexOf(',',ind3+1);
    waterlevel   = readstring.substring(ind3+1);//captures remain part of data after last ,
   
    Serial.print("Humidity: ");
    Serial.println(h);
    Serial.print("Temperature: ");
    Serial.println(t);    
    Serial.print("Soil Moisture: ");
    Serial.println(soilmoist);      
    Serial.print("Water Level: ");
    Serial.println(waterlevel);  
     
    ThingSpeak.setField(1, h);
    ThingSpeak.setField(2, t);
    ThingSpeak.setField(3, soilmoist);
    ThingSpeak.setField(4, waterlevel);
       
    ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);  
    delay(1000);
  }
}


To run the project access links are:
ThingSpeak Cloud Account:
https://thingspeak.com/login

Username: minorproject56@gmail.com
Password: Minorproject@12

Mobile Hotspot:
Username: TP-Link_8E98
Password: 86427920
