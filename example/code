//ADS1115 anschließen:
//        --> VDD - 3.3V
//        --> Ground - Ground
//        --> SCL - D1
//        --> SDA - D2


//Bibliotheken eibinden
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <Adafruit_ADS1X15.h>
#include <NTPClient.h>
#include <WiFiUdp.h>



//Konstanten 
//Zeitverschiebung UTC <-> MEZ (Winterzeit) = 3600 Sekunden (1 Stunde)
//Zeitverschiebung UTC <-> MEZ (Sommerzeit) = 7200 Sekunden (2 Stunden)
const long utcOffsetInSeconds = 7200;
//Zugang WLAN
const char* SSID = "FRITZ!Box Fon WLAN 7360"; //SSID WLAN-AP
const char* PSK = "B351-C56y-t2tz-g38s"; //PW-WLAN-AP
const char* MQTT_BROKER = "192.168.178.20"; //IP-Adresse MQTT-Broker
// Pin zum Ansteuern von Ventil 1
const int v1 = D8;


//Objekte erstellen
Adafruit_ADS1115 ads;
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org", utcOffsetInSeconds);
WiFiClient espClient;
PubSubClient client(espClient);



//Variablen
long lastMsg = 0;
char msg[50];
int value = 0;
float soil = 0;
float temp = 0;



void setup() {
  pinMode(v1, OUTPUT);

  Serial.begin(115200);
  client.setServer(MQTT_BROKER, 1883); //Port 1883 wird standardmäßig genutzt
  
  ads.setGain(GAIN_ONE); //Verstärkung einstellen --> Gain_one: 0,125mV pro Bit
  ads.begin();

  client.setCallback(callback);
  timeClient.begin();
}

//WiFi verbinden
void setup_wifi() {
  delay(10);
  
  //Serielle Ausgabe um Verbindungsaufbau darzustellen
  Serial.println("Connecting to");
  Serial.println(SSID);

  //WiFi Verbindung aufbauen
  WiFi.begin(SSID, PSK);


  //Verbindungsaufbau signalisieren
  while(WiFi.status() != WL_CONNECTED){
    delay(500);
    Serial.print(".");
  }

  //Verbindung hergestellt
  Serial.println("Wifi connected");
  Serial.println("IP-address: ");
  Serial.println(WiFi.localIP());
}

//Erneut verbinden falls Verbindung abgebrochen
void reconnect() {
  while(!client.connected()){
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect("ESP8266Client")) {
      Serial.println("connected");
      delay(5000);
      // Abonnieren des topics: "esp8266/output"
      client.subscribe("esp8266/output");
    }
    else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}
//Abfragen der MQTT-Nachricht
void callback(char* topic, byte* message, unsigned int length) {
  Serial.print("Message arrived on topic: ");
  Serial.print(topic);
  Serial.print(". Message: ");
  String messageTemp;
  
  for (int i = 0; i < length; i++) {
    Serial.print((char)message[i]);
    messageTemp += (char)message[i];
  } 
  Serial.println();

  //Pointer für Vergleich erstellen
  const char* m;
  m = "esp8266/output";

  // Prüfen ob eine Nachricht des Topics: "esp8266/output" erhalten wurde
  if(strcmp(topic, m) == 0){
      Serial.print("Changing Ventil to ");
      if(messageTemp == "0"){
        //Variablen MQTT-Übergabe
        String mesg = "";
        char msgf[25];
        
        mesg = "1";
        mesg.toCharArray(msgf, 25);
        client.publish(m, msgf);
        Serial.println("msg");
        
        digitalWrite(v1, HIGH);
        Serial.print("open");
        delay(1800000); //30 Minuten bewässern   
        //Schalter auf OFF setzen
        digitalWrite(v1, LOW);
        Serial.println("close");
      }
      else if(messageTemp == "1"){
        digitalWrite(v1, LOW);
        Serial.print("close");
      }
  }
  Serial.println();
}


//Zu jeder vollen Stunde werden Bodenfeuchte und Temperatur übermittelt, sowie der 
//Stand des Schalters zur manuellen Bewässerung abgefragt.

void loop() {  
 
  long now = millis();
  
  if (now - lastMsg > 3600000) {
    lastMsg = now;

    setup_wifi();

    //Prüfen ob verbunden ansonsten neu verbinden
    if(!client.connected()){
      reconnect();
    }
    if(!client.loop())
      client.connect("ESP8266Client5");

    //MQTT Übergabeparameter deklarieren
    String msg = "";
    char msgFeuchtigkeit[25];
    char msgTemperatur[25];
  
    //Bodenfeuchte einlesen
    Serial.println("Bodenfeuchte einlesen");
    soil = ads.readADC_SingleEnded(0);
    //soil = analogRead(A0);
    
  
    //Tatsächliche Bodenfeuchte kalkulieren
    soil = ads.computeVolts(soil);
    //soil = soil * (0.125/1000.0);
    //soil = soil * (3.2/1023);
    soil = soil / 3 * 50;
    msg = String(soil);
    msg = msg + "%";
    Serial.println(soil);
    msg.toCharArray(msgFeuchtigkeit, 25);
    client.publish("Bodenfeuchtigkeit", msgFeuchtigkeit);
    //delay(15000);
  
    //Temperatur einlesen
    temp = ads.readADC_SingleEnded(1);
    //temp = analogRead(A0);
    
  
    //Tatsächliche Temperatur kalkulieren
    temp = ads.computeVolts(temp);
    //temp = temp * (0.125/1000.0);
    //temp = temp * (3.2/1023);
    temp = (temp - 0.5) * 100;
    msg = String(temp);
    msg = msg + "°C";
    Serial.println(temp);
    msg.toCharArray(msgTemperatur, 25);
    client.publish("Temperatur", msgTemperatur);
    
    client.disconnect();
    WiFi.mode(WIFI_OFF);
  }
  //Zeit updaten
  timeClient.update();

  
  //Prüfen ob Bodenfeuchte unter Grenzwert und Bewässerungszeitpunkt 6 Uhr morgens
     if (soil < 22.00 && timeClient.getHours() == 6 && timeClient.getMinutes() <= 30){
      bewaessern();
  }
}

  //Funktion um 30 Minuten zu bewässern
void bewaessern (){
    digitalWrite(v1, HIGH);
    Serial.println("Ventil: High");
    delay(1800000);
    digitalWrite(v1, LOW);
    Serial.println("Ventil: Low");
    soil = 22.00;
}
