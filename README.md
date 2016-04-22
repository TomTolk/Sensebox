<img src="https://github.com/sensebox/OER/blob/master/senseBox_edu/images/sensebox_logo_neu.png" width="200"/>

# Sensebox mit Wind-, Temperatur-, Uv- und Feuchtigkeitssensoren
Wie im Titel schon geschrieben enthält diese Sensebox Wind-, Temperatur-, Uv- und Feutigkeitssensoren um diese verschieden Werte zu messen.

## Ziel
Die Sensebox mit dem folgenden Aufbau und dem folgenden Code soll die Temperatur, die UV Stärke die Luftfeuchtigkeit sowie Windrichtung und Windgeschwindigkeit messen.

## Materialien
#### Aus der senseBox:edu
* Genuino UNO
* Kombinierter Temperatur- und Luftfeuchtigkeitssensor (HDC 1008)
* UV-Sensor (VEML6070)
* Wiznet W5500 Ethernet Shield
*  Veschiedene Kabel

#### Zusätzliche Hardware
* Wetter Messeinheit von Watterrott SEN-08942 ( wobei hier nur der Windgeschwindigkeits und Windrichtungssenor verwendet wird.)
* Heißkleber



## Setup Beschreibung
#### Hardwarekonfiguration
Da die Winsensoren nicht direkt an der sensBox seinen können müssen diese mit einem längeren Kabel an der Sensebox angeschlossen werden. Hierbei muss man darauf achten das wenn man Zwischenkabel verwendet auch diese gut ineinander stecken und nicht irgendwo der Stromfluss durchbrochen wurde.
Auch ist daruf zu achten, dass der Windgeschwindkeitssensor wircklich an dem passenden Port angschlossen ist da nur dieser eine Port durch die Intterupt Funktion( siehe Code) verwendet werden kann.

Bei dem Schaltplan wurden nicht die genau so aussenden Sensoren verwendet wie sie in der Sensebox sind da es diese nicht gab.(Auf die Namen der Senoren und deren Ansteck punkte achten).

<img src="https://github.com/Valderag/Sensebox/blob/master/Sensebox_Steckplatine.jpg" width="200"/>


Für die Windsensoren müssen wir noch einen extra Wiederstand dazwischen packen( siehe Bild).

Bild Sensoren.

<img src="https://github.com/Valderag/Sensebox/blob/master/IMG-20160422-WA0002.jpg" width="200"/>

#### Softwaresketch
Bei dieser Implementierung ist das besondere das wir auf die Intterupt Funktion benutzen.
Diese Wird immer Aktiv wenn etwas betimmtes passiert. In unserem Fall wen Wind den Windsensor bewegt.
Um die durch die Windrichtung zurückgegbende Voltzahl in eine Windrichtung(in Grad)umzusetzten benutzen wir hier die swich case Anweisung.
Die Temperatur und die Luftfeuchtigkeit müssen nur von uns ausgelsen da der Sensor schon in der unten verlinkten Bibilotehk ist.

https://github.com/RFgermany/HDC100X_Arduino_Library

