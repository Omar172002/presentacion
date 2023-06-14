# presentacion

#codigo de esp32

#include "NewPing.h"      // include NewPing library
#include "DHT.h"
#include <WiFi.h>
#include <PubSubClient.h>
#include <Firebase_ESP_Client.h>
#include <addons/TokenHelper.h>

#include "DHT.h"
int Gas_analog = 34;    // used for ESP32
#define DHTPIN 15     // Digital pin connected to the DHT sensor
// Feather HUZZAH ESP8266 note: use pins 3, 4, 5, 12, 13 or 14 --
// Pin 15 can work but DHT must be disconnected during program upload.

// Uncomment whatever type you're using!
#define DHTTYPE DHT11   // DHT 11
//#define DHTTYPE DHT22   // DHT 22  (AM2302), AM2321
//#define DHTTYPE DHT21   // DHT 21 (AM2301)

// potenciometro entrada
int potenciometro = 35;

// for ESP32 microcontroller
int trigPin = 18;      // trigger pin
int echoPin = 19;      // echo pin


// sensor de sonido entrada
int sensor = 32 ;

DHT dht(DHTPIN, DHTTYPE);
// Replace the next variables with your SSID/Password combination
const char* ssid = "INFINITUMULD3_2.4";
const char* password = "HcFq4xF7RY";


// Insert Firebase project API Key
#define API_KEY "AIzaSyDMqqK3-GVK98Bw4lUAARFKkhXnJftnlIE"

// Insert RTDB URLefine the RTDB URL */
#define DATABASE_URL "https://proyecto1-1744b-default-rtdb.firebaseio.com"

//Define Firebase Data object
FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;

unsigned long sendDataPrevMillis = 0;
int intValue;
float floatValue;

bool signupOK = false;
int count = 0;


void setup() 
{
  Serial.begin(115200);
  Serial.println(F("DHTxx test!"));
  delay(10);
  setup_wifi();
  dht.begin(); 

  
}

void setup_wifi() {
  //WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  //* Assign the api key (required) */
  config.api_key = API_KEY;

  /* Assign the RTDB URL (required) */
  config.database_url = DATABASE_URL;

  /* Sign up */
  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("ok");
    signupOK = true;
  }


  /* Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons/TokenHelper.h

  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
}


void loop(){ 


// codigo del sensor de gas
  int gassensorAnalog = analogRead(Gas_analog);
  Serial.println(gassensorAnalog);
  delay(100);
/////////////////////////////////

// codigo del potenciometro
// leemos del pin 35 valor
int valor = analogRead(potenciometro);
///////////////////////////////

// codigo del sensor de ultrasonido
 int distance = sonar.ping_median(5);
 Serial.print(distance);
////////////////////////////

// codigo del sensor de sonido
float  valorsonido =  analogRead(sensor) ;
////////////////////////////////

// codigo del sensor de temepratura y humedad
  // Wait a few seconds between measurements.
  delay(2000);
  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
  float h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  float t = dht.readTemperature();
  // Read temperature as Fahrenheit (isFahrenheit = true)
  float f = dht.readTemperature(true);

  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t) || isnan(f)) 
  {
    Serial.println(F("Failed to read from DHT sensor!"));
    return;
  }

  // Compute heat index in Fahrenheit (the default)
  float hif = dht.computeHeatIndex(f, h);
  // Compute heat index in Celsius (isFahreheit = false)
  float hic = dht.computeHeatIndex(t, h, false);


 if (Firebase.ready() && signupOK && (millis() - sendDataPrevMillis > 15000 || sendDataPrevMillis == 0)) {
    sendDataPrevMillis = millis();
    // Write an Int number on the database path test/int
    if (Firebase.RTDB.setInt(&fbdo, "test/temperatura F", f)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }
    else {
      Serial.println("FAILED");
      Serial.println("REASON: " + fbdo.errorReason());
    }
    count++;


    // enviar el valor de humedad a firebase
    // Write an Float number on the database path test/float
    if (Firebase.RTDB.setFloat(&fbdo, "test/humedad", h)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }

    // envia el valor de temperatura a firebase
    // Write an Float number on the database path test/float
    if (Firebase.RTDB.setFloat(&fbdo, "test/temperatura C", t)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }

    // envia el valor del sensor de gas a firebase 
    // escribe los valores del sensor de gas
    if (Firebase.RTDB.setFloat(&fbdo, "test/Gas_ppm", gassensorAnalog)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }

    // envia el valor del potenciometro a firebase 
    if (Firebase.RTDB.setFloat(&fbdo, "test/potenciometro", valor)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }

     // envia los datos del sensor de sonido a firebase
     // escribe los valores del sensor de ultrasonido
    if (Firebase.RTDB.setFloat(&fbdo, "test/distancia", distance)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }

    // envia los datos del sensor de ultrasonido a firebase
    // escribe los valores del sensor de ultrasonido
    if (Firebase.RTDB.setFloat(&fbdo, "test/sonidoVolumen", valorsonido)) {
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
    }
    else {
      Serial.println("FAILED");
      Serial.println("REASON: " + fbdo.errorReason());
    }
  }
}
