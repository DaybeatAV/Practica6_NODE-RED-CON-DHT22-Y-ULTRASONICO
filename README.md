# Práctica 6 "NODE-RED CON DHT22 Y ULTRASÓNICO"
Dentro de este repositorio se mostrará cómo programa un Sensor DHT11 y un componente Ultrasónico enlazado con la herramienta NODE-RED.

## INTRODUCCION

### DESCRIPCION
El programa **NODE-RED** es una herramienta que permite interconectar hardware, software y servicios en la nube, facilitando la creacion de flujos de trabajo automatizados para diversas aplicaciones para el internet de la cosas. Recibiendo procesando, almacenando y visualizando datos  de sensores y otros dispositivos en tiempo real. En esta practica se utilizará para realizar un monitoreo de los datos (temperatura, humedad y distancia) del simulador WOKWI.
Vamos a utilizar una tarjeta ```ESP 32``` para adquirir los datos, utilizaremos un componente que en este caso será un sensor ```DTH11``` para otorgarnos una temperatura y humedad del entorno, además de ocupar un componente sensor ultrasonico para la medicion de  distancia.


## MATERIAL NECESARIO

Para realizar esta practica necesitas lo siguiente:

- Software [WOKWI](https://wokwi.com/)

- Tarjeta ```ESP 32```

- Sensor ```DHT11```

- ```HC-SR04 ULTRASONIC Distance sensor```

- Programa ```NODE-RED```

## INSTRUCCIONES PARA WOKWI

### Requisitos previos

El uso y operación de este repositorio necesitará entrar a la plataforma [WOKWI](https://wokwi.com/)

### Instrucciones de preparación de entorno

1. El primer paso será abrir la terminal de programación y colocar el siguiente código ya dentro del software de ```WOKWI```:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
DHTesp dhtSensor;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 15;   //Pin digital 3 para el Echo del sensor
const int DHT_PIN = 16;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "52.29.87.71";
String username_mqtt="JOSE DAVID A.V";
String password_mqtt="12345";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms

TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    doc["NOMBRE"] = "JOSE DAVID A.V";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = String(d);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DiplomadoJDAV", output.c_str());
  }
}
```


2. El segundo paso será agregar las siguientes librerias dentro del software para que se pueda realizar la práctica:

- ```DHT sensor library for ESPx```

- ```ArduinoJson```

- ```WiFi```

- ```PubSubClient```

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Librer%C3%ADas%20utilizadas.png)

3. Después procederemos a insertar y realizar la conexión de los componentes ```HC-SR04 ULTRASONIC Distance sensor``` y ```DHT11``` con la tarjeta ```ESP32```:

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Conexiones%20utilizadas.png)

## INSTRUCCIONES PARA INSTALACION DE **node-red**


### Requisitos previos

Para hacer uso de la herramienta ```Node-Red``` se necesita descargar el archivo Node.js v22.16.0 previamente en [NODE-RED](https://nodejs.org/en)

### Instrucciones de preparación de entorno

1.- Se debe descargar e instalar el programa siguiendo los pasos correctamente.


2.- El siguiente paso será dirigirse al CMD (simbolo del sistema) en modo administrador y escribir lo siguiente dentro de la terminal:

```
npm install -g --unsafe-perm node-red

```

3.- Después comprobaremos que funciona node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos).

```
node-red
```

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Arranque%20CMD.png)


4.- Para abrir la herramienta ```Node-Red``` nos vamos al explorador de red y buscamos el siguente link: 

```
localhost:1880
```

5.- Procederemos a instalar Dashboard abriendo la pestaña de opciones y elegimos *Manage palette*

6.- Seleccionamos *Install y buscamos node-red-dashboard*.

7.- Seleccionamos *node-red-dashboard*



## INSTRUCCIONES PARA LA COLOCACION DE BLOQUES Y LA CONEXION DE NODOS CORRESPONDIENTES EN  **Node-Red**

# Instrucciones

1.- Agregaremos un bloque *mqqtt in*.

2.- Configurar el bloque con el puerto mqtt con el ip 52.29.87.71 para hacer el enlace con el servidor que utilizaremos:


![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Configuraci%C3%B3n%20mqtt.png)



3.- Agregaremos un bloque *json* y lo configuraremos como se observa debajo:


![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Configuraci%C3%B3n%20json.png)



4.- Vamos a insertar 3 bloques *function* y los configuraremos con el siguente código respectivamente:

```
msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg
```


```
msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;
```

```
msg.payload = msg.payload.DISTANCIA;
msg.topic = "DISTANCIA";
return msg;
```


5.- Añadiremos un bloque de *debug*.


6.- Colocaremos los bloques de chart y gauge correspondientes con sus respectivas tablas.


7.- Un ejemplo de la conexión necesario entre los bloques es la siguiente:


![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Conexiones%20utilizadas%20en%20Node-Red.png)



### Instrucciónes de operación

1. Se inicia el simulador ```WOKWI```.

2. Procedemos a colocar la temperatura y humedad deseada dando doble click al sensor DHT11

3. Vamos a colocar una distancia dando doble click al sensor ultrasonico

4. Nos trasladaremos al navegador donde este abierto nuestro node-red y apretar el boton *instanciar*

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Acci%C3%B3n%20de%20Deploy.png)

5. En el *dash board* apretar en el siguiente boton:

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Acci%C3%B3n%20de%20dash%20board.png)

## Resultados.
Después de realizar las instrucciones de operación se deberán reflejar los datos simultaneos entre el simulador ```Wokwi``` y el programa ```Node-Red```.

#SIMULADOR DE WOKWI

![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Resultado%20final%20Wokwi.png)


#NODE-RED. TABLAS DE DATOS Y GRAFICOS


![](https://github.com/DaybeatAV/Practica6_NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/Pr%C3%A1ctica%206%20Resultado%20final%20Node-Red.png)



### Evidencias

https://github.com/user-attachments/assets/bc6e0d5b-4992-4828-911d-079e24b186fa

- Dentro de los archivos del repositorio hay un video que muestra la operación simultánea entre el programa Node-Red y el software Wokwi en la práctica

# Créditos

Creado y desarrollado por **JOSE DAVID AYALA VILLALBA**

-[GITHUB](https://github.com/DaybeatAV)
