# Reto, Equipo 1
Omar Arias Zepeda | A00830966 \
Diego Alejandro Michel Castro | A01641907 

=============================

## Planteamiento

Con los constantes avances en diversos componentes electrónicos podemos medir bastantes factores físicos para su análisis y uso, por medio de sensores y conectividad de dispositivos se puede desarrollar una transmición de información para su manejo y exposición a usuarios interesados por medio de aplicaciones, que es en lo que consta este proyecto.\
\
El desarrollo de los sistemas se dividió en 4 etapas: \
-Construcción del circuito físico y conexiones digitales \
-Creación de una firebase y conseción de permisos \
-Programación de los códigos para la conexión entre esp y firebase \
-Desarrollo de la aplicación para la transmición de resultados 

### Construcción del circuito físico y conexiones digitales

Para el desarrollo del trabajo se tomaron 5 mediciones físicas con sensores distintos para cada una, se trabajaron con los siguientes: \
\
1.) Medidor de temperatura en grados celsius y farenheit \
\
Diagrama de conexiones del sensor de humedad/temperatura \
<image width="350" height = "250" src="https://naylampmechatronics.com/img/cms/Blog/Tutorial%20DHT11%20y%20DHT22/conexion%20arduino%20y%20dht11.jpg" alt="Descripción de la imagen"> 
 
2.) Medidor de niveles de gas en partes por millón \
\
Diagrama de conexiones del sensor de gas \
<image width="350" height = "250" src="https://i0.wp.com/elprofealegria.com/wp-content/uploads/2020/09/Sensor-Gas-MQ2-MQ3-MQ7-Y-MQ135.png?resize=930%2C620&ssl=1" alt="Descripción de la imagen">
 
3.) Medidor de potencial eléctrico \
\ 
Diagrama de conexiones del potenciometro \
<image width="350" height = "250" src="https://eloctavobit.com/wp-content/uploads/2020/05/esquema-potenciometro_bb.jpg" alt="Descripción de la imagen">
 
4.) Sensor de humedad \
5.) Medidor de sonido/volúmen \
\
Diagrama de conexiones del sensor de sonido \
<image width="350" height = "250" src="https://www.prometec.net/wp-content/uploads/2016/05/montaje-protoboard-sensor-sonido-salida-anal%C3%B3gica.png" alt="Descripción de la imagen"> 

El ensamblado en protoboard usando periféricos del esp32 se construyó de la siguiente manera: \
![base de datos](circuito.jpg "Título alternativo")

============================= 
 
## Creación de una firebase y conseción de permisos

Para contar con una base de datos útil y manejable, se creó y modificó una firebase de google, accesible para los propietarios y usuarios con invitación recibida para su modificación como desarrolladores. 

Se decidió dejar los permisos públicos para que el esp pudiera publicar las mediciones tomadas sin mayores complicaciones, y poderse estar actualizando la información conforme se ejecuten los programas y se consigan los nuevos datos. 

```
{
 "rules" : {
  ".read" : true, //2023-5-26
  ".write" : true, //2023-5-26
 }
}
 
```

En la misma, se añadió la funcionalidad de Web App para la proyección de datos en una base a tiempo real contando con un ID identificador e información necesaria ingresada en los códigos que permiten su manejo y la app para el registro de datos puestos por el el usuario. 
 
 ```
// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
 
const firebaseConfig = {
  apiKey: "AIzaSyDMqqK3-GVK98Bw4lUAARFKkhXnJftnlIE",
  authDomain: "proyecto1-1744b.firebaseapp.com",
  databaseURL: "https://proyecto1-1744b-default-rtdb.firebaseio.com",
  projectId: "proyecto1-1744b",
  storageBucket: "proyecto1-1744b.appspot.com",
  messagingSenderId: "25820598689",
  appId: "1:25820598689:web:51fa9d7ae03c7ffdbe3be0",
  measurementId: "G-FV4NXXBYRZ"
};
 
 ```
Con la información de la firebase y los detalles de la red correctos en los que se está ejecutando para cada equipo, se procede a ejecutar el código para que las medidas captadas se reflejen en la firebase. En el circuito construido, el sensor de distancia resultó dañado, por lo que su valor marca 0. \
Base de datos de firebase con los datos de los sensores \
![base de datos](base.png "Título alternativo")
 
=============================
 
## Programación de los códigos para la conexión entre esp y firebase


```
#include "NewPing.h"      // include NewPing library
#include "DHT.h"
#include <WiFi.h>
#include <PubSubClient.h>
#include <Firebase_ESP_Client.h>
#include <addons/TokenHelper.h>

//ENTRADAS DE LOS SENSORES

// entrada del sensor de gas 
int Gas_analog = 34;    // used for ESP32

// entrada del sensor de temperatura humedad
#define DHTPIN 15     // Digital pin connected to the DHT sensor
// Uncomment whatever type you're using!
#define DHTTYPE DHT11   // DHT 11


// potenciometro entrada
int potenciometro = 35;

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

    // CODIGO PARA ENVIAR LOS DATOS A FIREBASE

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
```

## Desarrollo de la aplicación para la transmición de resultados 

El entorno usado para el diseño y construcción de la aplicación fué AppInventor, un entorno de desarrollo propio del MIT. En dicho entorno se colocan layouts y componentes visibles o no visibles para el usuario final, para su correcto  funcionamiento se deben agrupar bloques de comando para cada componente necesario en la vista de diseño.
 
Para la aplicación "Measure Wizard" creada, se segmentó el uso de pantalla total en 6 layouts alineados al centro, dentro de los cuales se establece la salida de información contenida en la base de datos por medio de labels y conectada a nuestra firebase por una funcionalidad nativa de appinventor. Para el ingreso de información por el usuario, se agregaron 2 TextBox para ingresar los datos, el correspondiente a nombre admite texto en caracteres mientras que el correspondiente a número solo admite dígitos.
 
La visualización previa en el entorno es ligeramente reducida por los tamaños de pantalla disponibles y los tamaños de fuente para los componentes visibles elegidos, la vista de diseño para el desarrollador es la siguiente:
 
![image](https://github.com/Omar172002/presentacion/assets/106403382/610529ac-dbf7-45ed-98a8-5702edd1f085)

Mientras que la vista de usuario en el dispositivo móvil es la siguiente:
 
![image](https://github.com/Omar172002/presentacion/assets/106403382/c7730f11-9da7-42fe-b0b0-85e0a7eba745)

Para poder mostrar los datos de los sensores almacenados en la firebase y permitir el registro de información para nombre y número en las mismas se organizaron bloques de comando que realicen dichas acciones.
Al conectar con la base se convierten los valores a texto y se busca dentro de la misma el nombre de la etiqueta correspondiente para el valor correcto, mientras que para el registro el campo designado a número, el valor ingresado se convierte a texto para guardarlo en el tópico test al igual que el campo para nombre, dicha información se almacena en el mismo sitio que las medidas de variables físicas.

![image](https://github.com/Omar172002/presentacion/assets/106403382/e6771885-4b38-45d1-a4f0-9299b35f7068)

 
 
