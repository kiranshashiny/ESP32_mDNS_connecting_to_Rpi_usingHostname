# ESP32_mDNS_connecting_to_Rpi_usingHostname


Putting this code out here so that it will help when connecting the ESP32 to a remote Raspberry Pi when we only know the name of the Rpi on the local network.

The test case is :

The Raspberry Pi's IP address can change each time the WiFi router reboots and we cannot hard code the IP address into ESP32 each time.

So the solution is to reach out to remote Raspberry Pi using it's hostname. ( the avahi daemon on the Rpi has to be running in the first place )

This resolution took a few days of effort and finally came across this blog post that resolved it.

https://techtutorialsx.com/2021/10/29/esp32-mdns-host-name-resolution/

The key piece of logic is in this section.

![image](https://github.com/kiranshashiny/ESP32_mDNS_connecting_to_Rpi_usingHostname/assets/14288989/0b6aa00f-af3c-44da-bdf3-5eb917bdfb80)


Look for mDNS() which initializes the multicast on ESP32, 
DebianBusterLite is the hostname of the Raspberry Pi on the same network having the 192.168.29.204 at this time and it change each time the WiFi Router reboots.

So to get the IP address of the Rpi to connect, I resolve the IP address with the hostname as shown below and then connect.

```
#include "ESPmDNS.h"
#include <WiFi.h>
    
const char* ssid = "XXXXXXXXXXXX";
const char* password =  "XXXXXXXX";

#define MDNS_DEVICE_NAME "my-esp32"
#define SERVICE_NAME "my-service"
#define SERVICE_PROTOCOL "udp"
#define SERVICE_PORT 5600

//String serverName = "DebianBusterLite.local";   // REPLACE WITH YOUR Raspberry Pi IP ADDRESS
String serverName = "192.168.29.204";   // REPLACE WITH YOUR Raspberry Pi IP ADDRESS

const int serverPort = 80;

WiFiClient client;

       
void setup(){
  Serial.begin(115200);
    
  WiFi.begin(ssid, password);
    
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
   
  if(!MDNS.begin(MDNS_DEVICE_NAME)) {
     Serial.println("Error encountered while starting mDNS");
     return;
  }
  
  MDNS.addService(SERVICE_NAME, SERVICE_PROTOCOL, SERVICE_PORT);
 
  Serial.println(WiFi.localIP());
  Serial.println(WiFi.getHostname());
 
  IPAddress serverIp;

  while (serverIp.toString() == "0.0.0.0") {
    Serial.println("Resolving host...");
    delay(250);
    serverIp = MDNS.queryHost("DebianBusterLite");
  }
 
  Serial.println("Host address resolved:");
  Serial.println(serverIp.toString());   
  
  
  if (client.connect(serverIp.toString().c_str(), serverPort)) {
    Serial.println("Connection successful!");    
  } else {

    Serial.println("Connection FAILED!");
  } 
  
}
    
void loop(){}

```

![image](https://github.com/kiranshashiny/ESP32_mDNS_connecting_to_Rpi_usingHostname/assets/14288989/7910b9bf-d45d-4710-a375-f0b6f4aa896d)


