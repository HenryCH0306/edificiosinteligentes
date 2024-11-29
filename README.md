# edificiosinteligentes
#include <Arduino.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <IRremoteESP8266.h>
#include <IRsend.h>
#include <ir_Fujitsu.h>

// Configurações de rede Wi-Fi
const char* ssid = "edint";
const char* password = "12345678";

// Configurações do Broker MQTT
const char* mqtt_server = "maquiatto.com";
const int mqtt_port = 1883;
const char* mqtt_user = "";
const char* mqtt_password = "";

// Configuração do cliente MQTT
WiFiClient espClient;
PubSubClient client(espClient);

// Configuração do DHT
#define DHTPIN 21
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

// Configuração do IR
const uint16_t kIrLed = 23;
IRsend irsend(kIrLed);

// Configuração do ar-condicionado
IRFujitsuAC ac(kIrLed);
bool ac_on = false;

// Função para reconectar ao Wi-Fi e MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.println("Conectando ao Broker MQTT...");
    if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
      Serial.println("Conectado ao MQTT");
      client.subscribe("home/ac_control");
      client.subscribe("home/request_data");
    } else {
      Serial.print("Falha ao conectar. Tentando novamente em 5 segundos. Erro: ");
      Serial.println(client.state());
      delay(5000);
    }
  }
}

// Função chamada ao receber mensagens MQTT
void callback(char* topic, byte* payload, unsigned int length) {
  String mensagem;
  for (int i = 0; i < length; i++) {
    mensagem += (char)payload[i];
  }
  Serial.println("Mensagem recebida: " + mensagem);

  if (String(topic) == "home/ac_control") {
    if (mensagem == "ON") {
      ac_on = true;
      ac.stateReset();  // Restaura para o estado padrão
      ac.setMode(kFujitsuAcModeCool); // Configura modo refrigeração
      ac.setTemp(20);                 // Configura temperatura para 20°C
      ac.on();
      ac.send();
      Serial.println("Ar-condicionado ligado.");
    } else if (mensagem == "OFF") {
      ac_on = false;
      ac.off();
      ac.send();
      Serial.println("Ar-condicionado desligado.");
    }
  } else if (String(topic) == "home/request_data") {
    if (mensagem == "TEMPERATURE") {
      float temperatura = dht.readTemperature();
      char tempStr[8];
      dtostrf(temperatura, 1, 2, tempStr);
      client.publish("home/temperature", tempStr);
      Serial.println("Temperatura enviada via MQTT.");
    } else if (mensagem == "HUMIDITY") {
      float umidade = dht.readHumidity();
      char humStr[8];
      dtostrf(umidade, 1, 2, humStr);
      client.publish("home/humidity", humStr);
      Serial.println("Umidade enviada via MQTT.");
    }
  }
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  irsend.begin();

  // Conecta ao Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando ao Wi-Fi...");
  }
  Serial.println("Conectado ao Wi-Fi");

  // Configura o cliente MQTT
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Lê os dados do sensor DHT
  float temperatura = dht.readTemperature();
  float umidade = dht.readHumidity();

  if (!isnan(temperatura) && !isnan(umidade)) {
    char tempStr[8], humStr[8];
    dtostrf(temperatura, 1, 2, tempStr);
    dtostrf(umidade, 1, 2, humStr);

    client.publish("home/temperature", tempStr);
    client.publish("home/humidity", humStr);

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.println(" °C");

    Serial.print("Umidade: ");
    Serial.print(umidade);
    Serial.println(" %");
  } else {
    Serial.println("Erro ao ler o sensor DHT.");
  }

  delay(5000);
}

