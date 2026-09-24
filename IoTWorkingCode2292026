// Library
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <DHT.h>
#include <WiFi.h> 
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Define Pin
#define LDR_PIN 32
#define DHT_PIN 4
#define LED_PIN 27
#define SCL_PIN 5 //Pin SCL OLED
#define SDA_PIN 18 //PIN SDA OLED

//Define OLED
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

//Define DHT Type
#define DHT_TYPE DHT11

//Initialize DHT
DHT dht(DHT_PIN,DHT_TYPE);

//Initialize OLED
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT,&Wire, OLED_RESET);

//Menentukan Threshold untuk Cahaya, Suhu, dan Kelembapan
const int light_threshold= 3000;
const float temp_threshold= 27.0;
const float hum_threshold= 65.0;

//Konfigurasi Wifi
const char* ssid = "";
const char* password = "";
const char* mqtt_server = "broker.hivemq.com";
const int port = 1883;
const char* topic = "smartroom/kelompok6/data";

WiFiClient espClient;
PubSubClient client(espClient);


void setup_wifi() {
 delay(10);
 Serial.println();
 Serial.print("Connecting to ");
 Serial.println(ssid);


 WiFi.mode(WIFI_STA);
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

//Untuk reconnect waktu disconnect
void reconnect() {
 while (!client.connected()) {
   Serial.print("Attempting MQTT connection...");
   String clientId = "ESP32Client-";
   clientId += String(random(0xffff), HEX);
   if (client.connect(clientId.c_str())) {
     Serial.println("Connected");
    } 
    else {
     Serial.println("Connection failed, try again in 5 seconds");
     delay(5000);
   }
  }
}

//format JSON yang dikirim ke MQTT Hivemq dan kirim ke hivemq
void publish_format(int lightValue, bool kondisiruangan, bool dht_condition, float temperature, float humidity, bool LEDON){
  JsonDocument doc;
  doc ["LightValue"]   = lightValue;
  doc ["KondisiRuangan"] = kondisiruangan;
  if(dht_condition){
    doc ["Temperatur"] = temperature;
    doc ["Kelembapan"] = humidity;
  }
  else{
    doc ["Temperatur"] = nullptr;
    doc ["Kelembapan"] = nullptr;
  }

  doc ["NyalaLED"] = LEDON;

  char data [256];
  serializeJsonPretty(doc, data);
  client.publish(topic, data);

  if(client.publish(topic, data)){
    Serial.println("Data sukses dikirim");
  }
  else{
    Serial.println("Data gagal dikirim");
  }
} 



void setup() {
Serial.begin(115200);

//Mulai DHT11
dht.begin();

//LED
pinMode(LED_PIN, OUTPUT);
digitalWrite(LED_PIN, LOW);


Wire.begin(SDA_PIN, SCL_PIN);

setup_wifi();
client.setServer(mqtt_server, port);

if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 
    Serial.println("OLED tidak ditemukan!, cek OLED");
    while(true){
    delay(1000); }
}
  // Clear OLED
  display.clearDisplay();
  //Set warna font di OLED
  display.setTextColor(WHITE);
  //START OLED
  display.setTextSize(2);
  display.setCursor(10, 10);
  display.println("SYSTEM");

  display.setTextSize(2);
  display.setCursor(10, 35);
  display.println("STARTING");

  display.display();

  delay(2000);
}