``` c
#include <SPI.h>
#include <Ethernet.h>
int pin = 3;
int count = 0;
unsigned long time = 0;
unsigned long timeold = 0;
double speed;
double wind;
#include <Wire.h>
#include <HDC100X.h>
#define I2C_ADDR 0x38
//Integration Time
#define IT_1_2 0x0 //1/2T
#define IT_1   0x1 //1T
#define IT_2   0x2 //2T
#define IT_4   0x3 //4T
HDC100X HDC1(0X43);
#define LED 13
bool state = false;
//SenseBox ID
#define SENSEBOX_ID "5719cb7c7514d05c121e4e5e"
//Sensor IDs
#define SENSOR1_ID "5719cb7c7514d05c121e4e64" // Windgeschwindigkeit 
#define SENSOR2_ID "5719cb7c7514d05c121e4e63" // Windrichtung 
#define TEMPSENSOR_ID "5719cb7c7514d05c121e4e62"
#define SENSOR3_ID "5719cb7c7514d05c121e4e61" // Luftfeuchtigkeit 
#define UVSENSOR_ID "5719cb7c7514d05c121e4e60"
//Ethernet-Parameter
char server[] = "www.opensensemap.org";
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
// Diese IP Adresse nutzen falls DHCP nicht möglich
IPAddress myIP(192, 168, 0, 42);
EthernetClient client;
//Messparameter
int postInterval = 10000; //Uploadintervall in Millisekunden
long oldTime = 0;
void setup()
{
  Serial.begin(9600); 
  Serial.print("Starting network...");
  //Ethernet Verbindung mit DHCP ausführen..
  if (Ethernet.begin(mac) == 0) 
  {
    Serial.println("DHCP failed!");
    //Falls DHCP fehltschlägt, mit manueller IP versuchen
    Ethernet.begin(mac, myIP);
  }
  Serial.println("done!");
  delay(1000);
  Serial.println("Starting loop.");
   pinMode(pin, INPUT);
    attachInterrupt(digitalPinToInterrupt(pin), blink, FALLING);
     HDC1.begin(HDC100X_TEMP_HUMI,HDC100X_14BIT,HDC100X_14BIT,DISABLE);
   while(!Serial); //wait for serial port to connect (needed for Leonardo only)

  Wire.begin();

  Wire.beginTransmission(I2C_ADDR);
  Wire.write((IT_1<<2) | 0x02);
  Wire.endTransmission();
}
void loop()
{
  //Upload der Daten mit konstanter Frequenz
  if (millis() - oldTime >= postInterval)
  {
    oldTime = millis();
    time = millis();
    int val = analogRead(0);
    val = map(val, 0.00, 1023.00, 0, 500);
    analogWrite(9, val);
      if(time > timeold + 1000)
      {
        speed = (0.66)* count;
        timeold = time;
        count = 0;
      }
      switch (val) 
      {
        case 384:
          wind = 0;
        break;
        case 198:
          wind = 22.5;
        break;
        case 225:
          wind = 45.0;
        break;
        case 041:
          wind = 67.5;
        break;
        case 45:
          wind = 90.0;
        break;
        case 32:
          wind = 112.5;
        break;
        case 90:
          wind = 135;
        break;
        case 62:
          wind = 157.5;
        break;
        case 140:
          wind = 180;
        break;
        case 119:
          wind = 202.5;
        break;
        case 308:
          wind = 225;
        break;
        case 293:
          wind = 247.5;
        break;
        case 462:
          wind = 270;
        break;
        case 404:
          wind = 292.5;
        break;
        case 478:
          wind = 315;
        break;
       case 343:
         wind = 337.5;
        break;
      }

        byte msb=0, lsb=0;
  uint16_t uv;

  Wire.requestFrom(I2C_ADDR+1, 1); //MSB
  delay(1);
  if(Wire.available())
    msb = Wire.read();
  Wire.requestFrom(I2C_ADDR+0, 1); //LSB
  delay(1);
  if(Wire.available())
    lsb = Wire.read();
  uv = (msb<<8) *5,65 | lsb;
  double temp = HDC.getTemp();
  double humi = HDC.getHumi();
     postFloatValue(uv, 2, UVSENSOR_ID);
     postFloatValue(speed, 2, SENSOR1_ID);
     postFloatValue(wind, 2, SENSOR2_ID);
     postFloatValue(temp, 2, TEMPSENSOR_ID);
     postFloatValue(humi, 2, SENSOR3_ID);
  }

 }
  void blink() 
   {
    count = count +1;
   }
void postFloatValue(float measurement, int digits, String sensorId)
{ 
  //Float zu String konvertieren
  char obs[10]; 
  dtostrf(measurement, 5, digits, obs);
  //Json erstellen
  String jsonValue = "{\"value\":"; 
  jsonValue += obs; 
  jsonValue += "}";  
  //Mit OSeM Server verbinden und POST Operation durchführen
  Serial.println("-------------------------------------"); 
  Serial.print("Connectingto OSeM Server..."); 
  if (client.connect(server, 8000)) 
  {
    Serial.println("connected!");
    Serial.println("-------------------------------------");     
    //HTTP Header aufbauen
    client.print("POST /boxes/");client.print(SENSEBOX_ID);client.print("/");client.print(sensorId);client.println(" HTTP/1.1");
    client.println("Host: www.opensensemap.org"); 
    client.println("Content-Type: application/json"); 
    client.println("Connection: close");  
    client.print("Content-Length: ");client.println(jsonValue.length()); 
    client.println(); 
    //Daten senden
    client.println(jsonValue);
  }else 
  {
    Serial.println("failed!");
    Serial.println("-------------------------------------"); 
  }
  //Antwort von Server im seriellen Monitor anzeigen
  waitForServerResponse();
}

void waitForServerResponse()
{ 
  //Ankommende Bytes ausgeben
  boolean repeat = true; 
  do{ 
    if (client.available()) 
    { 
      char c = client.read();
      Serial.print(c); 
    } 
    //Verbindung beenden 
    if (!client.connected()) 
    {
      Serial.println();
      Serial.println("--------------"); 
      Serial.println("Disconnecting.");
      Serial.println("--------------"); 
      client.stop(); 
      repeat = false; 
    } 
  }
  while (repeat);
 }
```

## OpenSenseMap Registrierung
Nachdem man sich auf opensensemap.org registriet hat mit allen dazu benötigten Daten muss man nur noch unter manueller Konfiguration die Sesoren einstellen. Nach dem erhallt der E-mail muss man dann die ID Werte jehner Sensoren noch im Code anpassen bzw. eingben.

## Stationsaufbau
Wegen der Windsensoren sollte die Station möglichst frei stehen damit der Wind von allen Seiten an die Sensoren kommen kann.

<img src="https://github.com/Valderag/Sensebox/blob/master/IMG-20160422-WA0003.jpg" width="200"/>


## Kontakt
Tom Tolksdorf, t_tolk01@uni-muenster.de, 15.04.2016

 
