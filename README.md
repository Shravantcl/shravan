# shravan
//ESP8266 msg of temp and humidity PH22
#include <ESP8266WiFi.h>
const char* ssid     = "********";      // Service set identifier of local network---------------------USER Input here
const char* password = "*******";   // Password on network -----------------------------------------------------
String result;
float h = 111;
float t = 222;
#include <DHT.h>
#define DHTPIN 0     // GPIO 0 on ESP12E Lolin is labelled pin D3 
// Uncomment whatever type you're using!
//#define DHTTYPE DHT11   // DHT 11 
#define DHTTYPE DHT22   // DHT 22  (AM2302)
//#define DHTTYPE DHT21   // DHT 21 (AM2301)

// Connect pin 1 (on the left) of the sensor to +5V
// NOTE: If using a board with 3.3V logic like an Arduino Due connect pin 1
// to 3.3V instead of 5V!
// Connect pin 2 of the sensor to whatever your DHTPIN is
// Connect pin 4 (on the right) of the sensor to GROUND
// Connect a 10K resistor from pin 2 (data) to pin 1 (power) of the sensor
// Initialize DHT sensor.
// Note that older versions of this library took an optional third parameter to
// tweak the timings for faster processors.  This parameter is no longer needed
// as the current DHT reading algorithm adjusts itself to work on faster procs.

DHT dht(DHTPIN, DHTTYPE); // Initialize DHT sensor
void setup() 
{
  Serial.begin(115200);
  Serial.setTimeout(2000);

  while(!Serial) { } // wait up to Serial ready
  WiFi.hostname("ESP8266HumiditySensor"); //This will changes the hostname of the ESP8266 to display neatly on the network esp on router.
  WiFi.begin(ssid, password);
  Serial.println("DHT22 test!");
  dht.begin();
  delay(2000);
  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
   h = dht.readHumidity();
   t = dht.readTemperature();
   delay(1000);

  // check if returns are valid, if they are NaN (not a number) then something went wrong!
 if (isnan(t) || isnan(h)) {
    Serial.println("Failed to read from DHT");
  } else {
    Serial.print("Humidity: "); 
    Serial.print(h);
    Serial.print(" % and ");
    Serial.print("Temperature: "); 
    Serial.print(t);
    Serial.println(" *C");
  }
  const char host[ ]        = "maker.ifttt.com";          // maker channel of IFTTT http://maker.ifttt.com
  const char trigger[ ]     = "humidity";                   //name of the trigger you would like to send to IFTTT --------------------
  const char APIKey[ ]      = "clzxVXormvuxqMz4Oi3ucB";      //Your maker key for Webhooks on IFTTT-----------
  Serial.print("Connect to: ");
  Serial.println(host);
  // WiFiClient to make HTTP connection
  WiFiClient client;
   if (!client.connect(host, 80)) {
    Serial.println("connection failed");
    return;
    }

// Build URL
  String url = String("/trigger/") + trigger + String("/with/key/") + APIKey + String("?value1=") + h + String("Temp=") + t;
  Serial.print("Requesting URL: ");
  Serial.println(url);

// Send request to IFTTT
  client.print(String("GET ") + url + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n"); 
  //FYI rn rn is just two new lines to comply with http format
   delay(20);

// Read all the lines of the reply from server and print them to Serial
  Serial.println("Respond:");
  while(client.available()){
    String line = client.readStringUntil('\r');
    Serial.print(line);
    }
  Serial.println();
  Serial.println("closing connection");
  Serial.println(); //space things our in serial monitor for purdy
  Serial.println();
  client.stop();  //V5 added this to disconnect
  Serial.println("Your Daily Report Humidity level and Temp"); //Send things to serial for handy dandy info
  Serial.print("humidity ");
  Serial.println(h);
  Serial.print("temp ");
  Serial.println(t);
  Serial.println("Deep sleep mode entered");
  ESP.deepSleep(30e6); //30 for 30 seconds 7200 for 2 hours 86400 for 1 day
 }
void loop() //Where nothing happens because esp8266 deep sleep mode
{ 

}