void loop() {

if (!client.connected()) {
   reconnect();
 }
 client.loop();

// Baca Sensor Cahaya
int lightValue=analogRead(LDR_PIN);

//Ambil nilai sensor LDR dan print di serial monitor
Serial.print("LDR: ");
Serial.println(lightValue);

//Clear display OLED dari System Starting
display.clearDisplay();

//Cek Value LDR
//Waktu Ruangan Terang
if(lightValue < light_threshold){

  Serial.println ("Ruangan Terang");

  //Baca DHT11
  float temperature=dht.readTemperature();
  float humidity=dht.readHumidity();

  //Cek apakah DHT11 jalan
  //Kalau DHT11 Tidak jalan
  //========================================
    if(isnan(temperature)||isnan(humidity)){
      digitalWrite(LED_PIN, LOW);
      Serial.println("DHT11 Error, Cek Koneksi DHT");

     //Output di OLED kalau DHT tidak jalan
      display.setTextSize(1);
      display.setCursor(22,0);
      display.println("Ruangan Terang");

      display.setTextSize(1);
      display.setCursor(31, 20);
      display.println("DHT11 Error");

      display.setTextSize(1);
      display.setCursor(43, 40);
      display.println("LED Off");

      display.display();
      //void publish_format(int lightValue, bool kondisiruangan, bool dht_condition, float temperature, float humidity, bool LEDON)
      publish_format(lightValue, true, false, 0, 0, false );
  }
  //=========================================
    //Kalau DHT11 Jalan
    else{
      Serial.print("Suhu: ");
      Serial.print(temperature);
      Serial.println(" C");

      Serial.print("Kelembapan: ");
      Serial.print(humidity);
      Serial.println(" %");
      
      //Kalau ruangan panas/lembap
      //=======================================================
      if(temperature>temp_threshold || humidity>hum_threshold){
        digitalWrite(LED_PIN,HIGH);
        Serial.println("Ruangan Panas");
        Serial.println("LED Menyala");

        //Output OLED Waktu Ruangan Panas
        display.setTextSize(1);
        display.setCursor(0,0);
        display.println("Ruangan Terang");

        display.setTextSize(1);
        display.setCursor(0,15);
        display.print("Suhu: ");
        display.print(temperature,1);
        display.println(" C");

        display.setTextSize(1);
        display.setCursor(0,30);
        display.print("Kelembapan: ");
        display.print(humidity,1);
        display.println(" %");

        display.setTextSize(1);
        display.setCursor(0,45);
        display.print("LED Menyala");

        display.display();
        //void publish_format(int lightValue, bool kondisiruangan, bool dht_condition, float temperature, float humidity, bool LEDON)
        publish_format(lightValue, true, true, temperature, humidity, true);
        }
      //=============================================================
      //Kalau ruangan normal
      //=============================================================
      else{
        digitalWrite(LED_PIN, LOW);
        Serial.println("Ruangan Normal");
        Serial.println("LED Mati");

        //Output OLED Waktu ruangan normal
        display.setTextSize(1);
        display.setCursor(0,0);
        display.println("Ruangan Terang");

        display.setTextSize(1);
        display.setCursor(0,15);
        display.print("Suhu: ");
        display.print(temperature,1);
        display.println(" C");

        display.setTextSize(1);
        display.setCursor(0,30);
        display.print("Kelembapan: ");
        display.print(humidity,1);
        display.println(" %");

        display.setTextSize(1);
        display.setCursor(0,45);
        display.print("LED Mati");

        display.display();
        //void publish_format(int lightValue, bool kondisiruangan, bool dht_condition, float temperature, float humidity, bool LEDON)
        publish_format(lightValue, true, true, temperature, humidity, false);
        }
      //==========================================================  
  }

}

//Waktu Ruangan Gelap
else{
  digitalWrite(LED_PIN, LOW);
  Serial.println("Ruangan Gelap");
  Serial.println("LED Mati");

  //Output OLED Waktu Ruangan Gelap
  display.setTextSize(2);
  display.setCursor(22,0);
  display.println("RUANGAN");

  display.setTextSize(2);
  display.setCursor(34,20);
  display.println("GELAP");

  display.setTextSize(1);
  display.setCursor(22,40);
  display.println("Nyalakan Lampu");

  display.display();
  //void publish_format(int lightValue, bool kondisiruangan, bool dht_condition, float temperature, float humidity, bool LEDON)
  publish_format(lightValue, false, false, 0, 0, false);

}
Serial.println("=====================================");
delay(3000);
}
