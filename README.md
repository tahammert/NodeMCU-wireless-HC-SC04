# NodeMCU wireless connection HC-SC04 via Web Server
Reading distance data from HC-SR04 sensor using ESP-8266 and sharing it wirelessly with devices on the same network via webserver.
At the beginning of the code, change 'your_ssid' to your wifi name and 'your_password' to that network's password. 

## How does it work?

First of all, nodemcu joins the wifi network you are connected to according to the information you enter. We connected the pins of the NodeMCU according to the table given below and defined them. We define the speed of sound. In order to see the data measured by the sensor, we started a serial port at 115200 baud. We set the pinmodes as they should be. Then we started the wifi connection. At this stage, we also added the code that prints the IP address that we need to connect to the serial port. We also adjusted the design of the html page that will appear when we connect it. (Changes can be made easily upon request.)

## Connections

| HC-SR04       | NodeMCU       |
| ------------- | ------------- |
|     Vdd       |      Vin      |
| Trig          | D6=12         |
| Echo  | D5=14  |
| Gnd  | GND  |

## Windows Screenshots
![pc_webserver_ss](https://user-images.githubusercontent.com/72361022/230805365-20ef4a3c-3112-410b-aca0-a23393bccce4.png)



## Code

```
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

/*Put your SSID & Password*/
const char* ssid = "your_ssid";  //Wifi Name
const char* password = "your_password";  //wifi Password

ESP8266WebServer server(80);

const int trigPin = 12;
const int echoPin = 14;
const int statusLed= 16;

#define SOUND_VELOCITY 0.034

float dist;
long duration;
float distanceCm;
 
void setup() {
  Serial.begin(115200);
  delay(100);

  pinMode(trigPin, OUTPUT); 
  pinMode(echoPin, INPUT);
  pinMode(statusLed,OUTPUT);
           
  Serial.println("Connecting to WiFi");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print("."); 
  }
  Serial.println("");
  Serial.println("WiFi Connected..!");
  Serial.print("IP: ");  Serial.println(WiFi.localIP());

  server.on("/", handle_OnConnect); 
  
  server.onNotFound(handle_NotFound);

  server.begin();
  Serial.println("HTTP Server Started");
}

void loop() {
  digitalWrite(statusLed, HIGH);
  server.handleClient(); 
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  duration = pulseIn(echoPin, HIGH);
  distanceCm = duration * SOUND_VELOCITY/2;

  Serial.print("Distance (cm): ");
  Serial.println(distanceCm);

  delay(1000);
}

void handle_OnConnect() {
  dist = distanceCm; // 
  server.send(200, "text/html", SendHTML(dist)); 
}

void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
}

String SendHTML(float dist){
  String ptr = "<!DOCTYPE html> <html>\n";
  ptr +="<head><meta http-equiv=\"refresh\" content=\"1\">\n"; // refresh the page every 10 seconds
  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>Using HC-SR04 Sensor with ESP8266 NodeMCU</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
  ptr +="</style>\n";
  ptr +="</head>\n";
  ptr +="<body bgcolor= BlanchedAlmond style=color:white>\n";
  ptr +="<div id=\"webpage\">\n";
  ptr +="<h1>Using HC-SR04 Sensor with ESP8266 NodeMCU</h1>\n";
  
  ptr +="<p>Distance (cm):";
  ptr +=(int)dist;
  ptr +=" cm</p>";

  ptr +="<a href=>";
  ptr +="<button>Measure</button></a>";

  ptr +="</div>\n";
  ptr +="</body>\n";
  ptr +="</html>\n";
  return ptr;
}
```
## iOS (Safari) Screenshots
![NodeMCU_webserver_iphone](https://user-images.githubusercontent.com/72361022/230800902-b776f215-b24f-4980-92da-3ea61ca36a2e.jpg)


