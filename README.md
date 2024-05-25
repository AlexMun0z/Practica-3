# Practica-3A
## Part A: Generació d'una Pàgina Web
### Codi complet
```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <Arduino.h>
#include <FS.h>
#include <SPIFFS.h>

// SSID & Password
const char* ssid = "Xiaomi_11T_Pro"; // Enter your SSID here
const char* password = "12345678"; //Enter your Password here
WebServer server(80); // Object of WebServer(HTTP port, 80 is defult)
void handle_root(void);
String getHtmlContentFromFile(const char* filename) {
  // Intenta abrir el archivo en modo lectura
  File file = SPIFFS.open(filename, "r");

  // Verifica si el archivo se abrió correctamente
  if (!file) {
    Serial.println("No se pudo abrir el archivo " + String(filename));
    return ""; // Devuelve una cadena vacía en caso de error
  }

  // Lee el contenido del archivo en una cadena
  String fileContent = file.readString();

  // Cierra el archivo
  file.close();

  // Devuelve el contenido del archivo
  return fileContent;
}

void setup() {
  Serial.begin(115200);

  if (!SPIFFS.begin(true)) {
        Serial.println("An error occurred while mounting SPIFFS");
        return;
    }

    Serial.println("SPIFFS mounted successfully");

// Abre el directorio raíz
    File root = SPIFFS.open("/");
    if (!root) {
        Serial.println("Failed to open directory");
        return;
    }

    Serial.println("Files in SPIFFS:");
    File file = root.openNextFile();
    while (file) {
        Serial.print("  ");
        Serial.println(file.name());
        file = root.openNextFile();
    }

  Serial.println("Try Connecting to ");
  Serial.println(ssid);

  // Connect to your wi-fi modem
  WiFi.begin(ssid, password);

  // Check wi-fi is connected to wi-fi network
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected successfully");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP()); //Show ESP32 IP on serial

  server.on("/", handle_root);

  server.begin();
  Serial.println("HTTP server started");
  delay(100);
}

void loop() {
  server.handleClient();
}

// HTML & CSS contents which display on web server
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
<img src= 'https://images7.memedroid.com/images/UPLOADED849/5fa07a82be0e0.jpeg'>\
</body>\
</html>";

// Handle root url (/)
void handle_root() {
  String htmlContent = getHtmlContentFromFile("/web.html");
  server.send(200, "text/html", htmlContent);
  //server.send(200, "text/html", HTML);
}
```
### Funcionament per parts del codi

__1. Inclusió de Biblioteques__
```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>
#include <FS.h>
#include <SPIFFS.h>

```
- Arduino.h: Biblioteca estàndart d'Arduino
- Wifi.h: Permet la conexió a xarxes Wi-Fi.
- WebServer.h: Proporciona la funcionalitat per crear un servidor web.
- FS.h: Sistema d'arxius genérico per Arduino
- SPIFFS.h: Sistema d'arxius SPIFFS específic per ESP32

__2. Definició de Credencials Wi-Fi i Creació del Servidor__
```cpp
const char* ssid = "Xiaomi_11T_Pro"; // SSID de la red Wi-Fi
const char* password = "12345678"; // Contraseña de la red Wi-Fi
WebServer server(80); // Crear un objeto servidor en el puerto 80

```
Aquesta part del codi serveix per definir el nostre objecte server i definir la xarxa i contrasenya que hem d'utilitzar per accedir a aquest servidor.


__3. Declaració de Funcions__
```cpp
void handle_root(void);
String getHtmlContentFromFile(const char* filename);

```
Es fa la declaració de les dues funcions que utlitzarem:
- handle_root(void):
- getHtmlContentFromFile(const char* filename)

__4. Definició de Funció per Llegir Arxius__
```cpp
String getHtmlContentFromFile(const char* filename) {
  File file = SPIFFS.open(filename, "r");

  if (!file) {
    Serial.println("No se pudo abrir el archivo " + String(filename));
    return "";
  }

  String fileContent = file.readString();
  file.close();

  return fileContent;
}

```
__5. Contingut HTML y Gestionador de Solicituts__
```cpp
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1>My Primera Pagina con ESP32 - Station Mode &#128522;</h1>\
<img src= 'https://images7.memedroid.com/images/UPLOADED849/5fa07a82be0e0.jpeg'>\
</body>\
</html>";

void handle_root() {
  String htmlContent = getHtmlContentFromFile("/web.html");
  server.send(200, "text/html", htmlContent);
}
```

__6. Setup__
```cpp
void setup() {
  Serial.begin(115200);

  if (!SPIFFS.begin(true)) {
    Serial.println("An error occurred while mounting SPIFFS");
    return;
  }

  Serial.println("SPIFFS mounted successfully");

  File root = SPIFFS.open("/");
  if (!root) {
    Serial.println("Failed to open directory");
    return;
  }

  Serial.println("Files in SPIFFS:");
  File file = root.openNextFile();
  while (file) {
    Serial.print("  ");
    Serial.println(file.name());
    file = root.openNextFile();
  }

  Serial.println("Try Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected successfully");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP());

  server.on("/", handle_root);
  server.begin();
  Serial.println("HTTP server started");
  delay(100);
}
```
S'estableix la configuració inicial del programa:
- Serial.begin(115200): Inicialitza la comunicació sèrie en 115200 baudis.
- SerialBT.begin("ESP32test2"): Inicialitza la conexió Bluetooth i ens indica el nom del dispositiu que hem de buscar.
- Serial.println: Imprimeix per pantalla que el dispositiu es disponible per linkar amb bluetooth.
  
__7. Loop__
```cpp
void loop() {
  server.handleClient();
}

```
- if (Serial.available()) {...}: .Comprova si hi han dades disponibles en el port sèrie(com pot haver-hi en el monitor serial).
- SerialBT.write(Serial.read());: Si hi ha  dades disponibles, les llegeix des del port sèrie i les envía a través de Bluetooth mitjançant l'objecte SerialBT. 
- if (SerialBT.available()) {...}: Comprova si hhi han dades disponibles a través de Bluetooth (dades enviades des del dispositiu emparellat).
- Serial.write(SerialBT.read());: Si hi han dades disponibles ,les llegeix des del dispositiu connectat per Bluetooth i les envie al port sèrie per poder mostrarles pel monitor.
- delay(20);: Introduce un pequeño retraso de 20 milisegundos en cada iteración del loop para evitar una lectura o escritura excesivamente rápida, lo que ayuda a estabilizar la comunicación. Introduiex un petit retard de 20 msen cada iteració per evitar una lectura o escriptura massa ràpida i que sigui mes estable la comunicació entre dispositius.

Basicament les dades que escribim ja sigui pel monitor serial o per el dispositiu emparellat per Bluetooth, les sortides dels dos dispositius mostren el mateix, ja sigui llegint les dades d'un canal i fent la transmissió com rebent les dades i mostrant-les pel monitor del dispositiu.
